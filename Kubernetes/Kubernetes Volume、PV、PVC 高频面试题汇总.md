# Kubernetes Volume、PV、PVC 高频面试题汇总

这部分几乎是 Kubernetes 存储面试的必考内容。

------

# 1. 什么是 Volume？

## 答案

Volume 是 Kubernetes 中 Pod 级别的存储卷。

用于解决：

```text
容器重启后数据丢失
多个容器共享数据
```

Volume 生命周期通常与 Pod 绑定。

例如：

```yaml
volumes:
- name: data
  emptyDir: {}
```

Pod 删除：

```text
Volume删除
数据丢失
```

------

# 2. Volume 和 Docker Volume 有什么区别？

## 答案

Docker Volume：

```text
容器级别
```

Kubernetes Volume：

```text
Pod级别
```

一个 Pod 内多个容器可共享同一个 Volume。

------

# 3. Kubernetes 常见 Volume 类型有哪些？

## 答案

### emptyDir

```text
临时目录
Pod删除数据消失
```

### hostPath

```text
宿主机目录映射
```

### ConfigMap

```text
配置文件挂载
```

### Secret

```text
密码证书挂载
```

### NFS

```text
网络共享存储
```

### PVC

```text
持久化存储
```

------

# 4. emptyDir 和 hostPath 区别？

## 答案

emptyDir：

```text
Pod创建时生成
Pod删除时删除
```

hostPath：

```text
直接使用宿主机目录
```

例如：

```yaml
hostPath:
  path: /data
```

Pod删除：

```text
数据仍在宿主机
```

------

# 5. hostPath 为什么不推荐生产使用？

## 答案

因为存在：

```text
Pod漂移
```

例如：

```text
Node1
  ↓
Pod删除

Node2
  ↓
重新调度
```

Node2 没有原来的数据。

------

# 6. NFS Volume 会不会因为 Pod 删除而丢失数据？

## 答案

不会。

例如：

```yaml
nfs:
  server: 192.168.1.100
  path: /data
```

数据存储在：

```text
NFS服务器
```

Pod删除：

```text
只是卸载
数据仍然存在
```

------

# 7. NFS 已经能持久化，为什么还需要 PV/PVC？

## 答案

NFS解决：

```text
数据持久化
```

PV/PVC解决：

```text
存储资源管理
容量管理
权限隔离
应用与存储解耦
```

------

# 8. 什么是 PV？

## 答案

PV（PersistentVolume）：

```text
集群级别存储资源
```

由管理员创建。

例如：

```text
CPU
Memory
Node
PV
```

都属于 Kubernetes Resource。

------

# 9. 什么是 PVC？

## 答案

PVC：

```text
PersistentVolumeClaim
```

表示：

```text
用户申请存储资源
```

PV：

```text
存储资源
```

PVC：

```text
存储申请单
```

------

# 10. PV 和 PVC 的关系？

## 答案

```text
PV ↔ PVC

1 : 1
```

一个 PVC 只能绑定一个 PV。

一个 PV 同时只能绑定一个 PVC。

------

# 11. Pod 与 PVC 的关系？

## 答案

```text
Pod → PVC

N : 1
```

多个 Pod 可以同时引用同一个 PVC。

例如：

```text
PodA
PodB
PodC
 ↓
shared-pvc
```

------

# 12. PVC 可以绑定多个 PV 吗？

## 答案

不能。

```text
PVC
 ↓
PV
```

只能是一对一。

------

# 13. 一个 PV 能绑定多个 PVC 吗？

## 答案

不能。

PV 一旦：

```text
Bound
```

就不能再被其他 PVC 使用。

------

# 14. Pod 删除后 PVC 会删除吗？

## 答案

不会。

例如：

```text
Pod
 ↓
PVC
 ↓
PV
```

删除：

```bash
kubectl delete pod xxx
```

结果：

```text
Pod 删除
PVC 保留
PV 保留
数据保留
```

------

# 15. PVC 删除后 PV 会删除吗？

## 答案

取决于：

```yaml
persistentVolumeReclaimPolicy
```

------

## Retain

