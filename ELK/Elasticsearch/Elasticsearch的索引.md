## 一、什么是 Elasticsearch 索引（Index）

> **Index 是 Elasticsearch 中用于存储一类 Document 的逻辑容器**

- 一个 Index 包含 **多个 Document**
- Index 是 **分布式的**
- Index 本质是：
  - 多个 **Shard（分片）**
  - 每个 shard 是一个 **Lucene Index**

📌 类比关系：

| 关系型数据库 | Elasticsearch |
| ------------ | ------------- |
| Database     | Index         |
| Table        | Index         |
| Row          | Document      |

⚠️ ES 中 **没有 Table 的概念**

------

## 二、索引的内部结构（非常重要）

![Image](./assets/Elasticsearch%E7%9A%84%E7%B4%A2%E5%BC%95/13z9uZh68kT2kvTCaDFDB3g.png)

![Image](./assets/Elasticsearch%E7%9A%84%E7%B4%A2%E5%BC%95/61be38b25fdb7878686b93e0_605c9c8427508704ace5ef96_ES_shards.png)

![Image](./assets/Elasticsearch%E7%9A%84%E7%B4%A2%E5%BC%95/Lucene-based-index-structure.png)

### 1️⃣ 分片（Shard）

- **Primary Shard（主分片）**
- **Replica Shard（副本分片）**

```text
Index = N 个 Primary Shard
每个 Primary Shard = 0..M 个 Replica
```

📌 特点：

- Document 只写入 **一个主分片**
- 副本用于 **高可用 + 读性能**
- 主分片数 **创建后不可修改**

------

### 2️⃣ Shard 本质

- 每个 shard = 一个 Lucene index
- 独立文件、独立查询
- 查询时会并行打到所有 shard

------

## 三、索引的核心配置（Settings）

### 1️⃣ 分片与副本

```json
PUT logs_index
{
  "settings": {
    "number_of_shards": 3,
    "number_of_replicas": 1
  }
}
```

#### 经验值（生产建议）

| 场景            | 主分片               |
| --------------- | -------------------- |
| 小数据（<10GB） | 1                    |
| 日志索引        | 3–6                  |
| 超大索引        | 控制单 shard 20–50GB |

------

### 2️⃣ Refresh / Flush / Translog

| 机制     | 作用            |
| -------- | --------------- |
| refresh  | 数据可搜索      |
| flush    | translog → 磁盘 |
| translog | 写性能保障      |

```json
"refresh_interval": "30s"
```

📌 日志场景可以调大 refresh 提升写入性能

------

## 四、索引 Mapping（索引结构定义）

> Mapping 决定索引中 Document 的字段结构

```json
PUT logs_index
{
  "mappings": {
    "properties": {
      "timestamp": { "type": "date" },
      "level": { "type": "keyword" },
      "service": { "type": "keyword" },
      "message": { "type": "text" }
    }
  }
}
```

📌 Mapping 一旦字段创建，**不能修改类型**

------

## 五、索引的创建方式

### 1️⃣ 手动创建（推荐）

```json
PUT logs_index
{
  "settings": {...},
  "mappings": {...}
}
```

### 2️⃣ 自动创建（不推荐）

```json
PUT logs_index/_doc/1
```

⚠️ 易导致：

- mapping 混乱
- keyword/text 误判

------

## 六、索引模板（Index Template）

> 用于 **批量管理索引配置**

### 1️⃣ 模板示例（日志）

```json
PUT _index_template/logs_template
{
  "index_patterns": ["logs-*"],
  "template": {
    "settings": {
      "number_of_shards": 3
    },
    "mappings": {
      "properties": {
        "timestamp": { "type": "date" },
        "service": { "type": "keyword" }
      }
    }
  }
}
```

📌 日志 / 监控 **必用**

------

## 七、索引生命周期管理（ILM）

![Image](./assets/Elasticsearch%E7%9A%84%E7%B4%A2%E5%BC%95/0GvYRu4aTCcex92kr.png)

![Image](./assets/Elasticsearch%E7%9A%84%E7%B4%A2%E5%BC%95/architecture_1.png)

### 1️⃣ ILM 阶段

| 阶段   | 作用        |
| ------ | ----------- |
| hot    | 写入 + 查询 |
| warm   | 只查询      |
| cold   | 冷数据      |
| delete | 删除        |

### 2️⃣ ILM 策略示例

```json
PUT _ilm/policy/logs_policy
{
  "policy": {
    "phases": {
      "hot": {
        "actions": {
          "rollover": { "max_size": "50GB", "max_age": "7d" }
        }
      },
      "delete": {
        "min_age": "30d",
        "actions": { "delete": {} }
      }
    }
  }
}
```

------

## 八、索引别名（Alias）

> Alias 是 **索引的逻辑引用**

### 1️⃣ 用途

- 无感切换索引
- Rollover
- 蓝绿发布

```json
POST _aliases
{
  "actions": [
    { "add": { "index": "logs-2025.01.10", "alias": "logs_current" } }
  ]
}
```

------

## 九、索引性能优化要点（运维重点）

### 1️⃣ 分片控制（最重要）

- shard 过多 = CPU 爆
- shard 过大 = 恢复慢

📌 **经验法则**

> 单 shard 20–50GB 最佳

------

### 2️⃣ 字段设计

- 高基数字段慎用 `keyword`
- 禁用不必要字段：

```json
"enabled": false
```

------

### 3️⃣ 查询优化

- 避免 wildcard 前缀 `*abc`
- filter 替代 query
- 使用 doc_values

------

## 十、索引状态管理

| 状态      | 说明     |
| --------- | -------- |
| open      | 可读写   |
| close     | 不占资源 |
| read_only | 只读     |

```json
POST logs_index/_close
```

------

## 十一、生产索引命名规范（强烈建议）

```text
<业务>-<类型>-<日期>
logs-app-2025.01.10
metrics-node-2025.01
```

------

## 十二、一句话总结

> **Index = 分布式 Lucene 容器 + 生命周期 + 性能边界**

