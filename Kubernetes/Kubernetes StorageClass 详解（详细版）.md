# Kubernetes StorageClass 详解（详细版）

`StorageClass` 是 Kubernetes **动态存储供应（Dynamic Provisioning）** 的核心资源对象。

它解决的问题是：

在没有 `StorageClass` 之前：

- 管理员必须提前创建很多 `PV（PersistentVolume）`
- 开发者创建 `PVC` 时只能绑定已有 PV
- 如果没有合适大小的 PV，PVC 就会 Pending

这样：

- 运维麻烦
- 资源浪费
- 不灵活

有了 `StorageClass` 后：

用户只需要申请 PVC：

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pvc
spec:
  storageClassName: nfs-storage
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 20Gi
```

K8s 会自动：

1. 找到 StorageClass
2. 调用 Provisioner
3. 动态创建 PV
4. PVC 自动绑定

这就是：

**PVC -> StorageClass -> Provisioner -> 自动创建 PV -> Pod 使用**

------

# 一、StorageClass 是什么

StorageClass 是：

> Kubernetes 中定义“如何创建存储卷”的模板

它描述：

- 用哪个存储插件
- 用什么参数创建
- 回收策略
- 卷绑定时机
- 扩容策略

类似：

如果 PVC 是“租房申请”

StorageClass 就是：

“房屋建设规则”

包括：

- 房子在哪建
- 多大
- 建成后能不能扩建
- 租客走后拆不拆

------

# 二、StorageClass 工作流程（核心）

完整流程：

```plaintext
Pod
 ↓
PVC
 ↓
StorageClass
 ↓
External Provisioner / CSI Driver
 ↓
创建底层存储
 ↓
生成 PV
 ↓
PVC 绑定 PV
 ↓
Pod 挂载
```

例如：

你创建：

```yaml
PVC:
  storage: 100Gi
  storageClassName: fast-ssd
```

控制器：

发现：

```plaintext
没有匹配 PV
↓
查看 fast-ssd StorageClass
↓
调用 csi provisioner
↓
云平台创建磁盘
↓
生成 PV
↓
绑定 PVC
↓
Pod 使用
```

这是动态供给。

------

# 三、StorageClass YAML 结构详解

标准例子：

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast-ssd
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp3
  fsType: ext4
reclaimPolicy: Delete
allowVolumeExpansion: true
volumeBindingMode: WaitForFirstConsumer
```

逐项拆解。

------

## 1、apiVersion

```yaml
apiVersion: storage.k8s.io/v1
```

StorageClass API 组。

固定写法。

------

## 2、kind

```yaml
kind: StorageClass
```

资源类型。

------

## 3、metadata.name

```yaml
metadata:
  name: fast-ssd
```

StorageClass 名称。

PVC 通过：

```yaml
storageClassName: fast-ssd
```

引用它。

------

## 4、provisioner（最核心）

指定：

> 谁负责创建底层卷

例如：

### AWS EBS

```yaml
provisioner: ebs.csi.aws.com
```

------

### GCE PD

```yaml
provisioner: pd.csi.storage.gke.io
```

------

### NFS

```yaml
provisioner: nfs.csi.k8s.io
```

------

### Ceph RBD

```yaml
provisioner: rbd.csi.ceph.com
```

------

### vSphere

```yaml
provisioner: csi.vsphere.vmware.com
```

------

作用：

K8s 收到 PVC 后调用对应 provisioner：

```plaintext
CreateVolume()
DeleteVolume()
ExpandVolume()
AttachVolume()
```

这些接口来自 CSI。

------

## 5、parameters（存储参数）

给 provisioner 的参数。

例如：

```yaml
parameters:
  type: gp3
  fsType: ext4
```

含义：

- 用 gp3 磁盘
- 格式化 ext4

不同存储支持不同参数。

------

### AWS

```yaml
parameters:
  type: gp3
  iops: "5000"
  throughput: "250"
```

控制：

- IOPS
- 吞吐

------

### Ceph

```yaml
parameters:
  pool: kube
  imageFormat: "2"
```

指定：

- 存储池
- 镜像格式

------

### NFS

可能：

```yaml
parameters:
  server: 10.0.0.10
  share: /data
```

------

注意：

参数错了：

PVC 会卡住：

```plaintext
ProvisioningFailed
```

查看：

```bash
kubectl describe pvc
```

------

## 6、reclaimPolicy（回收策略）

PVC 删除后，PV 怎么处理？

有两个：

------

### Delete（默认）

```yaml
reclaimPolicy: Delete
```

删除 PVC：

```plaintext
PVC 删除
↓
PV 删除
↓
底层磁盘删除
```

适合：

- 临时环境
- 测试

风险：

误删数据

------

### Retain

```yaml
reclaimPolicy: Retain
```

删除 PVC：

```plaintext
PVC 删除
PV Released
底层盘保留
```

