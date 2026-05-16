# StatefulSet 中的 Partition（分区更新）详解

在 StatefulSet 中：

```text
partition
```

是：

```text
滚动更新的“分段控制”
```

它允许：

```text
只更新部分 Pod
```

而不是：

```text
一次更新整个 StatefulSet
```

这是：

```text
生产环境数据库/中间件升级
非常重要的能力
```

尤其：

- Kafka
- Redis
- ZooKeeper
- Elasticsearch
- MySQL

这种：

```text
不能全量一起升级
```

的系统。

------

# 一、为什么需要 Partition

------

## 1. 普通 RollingUpdate 的问题

StatefulSet 默认：

```yaml
updateStrategy:
  type: RollingUpdate
```

更新时：

```text
按逆序更新
```

例如：

```text
mysql-2
↓
mysql-1
↓
mysql-0
```

------

## 2. 风险

某些场景：

```text
不能一次升级所有节点
```

例如：

- Kafka Broker
- Redis Cluster
- Elasticsearch

因为：

```text
可能导致集群不可用
```

------

## 3. Partition 的作用

实现：

```text
只升级部分 Pod
```

例如：

```text
先升级 mysql-2
观察
再升级 mysql-1
最后 mysql-0
```

------

# 二、Partition 工作原理

------

# 核心配置

```yaml
rollingUpdate:
  partition: N
```

含义：

```text
序号 >= N 的 Pod 才允许更新
```

而：

```text
序号 < N
不会更新
```

------

# 三、最核心理解（非常重要）

------

假设：

```text
replicas = 5
```

Pod：

```text
web-0
web-1
web-2
web-3
web-4
```

------

配置：

```yaml
partition: 3
```

则：

| Pod   | 是否更新 |
| ----- | -------- |
| web-0 | ×        |
| web-1 | ×        |
| web-2 | ×        |
| web-3 | √        |
| web-4 | √        |

即：

```text
只更新 3 和 4
```

------

# 四、完整 YAML 示例

------

```yaml
apiVersion: apps/v1
kind: StatefulSet

metadata:
  name: mysql

spec:
  serviceName: mysql-headless

  replicas: 5

  updateStrategy:
    type: RollingUpdate

    rollingUpdate:
      partition: 3

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
        image: mysql:8.4
```

------

# 五、更新过程详解

------

## 当前版本

假设：

```text
mysql:8.0
```

运行：

```text
mysql-0
mysql-1
mysql-2
mysql-3
mysql-4
```

------

## 修改镜像

改成：

```text
mysql:8.4
```

然后：

```bash
kubectl apply -f sts.yaml
```

------

## 由于 partition=3

只更新：

```text
mysql-4
↓
mysql-3
```

而：

```text
mysql-0
mysql-1
mysql-2
```

保持旧版本。

------

# 六、为什么 StatefulSet 逆序更新

更新顺序：

```text
4 → 3 → 2 → 1 → 0
```

原因：

```text
高编号通常是从节点
```

先更新：

```text
影响较小节点
```

最后：

```text
主节点
```

------

# 七、逐步灰度升级（生产核心）

------

## 第一步

```yaml
partition: 4
```

只更新：

```text
mysql-4
```

观察：

- 日志
- 集群状态
- 同步状态
- CPU
- 延迟

------

## 第二步

改成：

```yaml
partition: 3
```

更新：

```text
mysql-3
```

------

## 第三步

```yaml
partition: 2
```

继续更新。

------

## 最终

```yaml
partition: 0
```

全部升级。

------

# 八、Partition 与 Deployment 区别

Deployment：

```text
无法精确控制 Pod 序号更新
```

StatefulSet：

```text
可以按 Pod 编号更新
```

这是：

```text
有状态升级核心能力
```

------

# 九、Partition 更新机制

------

## StatefulSet Controller 判断

逻辑：

```text
pod ordinal >= partition
```

例如：

```text
partition = 2
```

则：

```text
2 3 4 更新
0 1 不更新
```

