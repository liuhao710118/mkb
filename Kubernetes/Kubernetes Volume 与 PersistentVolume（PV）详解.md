# Kubernetes Volume 与 PersistentVolume（PV）详解

这是 Kubernetes 存储体系中最核心的内容之一，也是面试高频考点。

很多初学者会混淆：

- Volume
- PersistentVolume（PV）
- PersistentVolumeClaim（PVC）
- StorageClass

实际上它们是逐步演进的关系：

```text
Volume
  ↓
PV + PVC
  ↓
StorageClass 动态供给
```

------

# 一、为什么需要 Volume

默认情况下，容器的数据是不持久化的。

例如：

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx
```

进入容器：

```bash
echo hello > /tmp/test.txt
```

删除 Pod：

```bash
kubectl delete pod nginx
```

重新创建：

```bash
kubectl apply -f nginx.yaml
```

文件消失。

因为：

```text
容器删除
↓
容器文件系统删除
↓
数据丢失
```

所以需要 Volume。

------

# 二、什么是 Volume

Volume 是：

```text
Pod级别的数据卷
```

用于：

```text
在 Pod 生命周期内保存数据
```

挂载过程：

```text
Volume
  ↓
Mount
  ↓
Container
```

示意图：

```text
Pod
├── Volume
│     └── data
│
├── Container A
│     └── /app/data
│
└── Container B
      └── /backup
```

两个容器共享同一个 Volume。

------

# 三、Volume特点

### 生命周期跟Pod一致

```text
Pod删除
↓
Volume删除
```

所以：

```text
Volume ≠ 持久化存储
```

这是很多人第一次学 Kubernetes 容易误解的地方。

------

# 四、Volume工作原理

当 Pod 调度到节点：

```text
Scheduler
↓
Node1
```

Kubelet：

```text
创建 Volume
↓
挂载到容器
```

Linux实际表现：

```bash
mount
```

或者：

```bash
bind mount
```

本质就是 Linux 挂载。

------

# 五、常见Volume类型

## emptyDir

最常用。

```yaml
volumes:
- name: cache
  emptyDir: {}
```

挂载：

```yaml
volumeMounts:
- name: cache
  mountPath: /cache
```

------

特点：

```text
Pod启动创建
Pod删除销毁
```

用途：

```text
缓存
临时文件
容器间共享数据
```

------

示意：

```text
Pod启动
↓
创建目录

/var/lib/kubelet/pods/xxx/volumes

↓
挂载到容器
```

------

## hostPath

直接挂宿主机目录。

```yaml
volumes:
- name: log
  hostPath:
    path: /data/log
```

------

挂载：

```yaml
volumeMounts:
- mountPath: /log
  name: log
```

------

效果：

```text
Node
└── /data/log

Pod
└── /log
```

------

特点：

```text
数据不会因为Pod删除而消失
```

因为数据在宿主机。

------

风险：

```text
Pod漂移
```

例如：

```text
Node1
↓
Pod删除

重新调度

↓
Node2
```

Node2没有：

```bash
/data/log
```

数据丢失。

所以生产环境不推荐。

------

## ConfigMap Volume

配置文件挂载。

```yaml
volumes:
- name: config
  configMap:
    name: nginx-config
```

------

容器：

```yaml
volumeMounts:
- mountPath: /etc/nginx
  name: config
```

------

效果：

```text
ConfigMap
↓
文件
↓
容器目录
```

------

## Secret Volume

挂载敏感信息。

```yaml
volumes:
- name: secret
  secret:
    secretName: mysql-secret
```

------

容器：

```yaml
volumeMounts:
- mountPath: /secret
  name: secret
```

------

效果：

```text
用户名
密码
证书
Token
```

变成文件。

------

## NFS Volume

共享存储。

```yaml
volumes:
- name: nfs-volume
  nfs:
    server: 192.168.1.100
    path: /data
```

------

多个Pod同时访问：

```text
Pod1
   \
    \
     NFS
    /
   /
Pod2
```

------

适合：

```text
共享文件
日志
上传文件
```

------

# 六、Volume核心字段

## volumes

定义存储。

```yaml
spec:
  volumes:
```

------

## volumeMounts

定义挂载。

```yaml
containers:
- volumeMounts:
```

------

完整例子：

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test
spec:
  volumes:
  - name: data

    emptyDir: {}

  containers:
  - name: busybox

    image: busybox

    volumeMounts:
    - name: data
      mountPath: /app/data
```

------

# 七、Volume的缺陷

Volume生命周期：

```text
跟Pod绑定
```

例如：

```text
Pod删除
↓
Volume删除
↓
数据丢失
```

生产数据库不能接受。

例如：

```text
MySQL
Redis
PostgreSQL
```

必须永久保存。

因此 Kubernetes 引入：

