# Kubernetes StatefulSet 详解

在 Kubernetes 中：

```text
StatefulSet
```

是专门用于：

```text
有状态应用
```

的控制器。

它解决：

- Pod 身份固定
- Pod 名称固定
- Pod 存储固定
- 启动顺序控制
- 删除顺序控制
- 稳定网络标识

------

# 一、为什么需要 StatefulSet

------

## 1. Deployment 的问题

Deployment 适合：

```text
无状态应用
```

例如：

- Nginx
- SpringBoot
- API Gateway

因为：

```text
Pod 谁都一样
```

Pod 删除重建后：

```text
IP 变
名字变
存储丢失
```

无所谓。

------

## 2. 有状态应用的问题

例如：

- MySQL
- Redis Cluster
- Kafka
- ZooKeeper
- Elasticsearch

这些系统：

```text
每个节点都不一样
```

例如：

```text
mysql-0 = 主库
mysql-1 = 从库
```

它们要求：

| 需求        | Deployment 是否满足 |
| ----------- | ------------------- |
| 固定 Pod 名 | ×                   |
| 固定 DNS    | ×                   |
| 固定存储    | ×                   |
| 启动顺序    | ×                   |
| 稳定身份    | ×                   |

因此：

```text
Deployment 不适合
```

------

# 二、StatefulSet 核心能力

| 能力          | 说明            |
| ------------- | --------------- |
| 固定 Pod 名称 | mysql-0/mysql-1 |
| 固定 DNS      | 永久域名        |
| 固定存储      | PVC 不丢        |
| 顺序启动      | 按序号          |
| 顺序删除      | 逆序            |
| Pod 唯一身份  | Stateful        |

------

# 三、StatefulSet 架构

------

## 核心结构

```text
StatefulSet
      ↓
固定 Pod
      ↓
固定 PVC
      ↓
固定 DNS
```

------

## 与 Deployment 最大区别

Deployment：

```text
Pod 无身份
```

StatefulSet：

```text
Pod 有身份
```

------

# 四、固定 Pod 名称（最重要）

------

## StatefulSet Pod 命名规则

格式：

```text
<StatefulSet名称>-<序号>
```

例如：

```text
mysql-0
mysql-1
mysql-2
```

------

## 特点

即使：

```text
Pod 重建
```

名字仍然：

```text
不变
```

------

## 为什么重要

很多数据库：

```text
依赖节点身份
```

例如：

Kafka：

```text
broker.id
```

Redis：

```text
cluster node id
```

ZooKeeper：

```text
myid
```

------

# 五、固定 DNS（核心）

------

## 1. StatefulSet 必须配合 Headless Service

必须：

```yaml
clusterIP: None
```

------

## 2. DNS 规则

格式：

```text
pod-name.service-name.namespace.svc.cluster.local
```

------

## 示例

StatefulSet：

```text
mysql
```

Headless Service：

```text
mysql-headless
```

Namespace：

```text
default
```

------

则：

```text
mysql-0.mysql-headless.default.svc.cluster.local
mysql-1.mysql-headless.default.svc.cluster.local
mysql-2.mysql-headless.default.svc.cluster.local
```

永远固定。

------

# 六、固定存储（超重要）

------

## 1. Deployment 的问题

Deployment：

```text
Pod 删除
↓
数据丢失
```

------

## 2. StatefulSet 的解决方案

StatefulSet：

```text
每个 Pod 独立 PVC
```

------

# 七、volumeClaimTemplates（核心）

------

## 自动创建 PVC

```yaml
volumeClaimTemplates:
```

作用：

```text
为每个 Pod 自动创建 PVC
```

------

# 八、示例

```yaml
volumeClaimTemplates:
- metadata:
    name: data

  spec:
    accessModes:
    - ReadWriteOnce

    resources:
      requests:
        storage: 10Gi
```

------

# 九、自动生成 PVC

------

假设：

```text
StatefulSet: mysql
PVC模板名: data
```

则：

```text
data-mysql-0
data-mysql-1
data-mysql-2
```

自动创建。

------

# 十、固定存储效果

------

即使：

```text
mysql-0 Pod 删除
```

重新创建后：

```text
仍挂载原 PVC
```

因此：

```text
数据不会丢
```

------

# 十一、顺序启动（核心）

------

## 1. StatefulSet 启动顺序

严格：

```text
0 → 1 → 2
```

------

## 2. 启动逻辑

```text
mysql-0 Ready
    ↓
mysql-1 创建
    ↓
mysql-1 Ready
    ↓
mysql-2 创建
```

------

## 3. 为什么重要

数据库集群：

```text
需要主节点先启动
```

例如：

- MySQL 主从
- Kafka Controller
- ZooKeeper Leader

------

# 十二、顺序删除

------

## 删除顺序：

```text
2 → 1 → 0
```

逆序删除。

------

# 十三、StatefulSet YAML（完整）

------

## Headless Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mysql-headless
spec:
  clusterIP: None
  selector:
    app: mysql
  ports:
  - port: 3306
```

------

## StatefulSet

```yaml
apiVersion: apps/v1
kind: StatefulSet

metadata:
  name: mysql
spec:
  serviceName: mysql-headless
  replicas: 3
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:8
        volumeMounts:
        - name: data
          mountPath: /var/lib/mysql
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes:
      - ReadWriteOnce
      resources:
        requests:
          storage: 10Gi
