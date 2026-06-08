严格来说，**Kubernetes 目前定义了 4 种 Access Mode**，但不是你写的那四种。

## Kubernetes 支持的 4 种 AccessMode

```yaml
ReadWriteOnce      (RWO)
ReadOnlyMany       (ROX)
ReadWriteMany      (RWX)
ReadWriteOncePod   (RWOP)
```

------

# 1. ReadWriteOnce（RWO）

## 含义

```text
一个节点(Node)可以读写挂载
```

注意：

```text
限制的是Node
不是Pod
```

例如：

```text
Node1
├── PodA
├── PodB
└── PodC
```

都挂载同一个 PVC：

```yaml
accessModes:
- ReadWriteOnce
```

允许。

------

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

常见存储：

- 云硬盘（EBS、ESSD、CBS）
- Local PV
- vSphere Volume

------

# 2. ReadOnlyMany（ROX）

## 含义

```text
多个节点可以挂载
但只能读
```

示意：

```text
Node1 -> Read
Node2 -> Read
Node3 -> Read
```

不能写。

------

适合：

```text
配置文件
软件仓库
镜像仓库缓存
静态资源
```

------

例如：

```yaml
accessModes:
- ReadOnlyMany
```

------

# 3. ReadWriteMany（RWX）

## 含义

```text
多个节点同时读写
```

示意：

```text
Node1 -> Read Write
Node2 -> Read Write
Node3 -> Read Write
```

------

例如：

```text
Web Pod
Upload Pod
Backup Pod
```

同时访问：

```text
/shared-data
```

------

常见存储：

- NFS
- CephFS
- GlusterFS
- EFS

------

# 4. ReadWriteOncePod（RWOP）

这是较新的模式（K8s 1.22+ 引入，后续版本稳定）。

------

## 含义

```text
整个集群只能有一个Pod挂载
```

注意：

它比 RWO 更严格。

------

RWO：

```text
一个节点多个Pod
允许
```

例如：

```text
Node1
├── PodA
└── PodB
```

允许同时使用。

------

RWOP：

```text
Node1
├── PodA
└── PodB
```

都不允许。

只能：

```text
Node1
└── PodA
```

唯一一个 Pod 使用。

------

适合：

```text
MySQL
PostgreSQL
Redis
单实例数据库
```

防止误挂载。

------

# 你写的两种模式实际上不存在

你提到：

```text
ReadOnlyOnce
ReadOnlyMany
ReadWriteOnce
ReadWriteMany
```

其中：

### ReadOnlyMany（ROX）

存在。

------

### ReadWriteOnce（RWO）

存在。

------

### ReadWriteMany（RWX）

存在。

------

### ReadOnlyOnce（ROO）

❌ Kubernetes 不存在这个模式。

因为：

```text
单节点只读
```

这种需求意义不大。

如果存储支持：

```text
ReadOnlyMany
```

自然也支持：

```text
单节点只读
```

没必要单独定义一个模式。

------

# 记忆口诀

```text
RWO
一个节点可读写

ROX
多个节点只读

RWX
多个节点可读写

RWOP
整个集群一个Pod独占
```

------

# 面试常考陷阱

面试官：

```text
RWO是不是只能一个Pod挂载？
```

正确回答：

```text
不是。

RWO限制的是Node，
不是Pod。

同一个Node上的多个Pod
可以同时挂载同一个RWO卷。

如果需要真正限制为
整个集群只有一个Pod使用，
应该使用RWOP(ReadWriteOncePod)。
```