管理员手动处理。

适合：

- 生产数据库
- 数据保护

------

生产推荐：

数据库：

```yaml
Retain
```

缓存：

```yaml
Delete
```

------

## 7、allowVolumeExpansion（在线扩容）

```yaml
allowVolumeExpansion: true
```

允许：

```yaml
storage: 20Gi -> 100Gi
```

修改 PVC：

```bash
kubectl edit pvc mysql-pvc
```

改：

```yaml
storage: 100Gi
```

系统自动：

- 扩云盘
- 扩文件系统

前提：

CSI 支持。

查看：

```bash
kubectl describe sc
```

------

如果：

```yaml
false
```

会报：

```plaintext
Forbidden: only dynamically provisioned pvc can be resized
```

------

## 8、volumeBindingMode（非常重要）

决定：

**什么时候创建 PV**

两个模式。

------

### Immediate（默认）

```yaml
volumeBindingMode: Immediate
```

PVC 创建立刻创建 PV。

流程：

```plaintext
PVC 创建
↓
立刻创建卷
↓
等待 Pod
```

问题：

如果卷有可用区限制：

东京区创建了盘

但 Pod 调度到大阪节点

挂载失败。

------

### WaitForFirstConsumer（生产推荐）

```yaml
volumeBindingMode: WaitForFirstConsumer
```

流程：

```plaintext
PVC 创建
↓
不创建卷
↓
等待 Pod 调度
↓
确定节点 AZ
↓
创建同 AZ 卷
↓
挂载
```

解决：

跨可用区问题。

云上强烈推荐。

------

# 四、默认 StorageClass

查看：

```bash
kubectl get sc
```

可能：

```plaintext
NAME                 PROVISIONER          DEFAULT
standard (default)   ebs.csi.aws.com      Yes
```

默认 StorageClass：

PVC 不写：

```yaml
storageClassName
```

自动用 default。

------

设置默认：

```yaml
metadata:
  annotations:
    storageclass.kubernetes.io/is-default-class: "true"
```

------

取消默认：

```yaml
"false"
```

------

注意：

只能一个默认。

多个默认可能行为异常。

------

# 五、创建 StorageClass

例：

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: standard
provisioner: nfs.csi.k8s.io
parameters:
  server: 192.168.1.10
  share: /k8s
reclaimPolicy: Retain
allowVolumeExpansion: true
volumeBindingMode: Immediate
```

应用：

```bash
kubectl apply -f sc.yaml
```

查看：

```bash
kubectl get sc
```

详情：

```bash
kubectl describe sc standard
```

------

# 六、PVC 如何使用 StorageClass

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: app-pvc
spec:
  accessModes:
  - ReadWriteOnce

  storageClassName: standard

  resources:
    requests:
      storage: 50Gi
```

创建：

```bash
kubectl apply -f pvc.yaml
```

检查：

```bash
kubectl get pvc
```

绑定：

```plaintext
STATUS=Bound
```

------

# 七、Pod 使用 PVC

```yaml
volumes:
- name: data
  persistentVolumeClaim:
    claimName: app-pvc

containers:
- name: app
  volumeMounts:
  - mountPath: /data
    name: data
```

挂载后：

容器 `/data`

就是动态卷。

------

# 八、StorageClass 常见故障

## 1、PVC Pending

原因：

Provisioner 不存在

检查：

```bash
kubectl get pods -n kube-system
```

CSI driver 是否运行。

------

## 2、参数错误

比如：

```yaml
type: gp10
```

查看：

```bash
kubectl describe pvc
```

会看到：

```plaintext
ProvisioningFailed
```

------

## 3、权限问题

云账号没权限创建盘：

```plaintext
AccessDenied
```

检查 IAM / RBAC。

------

## 4、扩容失败

CSI 不支持：

```plaintext
volume expansion not supported
```

------

# 九、生产最佳实践

## 数据库

```yaml
Retain
WaitForFirstConsumer
allowVolumeExpansion: true
```

------

## 缓存

```yaml
Delete
Immediate
```

------

## 多 AZ 集群

必须：

```yaml
WaitForFirstConsumer
```

------

## 开启监控

关注：

- PVC Pending
- Provision latency
- Volume attach latency

------

# 十、StorageClass vs PV vs PVC（彻底理解）

## StorageClass

定义：

怎么创建卷

像：

建筑规范

------

## PV

实际卷

像：

房子

------

## PVC

申请卷

像：

租房申请

------

关系：

```plaintext
PVC 请求
↓
StorageClass 按规则建 PV
↓
PV 绑定 PVC
↓
Pod 使用
```

所以：

StorageClass 是 Kubernetes 自动存储的核心入口。

没有它：

动态供给无法实现。

有了它：

K8s 才真正具备：

- 自动化
- 云原生
- 弹性存储
- 自服务申请存储

这也是生产环境几乎必配的能力。