```text
PVC删除
 ↓
PV保留
 ↓
数据保留
```

------

## Delete

```text
PVC删除
 ↓
PV删除
 ↓
存储删除
```

------

# 16. PVC 删除后，PV 能立即被新的 PVC 使用吗？

## 答案

不一定。

如果：

```yaml
persistentVolumeReclaimPolicy: Retain
```

PV状态会变成：

```text
Released
```

而不是：

```text
Available
```

不能自动重新绑定。

需要管理员处理。

------

# 17. PV 的状态有哪些？

## 答案

```text
Available
Bound
Released
Failed
```

------

## Available

```text
可绑定
```

------

## Bound

```text
已绑定
```

------

## Released

```text
PVC已删除
数据可能仍在
```

------

## Failed

```text
回收失败
```

------

# 18. 什么是 AccessMode？

## 答案

表示：

```text
存储访问方式
```

------

# 19. Kubernetes 支持哪些 AccessMode？

## 答案

### ReadWriteOnce（RWO）

```text
一个Node可读写
```

------

### ReadOnlyMany（ROX）

```text
多个Node只读
```

------

### ReadWriteMany（RWX）

```text
多个Node可读写
```

------

### ReadWriteOncePod（RWOP）

```text
整个集群仅一个Pod可使用
```

------

# 20. ReadOnlyOnce 存在吗？

## 答案

不存在。

Kubernetes 没有：

```text
ReadOnlyOnce
```

只有：

```text
ROX
RWO
RWX
RWOP
```

------

# 21. PV 的 AccessMode 表示什么？

## 答案

表示：

```text
PV支持什么能力
```

例如：

```yaml
accessModes:
- ReadWriteMany
```

表示：

```text
这个存储支持多个节点同时读写
```

------

# 22. PVC 的 AccessMode 表示什么？

## 答案

表示：

```text
用户需要什么能力
```

例如：

```yaml
accessModes:
- ReadWriteOnce
```

表示：

```text
我要申请RWO类型存储
```

------

# 23. PV 与 PVC 的 AccessMode 如何匹配？

## 答案

原则：

```text
PV能力 ≥ PVC需求
```

例如：

PV：

```text
RWX
```

PVC：

```text
RWO
```

可以绑定。

------

反之：

PV：

```text
RWO
```

PVC：

```text
RWX
```

不能绑定。

------

# 24. RWO 是不是只能一个 Pod 使用？

## 答案

不是。

RWO 限制的是：

```text
Node
```

不是：

```text
Pod
```

------

允许：

```text
Node1
├─ PodA
├─ PodB
└─ PodC
```

共同使用同一个 PVC。

------

# 25. RWOP 和 RWO 区别？

## 答案

RWO：

```text
一个节点可读写
多个Pod可共享
```

RWOP：

```text
整个集群
只能一个Pod使用
```

------

# 26. StatefulSet 为什么必须配合 PVC？

## 答案

数据库需要：

```text
Pod重建
数据不丢
```

StatefulSet 通过：

```yaml
volumeClaimTemplates
```

自动创建 PVC。

例如：

```text
mysql-0
 ↓
mysql-data-mysql-0

mysql-1
 ↓
mysql-data-mysql-1
```

Pod重建后：

```text
重新挂载原PVC
```

数据仍然存在。

------

# 27. 生产环境最常见的存储架构是什么？

## 答案

```text
StatefulSet
      ↓
PVC
      ↓
StorageClass
      ↓
PV
      ↓
NFS/CephFS/NAS/云盘
```

------

# 面试终极总结（一句话版）

```text
Volume：
Pod级存储

PV：
集群级存储资源

PVC：
存储申请

StorageClass：
动态创建PV

PV ↔ PVC = 1:1

Pod → PVC = N:1

RWO：
一个Node读写

RWX：
多个Node读写

RWOP：
一个Pod独占

删除Pod：
PVC不会删除

删除PVC：
是否删除PV取决于ReclaimPolicy

NFS解决数据持久化

PV/PVC解决存储资源管理与解耦
```

掌握上面 27 个问题，基本可以覆盖 Kubernetes 存储相关面试中 **80%~90% 的常见问题**。