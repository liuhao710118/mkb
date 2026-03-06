# Kafka 学习全景路线图（超详细版）

## 一、入门认知（先搞清楚它是干嘛的）

### 1️⃣ Kafka 是什么

- Kafka 定位：**高吞吐、分布式消息队列**
- Kafka 解决什么问题
  - 系统解耦
  - 异步削峰
  - 日志收集
  - 流式处理
- Kafka vs Redis Stream / RabbitMQ / RocketMQ
- Kafka 适合 & 不适合的场景

👉 目标：
**能清楚说明：为什么用 Kafka，而不是别的 MQ**

------

## 二、基础使用（必须能跑起来）

### 2️⃣ Kafka 基本架构

- Broker
- Topic
- Partition
- Replica
- Leader / Follower
- ISR

必须理解：

- **为什么 Kafka 吞吐量高**
- **为什么要分区**

------

### 3️⃣ Kafka 环境搭建

- 单机 Kafka
- Kafka + ZooKeeper
- Kafka KRaft 模式（新趋势）
- Docker / Docker Compose
- 常用配置文件说明
  - server.properties
  - log.dirs

------

### 4️⃣ Kafka 基础操作

- Topic 管理
  - 创建 / 删除 / 扩容分区
- Producer
- Consumer
- Consumer Group
- Offset 基本概念

------

## 三、Kafka 核心机制（重中之重）

### 5️⃣ 消息存储原理

Kafka 为啥快？**答案在这里**

- 顺序写磁盘
- Page Cache
- Segment
- Index 文件
- Log Retention
  - 时间
  - 大小

------

### 6️⃣ Producer 深度机制

- 发送流程
- 批处理（batch）
- linger.ms
- buffer.memory
- acks（0 / 1 / all）
- retries & 幂等性

------

### 7️⃣ Consumer 深度机制

- poll 模型
- 心跳机制
- Rebalance
- Offset 提交
  - 自动提交
  - 手动提交
- at-least-once / at-most-once / exactly-once

------

## 四、可靠性 & 一致性（面试必问）

### 8️⃣ 消息可靠性

- 消息丢失的可能场景
  - Producer
  - Broker
  - Consumer
- ISR 机制
- 副本同步原理

------

### 9️⃣ 消息重复 & 顺序

- 为什么会重复？
- Kafka 如何保证顺序？
- 分区内有序，跨分区无序
- 幂等消费设计

------

## 五、高级特性（进阶）

### 🔟 Kafka 事务

- Producer 事务
- Exactly Once 原理
- 事务协调器

------

### 1️⃣1️⃣ Kafka Streams

- 流处理模型
- 状态存储
- Window
- 应用场景

------

### 1️⃣2️⃣ Kafka Connect

- Source / Sink
- 数据同步（MySQL → Kafka → ES）

------

## 六、集群 & 高可用（你运维优势）

### 1️⃣3️⃣ 集群架构

- 多 Broker
- 副本数设计
- Leader 选举
- Controller 角色

------

### 1️⃣4️⃣ ZooKeeper / KRaft

- ZooKeeper 在 Kafka 的作用
- KRaft 架构变化
- 迁移思路（了解）

------

### 1️⃣5️⃣ 扩容 & 运维

- Broker 扩容
- 分区重分配
- Topic 扩容注意事项
- Rolling Restart

------

## 七、性能调优（高级 & 实战）

### 1️⃣6️⃣ Producer 调优

- batch.size
- linger.ms
- compression.type
- acks
- max.in.flight.requests

------

### 1️⃣7️⃣ Consumer 调优

- max.poll.records
- fetch.min.bytes
- fetch.max.wait.ms
- poll 间隔问题

------

### 1️⃣8️⃣ Broker 调优

- 磁盘 IO
- 页缓存
- 网络线程
- JVM 参数

------

## 八、Kafka 常见故障 & 排查

### 1️⃣9️⃣ 常见问题

- 消息积压
- Rebalance 频繁
- 消费延迟
- Offset 异常
- Controller 抖动

------

### 2️⃣0️⃣ 监控 & 运维

- JMX
- Kafka Manager / Kafdrop
- Prometheus + Grafana
- 关键指标
  - Lag
  - TPS
  - ISR

------

## 九、Java 整合（你一定会用）

### 2️⃣1️⃣ Java 使用 Kafka

- 原生 Kafka Client
- Spring Kafka
- 序列化（JSON / Avro）
- 手动提交 Offset
- 事务消费

------

### 2️⃣2️⃣ 实战项目（强烈推荐）

- 日志收集系统
- 订单异步处理
- 用户行为埋点
- 延迟消息方案
- Kafka + Redis + MySQL

------

## 🔟 面试终极问题清单（必背）

你至少要能讲清楚：

- Kafka 为什么吞吐高？
- Kafka 如何保证消息不丢？
- Kafka 消费为什么会重复？
- Rebalance 为什么慢？
- Kafka 如何保证顺序？
- Kafka 和 RocketMQ 区别？
- Kafka 为什么不用随机 IO？

------

## 推荐学习顺序（照着来）

```
Kafka 基本概念
→ Producer / Consumer
→ 存储原理
→ 副本 & ISR
→ 消费组 & Rebalance
→ 可靠性 & 事务
→ 集群运维
→ 性能调优
→ Java 实战
```

