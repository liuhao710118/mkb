参考链接：https://www.cnblogs.com/sexintercourse/p/18646560



学习 Elasticsearch（ES）可以从基础到高级，按照以下几个核心方面系统地构建知识体系。下表为你提供了一个清晰的概览。

| 学习方面              | 核心要点                                                     |
| --------------------- | ------------------------------------------------------------ |
| **1. 核心概念与架构** | 分布式架构、节点角色（Master, Data, Coordinating）、索引、文档、分片（Shard）与副本（Replica）、近实时（NRT）搜索 |
| **2. 数据建模与管理** | 映射（Mapping）与字段类型（如 `text`, `keyword`）、分词器（Analyzer）、索引的创建、配置和生命周期管理（ILM） |
| **3. 查询与搜索分析** | 查询DSL（Domain Specific Language）、精确查询与全文检索、复合查询（Bool）、聚合分析（Aggregation）、分页与排序 |
| **4. 性能优化与调优** | 写入优化（如Bulk API, 调整refresh_interval）、查询优化（使用filter缓存，避免深度分页）、索引设置与集群调优（JVM, 冷热架构） |
| **5. 运维监控与安全** | 集群健康状态监控（Green/Yellow/Red）、常见故障排查、快照备份与恢复、权限管理与安全配置（X-Pack） |
| **6. 生态集成与实践** | ELK/EFK技术栈（与Logstash, Kibana, Beats集成）、在日志分析、站内搜索、监控系统等场景下的实战应用 |



## 核心概念与架构

[分布式架构](./分布式架构.md)：es本质是分布式文档搜索与分析引擎, 支持 **水平扩展（Scale Out）** 高可用（HA） **高性能查询 + 实时写入**

[分片是什么](./分片是什么.md)：**分片是 Elasticsearch 中索引的最小数据单元**。一个索引在逻辑上是完整的，但在物理上会被**拆分**成多个分片，分布在不同节点上。

## 数据建模与管理

[映射（Mapping）](./映射Mapping.md):Elasticsearch 的 Mapping 类似于数据库中的表结构定义，它决定了数据如何被存储和索引

[(Document)文档是什么](./(Document)文档是什么.md)：**Document 是最小的数据单元，相当于数据库中的“一行记录”**

[索引(index)](./Elasticsearch的索引.md)：**Index 是 Elasticsearch 中用于存储一类 Document 的逻辑容器**