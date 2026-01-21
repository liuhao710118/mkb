## 一、什么是 `_cat` API ？

`_cat`（**Compact And Aligned Text**）是一组 **面向人类阅读** 的 Elasticsearch 管理接口，主要特点：

- 📊 表格化输出（适合终端 / 运维）
- 🚀 快速查看集群、索引、分片、节点状态
- ❌ **不建议程序解析**（给程序用请用 JSON API）

示例：

```bash
GET /_cat/indices?v
```

------

## 二、通用参数（所有 `_cat` 都适用）

| 参数          | 说明                              |
| ------------- | --------------------------------- |
| `v`           | 显示表头（强烈建议）              |
| `h`           | 指定显示字段                      |
| `s`           | 排序字段                          |
| `bytes`       | 单位（b/kb/mb/gb）                |
| `time`        | 时间单位（ms/s/m/h）              |
| `format=json` | JSON 输出（**仍不推荐给程序用**） |

### 示例

```bash
GET /_cat/indices?v&s=store.size:desc&bytes=gb
```

------

## 三、最常用 `_cat` 接口详解 ⭐

## 1️⃣ `_cat/health` —— 集群健康状态

```bash
GET /_cat/health?v
```

| 字段         | 含义                 |
| ------------ | -------------------- |
| `status`     | green / yellow / red |
| `node.total` | 节点数               |
| `shards`     | 分片总数             |
| `unassign`   | 未分配分片数         |

> ⚠️ **yellow**：副本未分配
> ❌ **red**：主分片未分配（危险）

------

## 2️⃣ `_cat/nodes` —— 节点信息（非常常用）

```bash
GET /_cat/nodes?v
```

常用字段：

| 字段           | 含义         |
| -------------- | ------------ |
| `ip`           | 节点 IP      |
| `name`         | 节点名       |
| `heap.percent` | JVM 堆使用率 |
| `ram.percent`  | 内存使用率   |
| `cpu`          | CPU 使用率   |
| `node.role`    | 节点角色     |

只看关键指标：

```bash
GET /_cat/nodes?v&h=ip,name,heap.percent,ram.percent,cpu,node.role
```

------

## 3️⃣ `_cat/indices` —— 索引状态（你用得最多的）

```bash
GET /_cat/indices?v
```

核心字段：

| 字段             | 说明         |
| ---------------- | ------------ |
| `health`         | 索引健康     |
| `status`         | open / close |
| `index`          | 索引名       |
| `pri`            | 主分片数     |
| `rep`            | 副本数       |
| `docs.count`     | 文档数       |
| `store.size`     | 总占用       |
| `pri.store.size` | 主分片大小   |

只看大索引：

```bash
GET /_cat/indices?v&s=store.size:desc
```

------

## 4️⃣ `_cat/shards` —— 分片分布（排错必备）

```bash
GET /_cat/shards?v
```

关键字段：

| 字段     | 说明                 |
| -------- | -------------------- |
| `index`  | 索引                 |
| `shard`  | 分片号               |
| `prirep` | p / r                |
| `state`  | STARTED / UNASSIGNED |
| `node`   | 所在节点             |

查未分配分片：

```bash
GET /_cat/shards?v | grep UNASSIGNED
```

------

## 5️⃣ `_cat/allocation` —— 磁盘与分片分配

```bash
GET /_cat/allocation?v
```

| 字段         | 含义     |
| ------------ | -------- |
| `disk.used`  | 已用磁盘 |
| `disk.avail` | 可用磁盘 |
| `shards`     | 分片数   |

------

## 6️⃣ `_cat/thread_pool` —— 线程池状态（高并发排查）

```bash
GET /_cat/thread_pool?v
```

重点关注：

- `search`
- `write`
- `bulk`

```bash
GET /_cat/thread_pool/search?v
```

------

## 7️⃣ `_cat/pending_tasks` —— 集群任务阻塞

```bash
GET /_cat/pending_tasks?v
```

> 如果这里有大量任务，集群通常已不稳定

------

## 8️⃣ `_cat/plugins` —— 插件

```bash
GET /_cat/plugins?v
```

------

## 四、排障常用组合 💥

### 1️⃣ 集群变慢

```bash
/_cat/nodes?v
/_cat/thread_pool/search?v
/_cat/pending_tasks?v
```

### 2️⃣ 索引变红

```bash
/_cat/indices?v
/_cat/shards?v | grep UNASSIGNED
```

### 3️⃣ 磁盘快满

```bash
/_cat/allocation?v
/_cat/indices?v&s=store.size:desc
```

------

## 五、`_cat` vs JSON API（重要）

| 对比项   | `_cat`   | JSON API |
| -------- | -------- | -------- |
| 面向对象 | 人       | 程序     |
| 输出     | 表格文本 | JSON     |
| 稳定性   | ❌ 不保证 | ✅ 稳定   |
| 自动化   | ❌        | ✅        |