```

------

# 十四、关键字段详解

------

## 1. serviceName（重点）

```yaml
serviceName: mysql-headless
```

作用：

```text
关联 Headless Service
```

用于：

```text
生成固定 DNS
```

------

## 2. replicas

```yaml
replicas: 3
```

创建：

```text
mysql-0
mysql-1
mysql-2
```

------

## 3. volumeClaimTemplates

自动生成：

```text
独立 PVC
```

------

# 十五、StatefulSet 更新策略

------

## 两种更新方式

| 策略          | 说明           |
| ------------- | -------------- |
| RollingUpdate | 滚动更新       |
| OnDelete      | 手工删除才更新 |

------

# 十六、RollingUpdate

默认：

```yaml
updateStrategy:
  type: RollingUpdate
```

更新顺序：

```text
2 → 1 → 0
```

逆序更新。

------

# 十七、OnDelete

```yaml
updateStrategy:
  type: OnDelete
```

特点：

```text
必须手工删除 Pod
```

才会更新。

适合：

```text
数据库慎重升级
```

------

# 十八、创建政策PodManagementPolicy

------

## 两种模式

| 模式         | 说明       |
| ------------ | ---------- |
| OrderedReady | 默认，顺序 |
| Parallel     | 并行       |

------

# 十九、OrderedReady

默认：

```yaml
podManagementPolicy: OrderedReady
```

特点：

```text
必须前一个 Ready
后一个才创建
```

------

# 二十、Parallel

```yaml
podManagementPolicy: Parallel
```

特点：

```text
并行创建 Pod
```

适合：

```text
无严格依赖
```

------

# 二十一、StatefulSet 与 Deployment 区别（重点）

| 对比     | Deployment | StatefulSet |
| -------- | ---------- | ----------- |
| Pod 名   | 随机       | 固定        |
| DNS      | 不固定     | 固定        |
| 存储     | 共享/临时  | 独立持久    |
| 顺序启动 | 无         | 有          |
| 顺序删除 | 无         | 有          |
| 适合     | 无状态     | 有状态      |

------

# 二十二、StatefulSet 删除行为（高频）

------

删除 StatefulSet：

```bash
kubectl delete sts mysql
```

默认：

```text
PVC 不会删除
```

这是：

```text
防止数据丢失
```

------

# 二十三、查看 StatefulSet

------

## 查看

```bash
kubectl get sts
```

------

## 查看 Pod

```bash
kubectl get pods
```

输出：

```text
mysql-0
mysql-1
mysql-2
```

------

## 查看 PVC

```bash
kubectl get pvc
```

输出：

```text
data-mysql-0
data-mysql-1
data-mysql-2
```

------

# 二十四、生产场景

------

## 1. MySQL 主从

```text
mysql-0 = master
mysql-1 = slave
```

------

## 2. Kafka

```text
broker-0
broker-1
broker-2
```

------

## 3. Redis Cluster

```text
redis-0
redis-1
redis-2
```

------

## 4. ZooKeeper

```text
zk-0
zk-1
zk-2
```

------

# 二十五、生产最佳实践

------

## 1. StatefulSet 必须搭配 Headless

```text
固定 DNS
```

------

## 2. 必须使用 PVC

不要：

```text
emptyDir
```

否则：

```text
数据丢失
```

------

## 3. 数据库推荐 OnDelete

避免：

```text
自动滚动升级
```

导致：

```text
集群异常
```

------

## 4. 一定加 Probe

特别：

```text
readinessProbe
```

否则：

```text
节点未初始化完成
就加入集群
```

------

# 二十六、常见问题

------

## 1. DNS 不生效

检查：

```yaml
serviceName
```

是否对应：

```text
Headless Service
```

------

## 2. PVC Pending

原因：

```text
没有 StorageClass
```

或：

```text
存储不足
```

------

## 3. Pod 卡住

常见：

```text
前一个 Pod 没 Ready
```

导致：

```text
后续 Pod 不创建
```

------

# 二十七、面试高频问题

------

## 1. StatefulSet 为什么需要 Headless？

因为：

```text
需要固定 DNS
```

------

## 2. StatefulSet 如何保证数据不丢？

```text
每个 Pod 独立 PVC
```

------

## 3. StatefulSet 与 Deployment 最大区别？

```text
StatefulSet 有状态身份
Deployment 无状态
```

------

## 4. StatefulSet 为什么顺序启动？

因为：

```text
数据库集群依赖初始化顺序
```

------

## 5. StatefulSet 删除后 PVC 会删吗？

默认：

```text
不会
```

------

# 二十八、总结

------

# StatefulSet 本质

```text
K8s 有状态应用控制器
```

------

# 核心能力

| 能力     | 说明       |
| -------- | ---------- |
| 固定身份 | Pod 名稳定 |
| 固定 DNS | Headless   |
| 固定存储 | PVC        |
| 顺序控制 | 启停有序   |

------

# 核心组合

```text
StatefulSet
+
Headless Service
+
PVC
```

------

# 最典型场景

| 系统          | 是否推荐 StatefulSet |
| ------------- | -------------------- |
| Kafka         | √                    |
| MySQL         | √                    |
| Redis Cluster | √                    |
| ZooKeeper     | √                    |
| Elasticsearch | √                    |

------

# 一句话理解

```text
Deployment：
Pod 都一样

StatefulSet：
每个 Pod 都有固定身份
```