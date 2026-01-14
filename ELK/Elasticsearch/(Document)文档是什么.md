## 一、什么是 Elasticsearch 文档（Document）

在 Elasticsearch 中：

> **Document 是最小的数据单元，相当于数据库中的“一行记录”**

- 一个 **Document = 一条 JSON 数据**
- Document 必须属于某个 **Index（索引）**
- Document 有唯一标识 **`_id`**

📌 类比关系：

| 关系型数据库 | Elasticsearch |
| ------------ | ------------- |
| Database     | Index         |
| Table        | Index（同）   |
| Row          | Document      |
| Column       | Field         |

------

## 二、Document 的基本结构

一个完整的 ES 文档在内部结构上包括：

```json
{
  "_index": "user_index",
  "_id": "1001",
  "_source": {
    "username": "liuh",
    "age": 30,
    "email": "liuh@example.com",
    "tags": ["ops", "devops"],
    "created_at": "2025-01-10T10:00:00"
  }
}
```

### 1️⃣ 元数据字段（Metadata Fields）

| 字段            | 说明             |
| --------------- | ---------------- |
| `_index`        | 所属索引         |
| `_id`           | 文档唯一 ID      |
| `_source`       | 原始 JSON 数据   |
| `_version`      | 版本号（乐观锁） |
| `_seq_no`       | 序列号           |
| `_primary_term` | 主分片任期       |

⚠️ `_source` 才是你**真正存储的数据**

------

## 三、Document 的字段类型（Field Types）

### 1️⃣ 常见字段类型

| 类型      | 示例             | 说明           |
| --------- | ---------------- | -------------- |
| `text`    | `"name": "刘浩"` | 分词、全文检索 |
| `keyword` | `"status": "OK"` | 精确匹配       |
| `integer` | `30`             | 数值           |
| `float`   | `3.14`           | 浮点           |
| `boolean` | `true`           | 布尔           |
| `date`    | `"2025-01-10"`   | 日期           |
| `object`  | `{}`             | 对象           |
| `nested`  | `[]`             | 嵌套文档       |

### 2️⃣ text vs keyword（重点）

```json
"name": {
  "type": "text",
  "fields": {
    "keyword": {
      "type": "keyword"
    }
  }
}
```

- `text`：用于搜索（会分词）
- `keyword`：用于聚合、排序、精确匹配

📌 **运维场景常用**

- 主机名、IP、状态 → `keyword`
- 日志内容、错误信息 → `text`

------

## 四、Document 的 Mapping（映射）

### 1️⃣ 什么是 Mapping

> Mapping 定义了 Document 中 **字段的类型、是否分词、是否索引**

相当于数据库的 **表结构**

------

### 2️⃣ 显式 Mapping 示例

```json
PUT user_index
{
  "mappings": {
    "properties": {
      "username": { "type": "keyword" },
      "age": { "type": "integer" },
      "email": { "type": "keyword" },
      "tags": { "type": "keyword" },
      "created_at": { "type": "date" }
    }
  }
}
```

📌 强烈建议：
**生产环境不要依赖动态 mapping**

------

### 3️⃣ Dynamic Mapping（自动）

```json
POST user_index/_doc
{
  "name": "liuh",
  "age": 30
}
```

ES 自动推断字段类型
⚠️ 风险：

- 字段爆炸
- 类型误判
- Mapping 无法修改

------

## 五、Document 的 CRUD 操作

### 1️⃣ 新增 Document

```json
POST user_index/_doc/1001
{
  "username": "liuh",
  "age": 30
}
```

或自动生成 ID：

```json
POST user_index/_doc
{
  "username": "liuh"
}
```

------

### 2️⃣ 查询 Document

```json
GET user_index/_doc/1001
```

------

### 3️⃣ 更新 Document

#### 局部更新（推荐）

```json
POST user_index/_update/1001
{
  "doc": {
    "age": 31
  }
}
```

#### 脚本更新

```json
POST user_index/_update/1001
{
  "script": {
    "source": "ctx._source.age += 1"
  }
}
```

------

### 4️⃣ 删除 Document

```json
DELETE user_index/_doc/1001
```

------

## 六、Document 的并发控制（重要）

### 1️⃣ 乐观锁（版本控制）

```json
POST user_index/_doc/1001?if_seq_no=10&if_primary_term=1
{
  "username": "liuh"
}
```

📌 用于：

- 防止并发写覆盖
- 高并发写场景

------

## 七、Document 与分片的关系

- Document **只存储在一个主分片**
- 通过 `_id` 哈希路由到分片
- 副本分片同步数据

📌 **分片一旦确定，Document 不能移动**

------

## 八、Document 的高级特性

### 1️⃣ Nested Document（嵌套）

```json
"servers": {
  "type": "nested",
  "properties": {
    "ip": { "type": "keyword" },
    "status": { "type": "keyword" }
  }
}
```

用于：

- 一对多
- 精确匹配子对象

------

### 2️⃣ Parent-Child（已不推荐）

- 性能差
- 查询复杂
- 多用于历史兼容

------

### 3️⃣ Source 过滤

```json
GET user_index/_search
{
  "_source": ["username", "age"]
}
```

------

## 九、Document 在运维 / 日志 / 监控中的设计建议

### 1️⃣ 日志 Document 示例

```json
{
  "timestamp": "2025-01-10T10:00:00",
  "level": "ERROR",
  "service": "order-service",
  "trace_id": "abc123",
  "message": "DB connection timeout"
}
```

### 2️⃣ 设计原则

✅ 字段扁平化
✅ keyword 控制基数
✅ 时间字段统一
✅ 预定义 mapping
❌ 避免超深嵌套
❌ 避免动态字段

------

## 十、一句话总结

> **Elasticsearch 文档 = 带类型的 JSON + 强索引 + 分布式存储**

