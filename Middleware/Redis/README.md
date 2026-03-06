## 一、入门阶段（你现在从这里开始）

### 1️⃣ Redis 是什么 & 能干嘛

**一定要先搞清楚“为什么要用它”**

- Redis 定位：**内存型 KV 数据库**
- Redis 的典型场景
  - 缓存（最核心）
  - 分布式锁
  - 计数器 / 限流
  - 排行榜
  - 会话 Session
  - 延迟队列
- Redis vs MySQL
- Redis vs 本地缓存（Guava / Caffeine）

👉 这一阶段目标：
**能一句话说明 Redis 的价值**

------

### 2️⃣ Redis 基础使用

必须动手（别只看）

- Redis 安装（Linux / Docker）
- redis-server / redis-cli
- 基本配置项
  - bind / port
  - requirepass
  - daemonize
- 常用命令
  - keys / scan
  - ttl / expire
  - del / exists

------

## 二、核心重点（Redis 的灵魂）

### 3️⃣ 五大数据结构（超级重要）

这是 **80% 面试 + 80% 实战** 的来源

#### String

- set / get
- incr / decr
- 应用场景
  - 缓存对象
  - 计数器
  - 分布式 ID

#### Hash

- hset / hget
- 应用
  - 用户信息
  - 对象存储

#### List

- lpush / rpush / blpop
- 应用
  - 消息队列
  - 任务队列

#### Set

- sadd / smembers
- 应用
  - 去重
  - 关注 / 粉丝

#### ZSet（重点中的重点）

- zadd / zrange
- 应用
  - 排行榜
  - 延迟队列

⚠️ 必须掌握：

- 每种结构的 **时间复杂度**
- 适合什么、不适合什么

------

### 4️⃣ Redis 内部数据结构（进阶必看）

**面试官最爱问**

- SDS（字符串）
- dict（哈希表）
- ziplist / quicklist
- skiplist（ZSet 核心）
- intset

👉 不要求你写源码，但要：

- 知道 **为什么快**
- 知道 **什么时候会变慢**

------

## 三、缓存体系（Java 程序员必修）

### 5️⃣ 缓存设计 & 缓存问题

重点背 + 理解

- 缓存穿透
  - 空值缓存
  - Bloom Filter
- 缓存击穿
  - 互斥锁
- 缓存雪崩
  - 随机过期时间
- 热点 Key 问题
- 大 Key / 热 Key

------

### 6️⃣ 缓存一致性

**高级面试题必考**

- 先删缓存还是先更新数据库？
- 双删策略
- 延迟双删
- MQ 异步更新
- 最终一致性 vs 强一致性

------

## 四、Redis 高级特性

### 7️⃣ 持久化机制

Redis 为什么还能当数据库？

- RDB
  - 优点 / 缺点
- AOF
  - appendfsync 三种策略
- RDB + AOF 混合

------

### 8️⃣ 事务 & Lua

- Redis 事务
  - multi / exec
  - 为什么不是强事务
- Lua 脚本
  - 原子性
  - 分布式锁
  - 秒杀场景

------

### 9️⃣ 分布式锁（必会）

- setnx + expire
- set key value nx ex
- 为什么要 value
- 锁误删问题
- RedLock（知道就行）

------

## 五、运维 & 高可用（你优势区）

### 🔟 主从复制

- replication 原理
- 全量同步
- 增量同步
- 读写分离问题

------

### 1️⃣1️⃣ Sentinel

- 作用
- 选主流程
- 故障转移

------

### 1️⃣2️⃣ Redis Cluster

- 哈希槽（16384）
- 数据分片
- 扩容 & 缩容
- 跨 slot 问题

------

## 六、性能 & 调优（高级）

### 1️⃣3️⃣ 性能优化

- pipeline
- 批量操作
- 合理 Key 设计
- 内存优化
- maxmemory 策略
  - LRU / LFU

------

### 1️⃣4️⃣ 常见故障排查

- 内存暴涨
- QPS 下降
- Redis 阻塞
- 慢查询分析
- bigkey 排查

------

## 七、Java 整合（你可以直接实战）

### 1️⃣5️⃣ Java 使用 Redis

- Jedis vs Lettuce
- Spring Data Redis
- 序列化问题
- 连接池配置

------

### 1️⃣6️⃣ 实战项目（强烈建议）

- 秒杀系统
- 登录 Token
- 计数 + 排行榜
- 延迟队列
- 分布式限流

------

## 八、面试终极清单（背这个就够）

你要能回答：

- Redis 为什么快？
- Redis 单线程为什么还快？
- Redis 如何保证高可用？
- Redis 怎么解决缓存一致性？
- Redis 分布式锁安全吗？
- Redis 和 MQ 的区别？

------

## 学习顺序建议（照着来）

👉 **推荐顺序**

```
基础命令
→ 五大数据结构
→ 缓存问题
→ 持久化
→ 分布式锁
→ 主从 / Sentinel / Cluster
→ 性能调优
→ Java 整合 + 项目
```

------