------

# 十、实际生产场景

------

## 1. Kafka 升级

Broker：

```text
broker-0
broker-1
broker-2
```

流程：

```text
先升级 broker-2
确认 ISR 正常
再升级 broker-1
最后 broker-0
```

------

## 2. Redis Cluster

先升级：

```text
slave
```

再：

```text
master
```

------

## 3. Elasticsearch

避免：

```text
所有节点同时重启
```

导致：

```text
脑裂
集群不可用
```

------

# 十一、查看更新状态

------

## 查看 Pod

```bash
kubectl get pods
```

------

## 查看镜像版本

```bash
kubectl get pod mysql-0 -o yaml
```

查看：

```yaml
image:
```

------

## 查看 rollout

```bash
kubectl rollout status sts/mysql
```

------

# 十二、Partition + OnDelete

------

也可以：

```yaml
updateStrategy:
  type: OnDelete
```

特点：

```text
必须手工删除 Pod
```

------

## 与 partition 区别

| 功能         | Partition | OnDelete |
| ------------ | --------- | -------- |
| 自动更新     | √         | ×        |
| 控制更新范围 | √         | ×        |
| 手工删除     | 不需要    | 必须     |

------

# 十三、Partition 与 PVC

Partition：

```text
只影响 Pod 更新
```

不会影响：

```text
PVC
数据
```

因此：

```text
数据不会丢
```

------

# 十四、Partition 更新失败

------

## 场景

如果：

```text
mysql-4 readiness 失败
```

则：

```text
更新停止
```

不会继续：

```text
mysql-3
mysql-2
```

------

## 这非常重要

因为：

```text
避免整个集群升级崩掉
```

------

# 十五、生产最佳实践

------

## 1. 数据库升级必须用 Partition

不要：

```text
一次更新整个 StatefulSet
```

------

## 2. 配合 readinessProbe

确保：

```text
节点真正 Ready
```

才继续升级。

------

## 3. 逐级降低 partition

推荐：

```text
4 → 3 → 2 → 1 → 0
```

逐步灰度。

------

## 4. Kafka/ES 强烈推荐

因为：

```text
集群对升级非常敏感
```

------

# 十六、常见问题

------

## 1. 为什么 Pod 不更新

检查：

```yaml
partition
```

可能：

```text
Pod 序号小于 partition
```

------

## 2. 为什么只更新部分节点

因为：

```text
partition 限制
```

------

## 3. partition=0 是什么

表示：

```text
全部允许更新
```

即：

```text
普通 RollingUpdate
```

------

## 4. partition 大于 replicas

例如：

```yaml
partition: 10
```

而：

```text
replicas: 5
```

则：

```text
不会更新任何 Pod
```

------

# 十七、面试高频问题

------

## 1. StatefulSet partition 是什么？

```text
滚动更新分区控制
```

------

## 2. partition=3 意味着什么？

```text
只有序号 >=3 的 Pod 更新
```

------

## 3. 为什么 Kafka 升级需要 partition？

因为：

```text
不能一次重启所有 Broker
```

------

## 4. StatefulSet 为什么逆序更新？

因为：

```text
高编号通常影响较小
```

------

## 5. partition=0 与普通 RollingUpdate 区别？

```text
没有区别
```

即：

```text
全部滚动更新
```

------

# 十八、总结

------

## Partition 本质

```text
StatefulSet 的灰度更新机制
```

------

## 核心规则

```text
ordinal >= partition
才允许更新
```

------

## 最重要用途

| 场景  | 原因            |
| ----- | --------------- |
| Kafka | Broker 逐步升级 |
| Redis | 主从灰度        |
| ES    | 避免集群震荡    |
| MySQL | 主从安全升级    |

------

## 生产核心策略

```text
partition:
4 → 3 → 2 → 1 → 0
```

实现：

```text
逐节点灰度升级
```

------

## 一句话理解

```text
Deployment：
整体滚动更新

StatefulSet partition：
按编号分批更新
```