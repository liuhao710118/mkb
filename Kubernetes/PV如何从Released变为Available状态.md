这是 Kubernetes 存储面试中一个非常经典的问题。

------

# 为什么 PV 会变成 Released？

例如：

```text
PV(mysql-pv)
    ↓
PVC(mysql-pvc)
```

绑定成功后：

```text
STATUS = Bound
```

------

删除 PVC：

```bash
kubectl delete pvc mysql-pvc
```

PV 状态变成：

```text
STATUS = Released
```

查看：

```bash
kubectl get pv
```

结果：

```text
NAME       CAPACITY   STATUS
mysql-pv   10Gi       Released
```

------

# 为什么不会直接变成 Available？

因为 Kubernetes 不知道：

```text
PV里面的数据
是否应该保留
```

例如：

```text
MySQL数据
用户数据
订单数据
```

如果自动变成：

```text
Available
```

新的 PVC 可能直接绑定到旧数据。

存在数据泄露风险。

所以 Kubernetes 默认：

```text
Bound
 ↓
Released
```

而不是：

```text
Bound
 ↓
Available
```

------

# 如何把 Released 变成 Available？

取决于 Kubernetes 版本和 PV 配置。

------

# 方法1：删除 claimRef（最常见）

查看 PV：

```bash
kubectl get pv mysql-pv -o yaml
```

会看到：

```yaml
spec:
  claimRef:
    apiVersion: v1
    kind: PersistentVolumeClaim
    name: mysql-pvc
    namespace: default
```

虽然 PVC 已删除，

但 claimRef 仍然存在。

------

删除 claimRef：

```bash
kubectl edit pv mysql-pv
```

删除：

```yaml
claimRef:
  apiVersion: v1
  kind: PersistentVolumeClaim
  name: mysql-pvc
  namespace: default
```

保存退出。

------

或者：

```bash
kubectl patch pv mysql-pv \
-p '{"spec":{"claimRef": null}}'
```

------

此时：

```bash
kubectl get pv
```

可能看到：

```text
STATUS
Available
```

------

# 方法2：删除 PV 重新创建

生产环境经常这么做。

------

先删除：

```bash
kubectl delete pv mysql-pv
```

------

然后重新：

```bash
kubectl apply -f pv.yaml
```

重新创建后：

```text
STATUS = Available
```

------

这种方式最简单。

尤其 NFS PV。

------

# 方法3：StorageClass 动态供给

如果是：

```text
PVC
 ↓
StorageClass
 ↓
动态PV
```

通常不会复用旧 PV。

------

删除 PVC：

```text
PVC删除
 ↓
PV删除
 ↓
云盘删除
```

------

新 PVC：

```text
重新创建新PV
```

------

所以：

```text
Released → Available
```

基本不会出现。

------

# 实验演示

## 创建 PV

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: test-pv
spec:
  capacity:
    storage: 5Gi

  accessModes:
  - ReadWriteMany

  persistentVolumeReclaimPolicy: Retain

  nfs:
    server: 192.168.1.100
    path: /data
```

------

创建 PVC：

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: test-pvc
spec:
  accessModes:
  - ReadWriteMany

  resources:
    requests:
      storage: 1Gi
```

------

查看：

```bash
kubectl get pv
test-pv   Bound
```

------

删除 PVC：

```bash
kubectl delete pvc test-pvc
```

------

查看：

```bash
kubectl get pv
test-pv   Released
```

------

执行：

```bash
kubectl patch pv test-pv \
-p '{"spec":{"claimRef":null}}'
```

------

再次查看：

```bash
kubectl get pv
test-pv   Available
```

------

# 面试标准回答

> 当 PVC 删除且 PV 的 ReclaimPolicy 为 Retain 时，PV 会进入 Released 状态。此时 Kubernetes 不会自动将其变为 Available，因为底层数据可能仍然存在，直接复用可能导致数据泄露。
>
> 如果需要重新使用该 PV，通常需要管理员手动清理数据，并删除 PV 中的 `spec.claimRef` 字段（或删除后重建 PV），使其重新进入 Available 状态，然后才能被新的 PVC 绑定。

------

# 实际生产经验

对于：

- NFS
- CephFS
- 本地 PV

很多公司会采用：

```text
PVC删除
 ↓
PV Released
 ↓
管理员确认数据已清理
 ↓
删除 claimRef
 ↓
PV Available
```

对于云盘类存储（如阿里云 ESSD、AWS EBS、腾讯云 CBS），更多使用：

```text
ReclaimPolicy: Delete
```

直接销毁旧 PV 和磁盘，再创建新的存储，而不是复用 Released 的 PV。