```text
PersistentVolume
(PV)
```

------

# 八、什么是PersistentVolume（PV）

PV：

```text
集群级别存储资源
```

由管理员提前创建。

类似：

```text
CPU
Memory
Node
```

属于集群资源。

------

对象关系：

```text
Node
Pod
Service
PV
```

都是 Kubernetes Resource。

------

# 九、PV核心思想

以前：

```text
Pod
 ↓
直接使用Volume
```

现在：

```text
Pod
 ↓
PVC
 ↓
PV
 ↓
NFS/云盘
```

------

示意图：

```text
Application

    ↓

PVC

    ↓

PV

    ↓

Storage
(NFS/云盘/CEPH)
```

------

# 十、PV与PVC

PV：

```text
实际存储资源
```

PVC：

```text
存储申请单
```

关系：

```text
PV ←→ PVC
```

类似：

```text
房子 ←→ 租房合同
```

------

# 十一、PV创建

例如NFS：

```yaml
apiVersion: v1
kind: PersistentVolume

metadata:
  name: pv-nfs

spec:

  capacity:
    storage: 10Gi

  accessModes:
  - ReadWriteMany

  nfs:
    server: 192.168.1.100
    path: /data
```

创建：

```bash
kubectl apply -f pv.yaml
```

查看：

```bash
kubectl get pv
```

结果：

```text
NAME     CAPACITY
pv-nfs   10Gi
```

------

# 十二、PVC创建

申请5Gi：

```yaml
apiVersion: v1
kind: PersistentVolumeClaim

metadata:
  name: pvc-test

spec:

  accessModes:
  - ReadWriteMany

  resources:
    requests:
      storage: 5Gi
```

创建：

```bash
kubectl apply -f pvc.yaml
```

------

系统自动匹配：

```text
PVC 5Gi
↓
PV 10Gi
↓
绑定成功
```

------

查看：

```bash
kubectl get pvc
STATUS
Bound
```

------

# 十三、Pod使用PVC

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: test

spec:

  volumes:
  - name: data

    persistentVolumeClaim:
      claimName: pvc-test

  containers:
  - name: nginx

    image: nginx

    volumeMounts:
    - mountPath: /data
      name: data
```

流程：

```text
Pod
↓
PVC
↓
PV
↓
NFS
```

------

# 十四、AccessMode详解

## ReadWriteOnce（RWO）

```text
一个节点读写
Node1
 └── PodA

Node2
 └── 不允许
```

最常见。

云盘基本都是：

```text
RWO
```

------

## ReadOnlyMany（ROX）

```text
多个节点只读
```

------

## ReadWriteMany（RWX）

```text
多个节点读写
```

例如：

```text
NFS
CephFS
GlusterFS
```

------

示意：

```text
Pod1
  \
   \
    Storage
   /
  /
Pod2
```

------

# 十五、Reclaim Policy（回收策略）

PV删除PVC后怎么办？

------

## Retain

保留数据

```text
PVC删除
↓
PV保留
↓
数据保留
```

生产推荐。

------

## Delete

自动删除

```text
PVC删除
↓
PV删除
↓
存储删除
```

云盘常见。

------

## Recycle（已废弃）

```text
rm -rf 数据
```

已经不建议使用。

------

# 十六、StorageClass（现代方案）

手工创建PV太麻烦：

```text
创建PV
创建PVC
绑定
```

所以出现：

```text
StorageClass
```

动态创建PV。

------

流程：

```text
Pod
 ↓
PVC
 ↓
StorageClass
 ↓
自动创建PV
 ↓
云盘/NFS
```

------

# 十七、面试高频问题

### Volume和PV区别

Volume：

```text
Pod级别
生命周期跟Pod一致
```

PV：

```text
集群级别
独立于Pod
持久化存储
```

------

### PV和PVC区别

PV：

```text
存储资源
```

PVC：

```text
存储申请
```

------

### PVC删除数据会丢吗

看：

```yaml
persistentVolumeReclaimPolicy
```

- Retain → 不丢
- Delete → 删除

------

### 为什么要PVC

解耦。

开发人员：

```text
我要10Gi存储
```

管理员：

```text
给你NFS
给你Ceph
给你云盘
```

双方不需要关心彼此实现。

------

# 最后用一句话总结

```text
Volume = Pod内部临时存储

PV = Kubernetes集群中的持久化存储资源

PVC = 用户申请存储的请求单

StorageClass = 自动创建PV的模板

关系：

Pod
 ↓
PVC
 ↓
PV
 ↓
NFS/CEPH/云盘
```

生产环境中，绝大多数有状态应用（MySQL、Redis、PostgreSQL、Kafka、MinIO 等）都会采用：

```text
StatefulSet
   +
PVC
   +
StorageClass
```

这是当前 Kubernetes 持久化存储的标准架构。