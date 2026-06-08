**PV 上的 accessModes 和 PVC 上的 accessModes 含义并不完全一样。**

------

# 先说结论

## PV 的 accessModes

表示：

```text
这个存储本身支持什么访问模式
```

相当于：

```text
PV 的能力声明
```

例如：

```yaml
spec:
  accessModes:
  - ReadWriteMany
```

意思是：

```text
这个PV对应的后端存储
支持多个节点同时读写
```

------

## PVC 的 accessModes

表示：

```text
我需要什么访问模式
```

相当于：

```text
PVC 的需求声明
```

例如：

```yaml
spec:
  accessModes:
  - ReadWriteOnce
```

意思：

```text
我申请一个支持单节点读写的存储
```

------

# 类比理解

假设：

## 房东（PV）

房子支持：

```text
4个人住
```

即：

```yaml
PV
accessModes:
- RWX
```

------

## 租客（PVC）

需求：

```text
我只需要住1个人
```

即：

```yaml
PVC
accessModes:
- RWO
```

------

可以租：

```text
PV能力 ≥ PVC需求
```

所以：

```text
RWX
满足
RWO
```

能够绑定。

------

# Kubernetes是如何匹配的

PV：

```yaml
accessModes:
- ReadWriteMany
```

PVC：

```yaml
accessModes:
- ReadWriteMany
```

匹配成功。

------

PV：

```yaml
accessModes:
- ReadWriteMany
```

PVC：

```yaml
accessModes:
- ReadWriteOnce
```

通常也可以匹配。

因为：

```text
PV支持RWX
自然也支持RWO使用场景
```

------

但是：

PV：

```yaml
accessModes:
- ReadWriteOnce
```

PVC：

```yaml
accessModes:
- ReadWriteMany
```

不能绑定。

因为：

```text
PVC要求多个节点读写

PV做不到
```

------

# 常见误区

很多人以为：

```text
PVC的accessMode
会限制Pod访问
```

实际上不是。

------

例如：

PVC：

```yaml
accessModes:
- ReadWriteOnce
```

绑定：

```yaml
PV:
accessModes:
- ReadWriteOnce
```

------

然后创建：

```text
Node1
├── PodA
└── PodB
```

同时挂载同一个PVC。

这是允许的。

------

因为：

```text
RWO限制的是节点(Node)

不是Pod
```

------

# RWO到底是什么意思

官方定义：

```text
ReadWriteOnce
```

含义：

```text
只能被一个节点以读写方式挂载
```

不是：

```text
只能被一个Pod挂载
```

------

例如：

```text
Node1
├── PodA
├── PodB
├── PodC
```

全部使用同一个PVC

允许

```
---

但是：

```text
Node1
└── PodA

Node2
└── PodB
```

同时挂载：

```text
不允许
```

------

# 为什么要在PVC中声明accessMode

因为PVC不知道后端是什么存储。

可能是：

```text
NFS
CephFS
AWS EFS
AWS EBS
GCE PD
Local PV
```

能力完全不同。

------

PVC需要告诉调度器：

```text
我要什么类型的存储
```

例如：

```yaml
spec:
  accessModes:
  - ReadWriteMany
```

系统就知道：

```text
必须找支持RWX的PV
```

------

# 动态供给(StorageClass)时更明显

PVC：

```yaml
apiVersion: v1
kind: PersistentVolumeClaim

spec:
  accessModes:
  - ReadWriteMany

  storageClassName: nfs
```

------

Provisioner看到：

```text
RWX
```

就会创建：

```text
NFS
CephFS
EFS
```

这种共享存储。

------

如果：

```yaml
accessModes:
- ReadWriteOnce
```

可能创建：

```text
AWS EBS
阿里云ESSD
腾讯云CBS
本地磁盘
```

------

# 面试标准回答

**PV 的 accessModes 表示该存储资源支持的访问能力；PVC 的 accessModes 表示用户申请存储时要求的访问能力。**

绑定时 Kubernetes 会检查：

```text
PVC需求
≤
PV能力
```

才能成功绑定。

其中：

```text
RWO
限制的是节点(Node)

不是Pod
```

因此多个 Pod 位于同一个 Node 时，仍然可以同时挂载同一个 RWO 类型的 PVC；但多个 Node 不能同时以读写方式挂载同一个 RWO 存储。