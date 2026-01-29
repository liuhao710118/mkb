# 什么是DSL

> DSL（Domain Specific Language）是 Elasticsearch 的 JSON 查询语言

特点：

- JSON 结构
- 可组合
- 支持全文检索 + 精确查询 + 聚合 + 排序



一个最基本的查询结构

```
GET logs_index/_search
{
  "query": {
    "match_all": {}
  }
}
```

完整的查询结构

```
{
  "query": {},        // 查询条件
  "from": 0,
  "size": 10,
  "sort": [],
  "_source": [],
  "aggs": {}
}

```



# 查询

## 查询全部数据 match_all

`match_all`是 Elasticsearch 查询语言（DSL）中最简单直接的一个查询语句。它的作用就像它的名字一样：**匹配所有文档**，也就是把索引里的数据，不加任何条件地全部查出来。

### 💡 通俗的理解

你可以把 Elasticsearch 的索引想象成一个文件夹里的所有文件。

- **普通查询（如 `match`）**：就像你带着一个具体问题去文件夹里找文件，比如“找出所有提到‘项目报告’的文件”。这是在**筛选**文件。
- **`match_all`查询**：则像是你简单地说：“把文件夹里所有文件都给我列出来”。它不关心文件内容是什么，只是要**全部**文件。

因此，`match_all`通常用于需要获取索引中完整数据列表的场景，比如数据预览、统计总数或与其他操作结合使用。

### 🛠️ 如何使用

它的基本语法非常简单：

```
GET /你的索引名/_search
{
  "query": {
    "match_all": {}
  }
}
```

你还可以轻松地配合其他参数使用，例如：

- **限制返回数量与分页**：使用 `size`和 `from`参数。

```
{
  "query": { "match_all": {} },
  "size": 5,  // 只返回5条结果
  "from": 10  // 跳过前10条结果，常用于分页
}
```

- **排序**：使用 `sort`参数。

```
{
  "query": { "match_all": {} },
  "sort": [ { "create_date": { "order": "desc" } } ] // 按创建日期降序排列
}
```

- **选择返回字段**：使用 `_source`参数指定需要返回的字段。

```
{
  "query": { "match_all": {} },
  "_source": ["title", "create_date"] // 只返回标题和创建日期字段
}
```

### ⚠️ 注意事项

1. **默认返回数量**：出于性能考虑，Elasticsearch 默认不会一次性返回所有结果。即使使用 `match_all`，默认也只返回前 **10** 条记录。如果需要更多，必须通过 `size`参数明确指定。
2. **谨慎处理大数据集**：如果索引中有数百万甚至更多文档，直接使用 `match_all`获取全部数据会对集群性能造成巨大压力，甚至导致问题。对于深度分页（例如想获取第10000页之后的数据），不建议使用 `from`/`size`，而应考虑使用 `scroll`或 `search_after`等更高效的方式。

### 💎 总结

简单来说，`match_all`就是 Elasticsearch 里的“**全选**”操作。它是你进行条件筛选前的基础工具，非常适合用来快速查看数据概貌或测试连接。但在实际业务搜索中，你更常使用的会是像 `match`、`term`或 `bool`这样的条件查询。

希望这个解释对你有帮助！如果你想了解如何进行条件搜索，我很乐意为你介绍 `match`查询等其他 DSL 语句。





## 精确匹配 term

`term`是 Elasticsearch 中用于**精确值匹配**的查询，类似于数据库里的等值查询。下面这个表格能帮你快速抓住核心，了解它和 `match`查询的区别。

| 特性         | `term`查询                                           | `match`查询                                |
| ------------ | ---------------------------------------------------- | ------------------------------------------ |
| **查询方式** | **精确匹配**，不分析搜索词，直接匹配索引中的词项     | **全文检索**，会先分析搜索词，然后进行匹配 |
| **适用场景** | 数字、日期、标签等**结构化数据**（如商品ID、状态码） | 文章内容、描述等**非结构化文本**           |
| **通俗理解** | **身份证识别**：必须完全一致才算匹配                 | **搜索引擎**：意思相关即可，允许部分匹配   |

### 💡 字段类型是关键

使用 `term`查询时，要特别留意目标字段的类型（`text`还是 `keyword`），这直接决定查询字符串是否需要和字段被索引的方式完全一致。

- **`keyword`类型字段**：字段值会被当作一个完整的整体存储。用 `term`查询时，搜索词必须和文档中的值**完全一致**（包括大小写和空格）才能匹配。
- **`text`类型字段**：字段值会被分词器拆分成多个独立的词项（如"颐和园路5号"会被分成"颐"、"和"、"园"等）。此时，若用 `term`查询去搜索完整的"颐和园路5号"，反而无法匹配，但搜索单个分词如"颐"则可以匹配。对于 `text`字段，若要精确匹配完整内容，通常使用其内置的 `.keyword`子字段。

### 🛠️ 如何使用

基本语法如下，你需要在查询中指定字段名和要精确匹配的值：

```
GET /your_index/_search
{
  "query": {
    "term": {
      "author": {
        "value": "J.K.罗琳"
      }
    }
  }
}
```

简写形式可以是：

```
{
  "query": {
    "term": {
      "author": "J.K.罗琳"
    }
  }
}
```

如果需要同时匹配多个精确值，可以使用 `terms`查询：

```
{
  "query": {
    "terms": {
      "tags": ["小说", "科幻", "经典"]
    }
  }
}
```

### 💎 总结与实用建议

简单来说，`term`查询是 Elasticsearch 里的“**精确查找**”工具。

在实际应用中，你可以参考以下建议：

- **精确匹配**时（如状态、ID、标签），优先考虑 `term`查询。
- **全文搜索**时（如文章、描述），应使用 `match`查询。
- 对 `text`字段进行精确匹配时，记得使用 `.keyword`。



## 全文检索 分词匹配 match

简单来说，Elasticsearch 中的 `match`查询就像是搜索引擎里的**智能模糊搜索**。它不会要求文档必须一字不差地包含你的搜索词，而是试图理解你的意图，找出那些意思上相关的文档，并聪明地给更匹配的文档更高的排名。

为了让你快速抓住精髓，下面这个表格清晰地对比了 `match`查询和另一种常用查询 `term`的核心区别。

| 特性           | `match`查询 (智能模糊搜索)                 | `term`查询 (精确匹配)                            |
| -------------- | ------------------------------------------ | ------------------------------------------------ |
| **查询方式**   | **全文检索**，会先分析搜索词，然后进行匹配 | **精确匹配**，不分析搜索词，直接匹配索引中的词项 |
| **是否分词**   | ✅ 是，对查询内容进行分词                   | ❌ 否，直接查询确切值                             |
| **相关性评分** | ✅ 计算 `_score`，用于结果排序              | 通常不计算（用于过滤语境时）                     |
| **适用场景**   | 文章标题、内容描述等**非结构化文本**       | 商品ID、状态码、标签等**结构化数据**             |
| **通俗理解**   | **搜索引擎**：意思相关即可，允许部分匹配   | **身份证识别**：必须完全一致才算匹配             |

### 🔍 工作原理四步走

`match`查询的工作方式很像一个标准的搜索引擎流程：

1. **检查字段类型**：首先确认你要搜索的字段是否是支持分词的文本类型。
2. **分析查询字符串**：将你输入的搜索内容（例如“宝马多少马力”）交给**分析器** 处理。分析器通常会做分词、转小写、去除标点停用词等操作，最终得到一系列词项，如“宝马”、“多少”、“马力”。
3. **查找匹配文档**：用这些分好的词项，去倒排索引中查找包含其中任何一个或多个词项的文档。
4. **为每个文档评分**：根据复杂的算法计算相关性评分 `_score`。一个文档包含的搜索词越多、词频越合理、字段越短，得分通常就越高，排名也越靠前。

### ⚙️ 核心参数与用法

`match`查询的强大之处在于它提供了参数让你控制匹配的松紧程度。

| 参数                       | 作用                                                         | 示例说明                                                     |
| -------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **`operator`**             | 控制分词后的逻辑关系。默认是 `or`，文档匹配任意一词即可；设为 `and`则要求文档必须包含所有词项。 | 搜索“宝马 马力”，`or`会返回包含“宝马”或“马力”的文档；`and`只返回同时包含两者的文档。 |
| **`minimum_should_match`** | 更灵活地控制匹配精度。可以指定一个最小匹配词项数或百分比。   | 搜索“quick brown dog”三个词，设置 `"75%"`（相当于至少匹配2个词），能平衡查全率和查准率。 |
| **`fuzziness`**            | 允许容错匹配，应对拼写错误。可以设置为一个可容忍的编辑距离。 | 搜索“john”，通过模糊匹配也可能找到“jon”或“jhon”的文档。      |

### 💡 实用建议

1. **字段类型是关键**：使用 `match`查询的字段通常应映射为 `text`类型，因为这类字段会被分词，适合全文搜索。如果你需要对一个字段同时进行全文搜索和精确匹配（如按产品名精确查询），可以在映射时为其同时设置 `text`类型和 `keyword`子字段。
2. **性能考量**：由于需要分词和计算相关性评分，`match`查询通常比作为过滤条件使用的 `term`查询开销更大。在需要进行频繁、严格的精确匹配或范围过滤时（如“状态为已发布”、“价格在100-200元之间”），可考虑使用 `filter`子句，因为它会缓存结果且不计算分数，效率更高。
3. **探索更多变体**：
   - **`match_phrase`**：用于精确匹配词组，词序和相对位置都需要符合要求。
   - **`multi_match`**：允许在多个字段中执行同一个 `match`查询。

Elasticsearch 的 `match`查询是进行全文搜索的首选工具，它懂得如何智能地处理文本。下面通过一个产品索引的例子，为你展示其核心用法。

### 📌 基础单字段匹配

最基本的用法是搜索单个字段。例如，在商品描述中查找包含 **"leather"** 或 **"jacket"** 的商品。由于 `operator`参数默认为 `or`，以下查询会匹配描述中出现任一词汇的商品。

```
GET /products/_search
{
  "query": {
    "match": {
      "description": "leather jacket"
    }
  }
}
```

### 🔍 控制匹配精度

当需要更精确的结果时，可以调整匹配规则。

- **要求包含所有词条**：使用 `"operator": "and"`，确保文档包含查询中的所有词汇。下面的查询要求文档必须同时包含 **"leather"**、**"jacket"**、**"winter"** 和 **"wear"** 才会被匹配。

```
GET /products/_search
{
  "query": {
    "match": {
      "description": {
        "query": "leather jacket winter wear",
        "operator": "and"
      }
    }
  }
}
```

- **灵活匹配词条数**：`minimum_should_match`参数更灵活，可以指定需要匹配的词条最小数量或百分比。例如，查询三个词时设置 `"75%"`（相当于至少匹配 `2`个），可以在查全率和查准率之间取得平衡。

```
GET /products/_search
{
  "query": {
    "match": {
      "description": {
        "query": "quick brown dog",
        "minimum_should_match": "75%"
      }
    }
  }
}
```

### ✨ 模糊匹配与高亮

`match`查询还能处理一些不精确的情况。

- **应对拼写错误**：通过 `fuzziness`参数开启模糊匹配，允许查询词与索引中的词存在一定差异（如拼写错误）。这可以找到类似 **"quik"** 或 **"quack"** 的文档。
- **高亮匹配内容**：使用 `highlight`功能，可以在返回结果中突出显示匹配到的关键词，提升用户体验。

```
GET /products/_search
{
  "query": {
    "match": {
      "description": "leather"
    }
  },
  "highlight": {
    "fields": {
      "description": {}
    }
  }
}
```

### ⚙️ 高级组合查询

在实际应用中，`match`查询经常与其他查询组合使用，构建复杂的搜索逻辑。例如，在 `bool`查询的 `must`子句中使用 `match`查询进行关键词搜索，同时在 `filter`子句中使用 `range`查询进行范围过滤（如价格、日期）。过滤器不影响相关性评分，但能高效地缩小结果集。

```
GET /products/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "description": "winter coat"
          }
        }
      ],
      "filter": [
        {
          "range": {
            "price": {
              "gte": 100,
              "lte": 300
            }
          }
        }
      ]
    }
  }
}
```

### 💎 核心要点速览

下表总结了 `match`查询的关键特性和适用场景，方便你快速回顾：

| 特性/场景    | 说明                                               | 常用参数值                             |
| ------------ | -------------------------------------------------- | -------------------------------------- |
| **全文搜索** | 对文本字段进行搜索，查询词会被分词。               | 默认行为                               |
| **匹配逻辑** | 控制词条间的逻辑关系。                             | `operator`: `or`（默认） / `and`       |
| **匹配精度** | 灵活控制需要匹配的词条数量。                       | `minimum_should_match`: 百分比或固定数 |
| **容错搜索** | 应对拼写错误或近似匹配。                           | `fuzziness`: 如 `"AUTO"`               |
| **结果排序** | 按相关性评分(`_score`)排序，更相关的文档排在前面。 | 自动计算                               |

`match`查询的强大之处在于它的智能和灵活。你可以根据具体的搜索需求，组合使用这些参数来精确控制搜索行为。

希望这些案例能帮助你更好地理解和使用 `match`查询。如果你对某个特定场景有更深入的疑问，或者想了解 `match_phrase`（短语匹配）等进阶用法，随时可以告诉我。



## 全文检索 短语匹配 match_phrase

------

### 一、一句话理解 match_phrase

> **`match_phrase` = “词必须按顺序、紧挨着（或几乎紧挨着）出现” 的全文搜索**

对比：

- `match`：词出现就行，顺序/距离不重要
- `match_phrase`：**顺序重要，位置重要**

------

### 二、最直观对比（先看这个）

#### 文档内容

```text
message: "connection timeout to mysql server"
```

#### match 查询

```json
{
  "query": {
    "match": {
      "message": "timeout mysql"
    }
  }
}
```

✅ 能匹配（词都出现了）

------

#### match_phrase 查询

```json
{
  "query": {
    "match_phrase": {
      "message": "timeout mysql"
    }
  }
}
```

❌ **匹配不到**
因为原文是：`timeout to mysql`
中间多了一个 `to`

------

### 三、match_phrase 的底层原理（重要）

#### 1️⃣ 分词（Analyzer）

```text
"timeout mysql"
→ ["timeout", "mysql"]
```

#### 2️⃣ 位置匹配（Position）

`match_phrase` 不只看词是否存在，还看：

- **顺序**
- **词与词之间的距离（position）**

------

### 四、slop：允许中间隔几个词（重点）

#### 默认（slop = 0）

```json
{
  "match_phrase": {
    "message": {
      "query": "timeout mysql",
      "slop": 0
    }
  }
}
```

❌ 不匹配 `timeout to mysql`

------

#### 设置 slop

```json
{
  "match_phrase": {
    "message": {
      "query": "timeout mysql",
      "slop": 1
    }
  }
}
```

✅ 匹配成功
👉 允许中间 **插入 1 个词**

📌 经验值：

- `slop: 1~3` 最常见
- slop 越大，性能越差

------

### 五、match_phrase vs match（核心区别）

| 对比项       | match | match_phrase |
| ------------ | ----- | ------------ |
| 是否分词     | ✅     | ✅            |
| 是否要求顺序 | ❌     | ✅            |
| 是否要求相邻 | ❌     | ✅（可 slop） |
| 精确度       | 低    | ⭐ 高         |
| 性能         | 快    | 较慢         |

------

### 六、match_phrase vs term（常见误区）

| 查询           | 用途                   |
| -------------- | ---------------------- |
| `term`         | **完全不分词**，精确值 |
| `match_phrase` | 分词 + 顺序匹配        |

#### 错误示例（新手常犯）

```json
{
  "term": {
    "message": "timeout mysql"
  }
}
```

❌ 永远匹配不到（message 是 text）

------

### 七、运维 / 日志场景实战（非常常用）

#### 1️⃣ 精确错误信息匹配

```json
{
  "query": {
    "match_phrase": {
      "message": "connection timeout"
    }
  }
}
```

适合：

- 固定错误文案
- 第三方组件报错

------

#### 2️⃣ bool + match_phrase（最佳实践）

```json
{
  "query": {
    "bool": {
      "filter": [
        { "term": { "service.keyword": "order" } },
        { "range": { "@timestamp": { "gte": "now-10m" } } }
      ],
      "must": [
        {
          "match_phrase": {
            "message": {
              "query": "connection timeout",
              "slop": 1
            }
          }
        }
      ]
    }
  }
}
```

👉 **先用 filter 缩小范围，再用 phrase 精确匹配**

------

### 八、中文 match_phrase 特别注意 ⚠️

#### 1️⃣ 中文分词器决定一切

```text
"订单创建失败"
```

不同分词器结果不同：

- IK_max_word：订单 / 创建 / 失败 / 创建失败 …
- IK_smart：订单 / 创建失败

📌 **match_phrase 是否命中，完全取决于分词结果**

------

#### 2️⃣ 中文建议

- 日志、错误码 👉 **结构化字段**
- 中文搜索 👉 调整 analyzer 或用 `keyword` + wildcard

------

### 九、什么时候该用 match_phrase？

✅ 使用场景：

- 固定错误信息
- 异常栈关键片段
- 业务关键短语
- 安全审计关键词

❌ 不适合：

- 模糊搜索
- 大文本随意匹配
- 高 QPS 场景（慎用）

------

### 十、一句话总结

> **match_phrase = “按顺序、按距离匹配的全文搜索”**
> **精确度高，性能比 match 略差**



## wildcard 模糊匹配 

> 主要用于keyword字段 在“不分词的字段”上做模糊匹配（像 Linux 的 `*`、`?`） 性能开销大（尤其前缀是 `*`）

### 一、一句话理解 wildcard

> **`wildcard` = 在“不分词的字段”上做模糊匹配（像 Linux 的 `*`、`?`）**

⚠️ 重点：

* **不分词**
* **主要用于 `keyword` 字段**
* **性能开销大（尤其前缀是 `*`）**

---

### 二、最基础用法

#### 1️⃣ 匹配包含某段字符串

```json
{
  "query": {
    "wildcard": {
      "message.keyword": {
        "value": "*timeout*"
      }
    }
  }
}
```

👉 匹配：

* `connection timeout`
* `read timeout from mysql`

---

#### 2️⃣ 通配符规则

| 符号 | 含义                    |
| ---- | ----------------------- |
| `*`  | 匹配 **0 个或多个字符** |
| `?`  | 匹配 **1 个字符**       |

```json
"value": "error-202?-*"
```

---

### 三、wildcard 的底层原理（为啥慢）

* `keyword` 字段是 **整串字符串**
* `wildcard` 会：

  * 扫描 term dictionary
  * 做 **逐字符匹配**
* **无法像 term 那样走倒排索引**

📉 性能排序（慢 → 快）：

```
wildcard (*xxx*)  >  wildcard (xxx*)  >  prefix  >  term
```

---

### 四、最危险的写法（生产事故来源）

❌ **前缀是 `*`**

```json
"*timeout"
"*error*"
```

原因：

* 不能利用索引前缀
* 几乎等于全表扫描

🚨 **高基数字段 + 大索引 = 集群抖动**

---

### 五、正确 & 推荐的用法（重点）

#### 1️⃣ 只在 keyword 字段上用

```json
"message.keyword"
"traceId"
"request_uri"
```

❌ 不要用在：

```json
"text" 字段
```

---

#### 2️⃣ 用 prefix 能不用 wildcard

```json
{
  "query": {
    "prefix": {
      "request_uri.keyword": "/api/order"
    }
  }
}
```

✅ 比 wildcard 快很多

---

#### 3️⃣ 结合 bool + filter 使用

```json
{
  "query": {
    "bool": {
      "filter": [
        { "term": { "service.keyword": "order" } },
        {
          "wildcard": {
            "request_uri.keyword": "/api/order/*"
          }
        }
      ]
    }
  }
}
```

👉 **不算分，能缓存，相对安全**

---

### 六、wildcard vs match vs match_phrase

| 查询           | 是否分词 | 顺序要求 | 性能 | 典型用途        |
| -------------- | -------- | -------- | ---- | --------------- |
| `match`        | ✅        | ❌        | 快   | 模糊全文        |
| `match_phrase` | ✅        | ✅        | 中   | 固定短语        |
| `wildcard`     | ❌        | 字符级   | 慢   | URI / ID / 路径 |

---

### 七、日志 / 运维常见场景

#### 1️⃣ URL / URI 匹配

```json
{
  "wildcard": {
    "request_uri.keyword": "/api/*/detail"
  }
}
```

---

#### 2️⃣ traceId / requestId

```json
{
  "wildcard": {
    "traceId": "*a3f9*"
  }
}
```

⚠️ 如果经常这么查，说明 **字段设计有问题**

---

#### 3️⃣ 文件路径

```json
{
  "wildcard": {
    "log_path.keyword": "/var/log/*/error.log"
  }
}
```

---

### 八、性能优化建议（非常重要）

#### ✅ 最佳实践

* 优先：

  1. `term`
  2. `prefix`
  3. `match`
* `wildcard` 作为 **兜底方案**
* 放在 `filter` 里
* 缩小时间范围

---

#### ❌ 不推荐

* `*xxx*` 大范围扫描
* 在 text 字段用 wildcard
* 高 QPS 场景使用

---

### 九、如果你 wildcard 用得很多，正确解法是…

#### 方案 1️⃣：mapping 设计优化

```json
"message": {
  "type": "text",
  "fields": {
    "keyword": { "type": "keyword" }
  }
}
```

---

#### 方案 2️⃣：用 ngram（替代 wildcard）

```json
"analyzer": "ngram_analyzer"
```

👉 查询快很多，但索引变大

---

### 十、一句话总结

> **wildcard = 不分词的字符串模糊匹配**
> **能不用就不用，能 prefix 就别 wildcard**



## prefix 前缀匹配

---

### 一、一句话理解 prefix

> **`prefix` = “字段值以某个字符串开头” 的查询**

类似：

* SQL：`LIKE 'xxx%'`
* 字符串：`startsWith("xxx")`

---

### 二、最基础用法

#### 1️⃣ keyword 字段（最常见、最推荐）

```json
{
  "query": {
    "prefix": {
      "request_uri.keyword": "/api/order"
    }
  }
}
```

👉 能匹配：

* `/api/order/create`
* `/api/order/detail/1`

---

#### 2️⃣ text 字段（不推荐）

```json
{
  "query": {
    "prefix": {
      "message": "error"
    }
  }
}
```

⚠️ 实际是对 **分词后的 term** 做前缀匹配
结果不可控、性能差，**基本不用**

---

### 三、prefix 的底层原理（为什么比 wildcard 快）

* 基于 **倒排索引的 term dictionary**
* 利用 **FST（Finite State Transducer）**
* **可以快速定位前缀范围**

📊 性能对比（快 → 慢）：

```
term  >  prefix  >  wildcard(xxx*)  >  wildcard(*xxx*)
```

👉 所以：

* **能 prefix，就别 wildcard**

---

### 四、prefix vs wildcard（重点对比）

| 对比项           | prefix | wildcard |
| ---------------- | ------ | -------- |
| 是否分词         | ❌      | ❌        |
| 是否支持中间匹配 | ❌      | ✅        |
| 是否支持前缀     | ✅      | ✅        |
| 性能             | ⭐ 快   | 慢       |
| 是否推荐         | ✅      | ⚠️ 谨慎   |

#### 等价但性能差异巨大

```json
prefix:   "abc"
wildcard:"abc*"
```

➡️ **结果一样，prefix 快很多**

---

### 五、运维 / 日志常见实战场景

#### 1️⃣ 接口 URI

```json
{
  "query": {
    "prefix": {
      "request_uri.keyword": "/api/order"
    }
  }
}
```

👉 **最经典使用场景**

---

#### 2️⃣ 文件路径

```json
{
  "query": {
    "prefix": {
      "log_path.keyword": "/var/log/nginx/"
    }
  }
}
```

---

#### 3️⃣ 业务 ID（有规律）

```json
{
  "query": {
    "prefix": {
      "order_id": "ORD2026"
    }
  }
}
```

---

### 六、prefix 的正确打开方式（最佳实践）

#### ✅ 放到 filter 中

```json
{
  "query": {
    "bool": {
      "filter": [
        { "prefix": { "request_uri.keyword": "/api/order" } },
        { "range": { "@timestamp": { "gte": "now-10m" } } }
      ]
    }
  }
}
```

* 不算分
* 可缓存
* 性能最好

---

#### ❌ 不推荐写法

```json
{
  "prefix": {
    "message": "error"
  }
}
```

原因：

* message 是 text
* 分词后前缀不稳定
* 查询结果不可预测

---

### 七、prefix 的限制 & 注意事项

1️⃣ **只能匹配开头**

* 想匹配中间？👉 wildcard / ngram

2️⃣ **大小写敏感**

* keyword 默认区分大小写
* 可用 normalizer 解决

3️⃣ **高基数字段要小心**

* 前缀太短（如 `"a"`）
  👉 命中太多 term，变慢

---

### 八、prefix + mapping 优化（进阶）

#### keyword + lowercase normalizer

```json
"settings": {
  "analysis": {
    "normalizer": {
      "lowercase_norm": {
        "type": "custom",
        "filter": ["lowercase"]
      }
    }
  }
}
```

```json
"request_uri": {
  "type": "keyword",
  "normalizer": "lowercase_norm"
}
```

➡️ 实现 **大小写不敏感 prefix**

---

### 九、一句话总结

> **prefix = 高性能的 startsWith 查询**
> **URI / 路径 / 规则 ID 场景首选**



## regex 正则匹配

### 一、一句话理解 regexp

> **`regexp` = 在 keyword 字段上，用正则表达式做字符级匹配**

⚠️ 关键词：

* **不分词**
* **正则匹配**
* **性能最差的一类查询**

---

### 二、最基础用法

```json
{
  "query": {
    "regexp": {
      "request_uri.keyword": {
        "value": "/api/.*/detail"
      }
    }
  }
}
```

👉 能匹配：

* `/api/order/detail`
* `/api/user/detail/1`

---

### 三、regexp 支持的正则（ES 版本）

ES 使用的是 **Lucene RegExp**（不是 PCRE）

#### 支持

* `.` 任意字符
* `.*` 任意多个字符
* `[a-z]`
* `|` 或
* `? + * {m,n}`

#### 不支持 / 有限制

* ❌ 反向引用
* ❌ 零宽断言（`(?=)`）
* ❌ 贪婪/懒惰控制
* ❌ lookahead / lookbehind

👉 **别拿 Java/Python 正则直接用**

---

### 四、regexp 为什么这么慢（核心）

#### 执行方式

* 遍历 **term dictionary**
* 对每个 term 执行 **状态机匹配**
* 无法利用倒排索引

📉 性能排序（慢 → 快）：

```
regexp  >  wildcard(*xxx*)  >  wildcard(xxx*)  >  prefix  >  term
```

---

### 五、最危险的正则写法（生产事故源头）

❌ **以 .* 开头**

```json
".*error"
".*timeout.*"
```

➡️ 几乎等于全索引扫描

---

❌ **过于宽泛的正则**

```json
".*"
"[a-z].*"
```

➡️ CPU 飙升

---

### 六、正确 & 可控的 regexp 用法（重点）

#### 1️⃣ 一定加前缀锚定

```json
{
  "regexp": {
    "request_uri.keyword": {
      "value": "/api/order/.*"
    }
  }
}
```

✅ **以确定前缀开始**
❌ 不要 `.*xxx`

---

#### 2️⃣ 放到 filter 里

```json
{
  "query": {
    "bool": {
      "filter": [
        {
          "regexp": {
            "request_uri.keyword": "/api/order/.+"
          }
        }
      ]
    }
  }
}
```

* 不算分
* 尽量减少影响

---

#### 3️⃣ 限制匹配数量

```json
"max_determinized_states": 10000
```

防止复杂正则炸 JVM

---

### 七、regexp vs wildcard vs prefix（怎么选）

| 场景          | 推荐       |
| ------------- | ---------- |
| 纯前缀        | `prefix`   |
| 前缀 + 任意   | `wildcard` |
| 多规则 / 分支 | `regexp`   |
| 模糊全文      | `match`    |

👉 **能不用 regexp 就不用**

---

### 八、运维 / 日志真实场景

#### 1️⃣ 匹配多种接口规则

```json
{
  "regexp": {
    "request_uri.keyword": "/api/(order|user)/.*"
  }
}
```

---

#### 2️⃣ 匹配版本号

```json
{
  "regexp": {
    "app_version.keyword": "v1\\.[0-9]+\\..*"
  }
}
```

---

#### 3️⃣ 日志路径

```json
{
  "regexp": {
    "log_path.keyword": "/var/log/.*/error\\.log"
  }
}
```

---

### 九、regexp 的替代方案（强烈建议）

#### ✅ 能 prefix 就 prefix

```json
prefix: "/api/order"
```

#### ✅ 能 wildcard 就 wildcard

```json
"/api/order/*"
```

#### ✅ 高频 regexp → 拆字段

```json
service=order
type=detail
```

---

#### 🚀 高级方案：ngram

* 查询快
* 索引变大
* 适合高 QPS 模糊匹配

---

### 十、一句话总结

> **regexp = 最强，但也是最慢、最危险的查询**
> **只在规则复杂、低频查询时使用**

## 多字段搜索 multi_match

简单来说，Elasticsearch 里的 `multi_match`查询，就像是**一次派出多个侦察兵，到不同的区域去搜索同一个目标**。只要任何一个侦察兵在任何一个区域里找到了目标，就算成功。

它的核心作用就是**允许你同时在多个字段中搜索同一个关键词**。

### 💡 直观理解：多字段搜索

为了让你快速抓住精髓，下面这个表格对比了 `multi_match`和基础的 `match`查询。

| 特性           | `match`查询 (单兵侦察)                     | `multi_match`查询 (多路出击)                                 |
| -------------- | ------------------------------------------ | ------------------------------------------------------------ |
| **搜索字段数** | **一个**                                   | **多个**                                                     |
| **适用场景**   | 明确知道要在哪个字段（如“商品名称”）中搜索 | 不确定信息在哪个字段，需要在多个字段（如“商品名称”、“描述”、“品牌”）中广泛查找 |
| **性能影响**   | 对查询性能影响相对较小                     | 参与查询的字段越多，对查询性能的影响通常越大                 |
| **通俗比喻**   | 派一个侦察兵去一个指定地点搜索             | 同时派出多个侦察兵到多个可能的地点搜索                       |

### 🛠️ 如何使用

它的基本语法很简单，你只需要指定要搜索的**关键词**（`query`）和**目标字段列表**（`fields`）。

```
GET /indexName/_search
{
  "query": {
    "multi_match": {
      "query": "搜索关键词", 
      "fields": ["字段1", "字段2", "字段3"] 
    }
  }
}
```

你还可以通过 `^`符号来提升某个字段的权重。例如，下面的查询表示，在 `title`字段中找到的结果比在 `content`字段中找到的结果更重要，相关性得分会更高。

```
{
  "query": {
    "multi_match": {
      "query": "苹果",
      "fields": ["title^3", "content"] // `title`字段的权重是`content`字段的3倍
    }
  }
}
```

### ⚙️ 高级模式：`type`参数

`multi_match`的强大之处在于它的 `type`参数，这个参数决定了多个字段的搜索结果如何合并和评分。以下是几种常见的类型：

| 类型                     | 通俗解释                                                     | 适用场景                                                     |
| ------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **`best_fields`** (默认) | **赢家通吃**。只使用**得分最高**的那个字段的分数作为最终得分。 | 关键词很可能只在一个字段中出现，你只想找到那个最匹配的字段所在的文档。 |
| **`most_fields`**        | **票数取胜**。将所有匹配字段的**分数合并**起来作为最终得分。 | 文档的不同字段有不同分词方式，需要综合考量。                 |
| **`cross_fields`**       | **化零为整**。将多个字段视为一个**大字段**进行搜索，要求每个搜索词项至少在某个字段中出现。 | 需要跨字段匹配完整信息，比如同时搜索“姓名”和“地址”来找到一个完整的人名。 |



### `multi_match`查询在不同类型字段上的表现和特点



**`multi_match`查询主要设计用于 `text`字段，但它实际上可以用于任何类型的字段**。不过，其**核心价值和典型应用场景确实集中在 `text`字段的全文搜索上**。

下面这个表格帮你快速了解 `multi_match`查询在不同类型字段上的表现和特点。

| 字段类型          | `multi_match`查询行为特点                                    | 典型应用场景与说明                                           |
| ----------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **`text`字段**    | ✅ **最主要的使用场景**。查询词会先被分词，然后在多个指定字段中进行**全文检索**，计算相关性得分（`_score`）。 | 在文章的`title`、`content`、`abstract`等多个文本字段中搜索同一个关键词。 |
| **`keyword`字段** | ⚠️ **行为改变**。查询词**不会被分词**，进行的是**精确匹配**。通常需要查询词与字段值完全一致才能匹配。 | 比如，在商品的`tags`（标签）字段中精确查找包含特定标签的商品。但这种情况更常直接使用 `term`查询。 |
| **数值/日期字段** | ⚠️ **通常不适用**。虽然语法允许，但对数字或日期进行“多字段匹配”的实际业务意义不大。 | 对这些类型的精确查找或范围过滤，使用 `term`或 `range`查询更为清晰和高效。 |

### 💎 总结与实用建议

简单来说，`multi_match`就是你进行**广泛、模糊搜索**时的利器。

在实际应用中，你可以参考以下建议：

- **明确搜索目标时**：如果很清楚要找的内容在哪个字段，优先使用 `match`查询，效率更高。
- **需要广泛搜索时**：当不确定信息存在哪里，需要在多个字段中查找时，使用 `multi_match`。
- **提升查询性能**：如果经常需要多字段搜索，可以考虑使用 `copy_to`参数将这些字段的内容拷贝到一个新的组合字段中，然后只需对这个新字段进行 `match`查询，这通常比 `multi_match`性能更好。



## 范围查询 range

Elasticsearch DSL 中的 `range`查询用于查找指定字段的值落在给定范围内的文档。这个功能在筛选数值、日期乃至某些字符串类型的数据时非常实用。

下表汇总了 `range`查询的核心参数和适用场景，方便你快速了解：

| 参数            | 含义                           | 适用于     |
| --------------- | ------------------------------ | ---------- |
| **`gt`**        | 大于（不包含边界值）           | 数字、日期 |
| **`gte`**       | 大于或等于（包含边界值）       | 数字、日期 |
| **`lt`**        | 小于（不包含边界值）           | 数字、日期 |
| **`lte`**       | 小于或等于（包含边界值）       | 数字、日期 |
| **`format`**    | 指定日期格式                   | 日期       |
| **`time_zone`** | 指定时区                       | 日期       |
| **`relation`**  | 指定范围查询与字段值的匹配关系 | 范围字段   |

### 🔢 查询数值范围

这是最直接的用法，常用于价格、年龄、分数等场景。

```
GET /products/_search
{
  "query": {
    "range": {
      "price": {
        "gte": 100,
        "lte": 200
      }
    }
  }
}
```

这个查询会找出所有价格在 100 到 200 之间（包含100和200）的商品。

### 📅 查询日期范围

日期范围查询功能强大，支持相对时间（如"现在"）和数学表达式。

**1. 基本日期查询**

使用具体的日期字符串，注意通过 `format`参数指定格式以保准确定性。

```
GET /logs/_search
{
  "query": {
    "range": {
      "timestamp": {
        "gte": "2022-01-01",
        "lte": "2022-01-31",
        "format": "yyyy-MM-dd"
      }
    }
  }
}
```

**2. 使用日期数学表达式**

Elasticsearch 支持灵活的日期计算，`now`表示当前时间。

- `now-1d/d`：昨天一整天（`-1d`减一天，`/d`四舍五入到天）
- `now/d`：今天

```
GET /website/_search
{
  "query": {
    "range": {
      "post_date": {
        "gte": "now-1d/d",
        "lt": "now/d"
      }
    }
  }
}
```

这个查询会返回最近一天发布的博客。

**3. 时区处理**

如果涉及具体时刻，使用 `time_zone`参数确保时间解读正确。

```
GET /website/_search
{
  "query": {
    "range": {
      "post_date": {
        "gte": "2018-01-01 00:00:00",
        "lte": "now",
        "format": "yyyy-MM-dd hh:mm:ss",
        "time_zone": "+08:00"
      }
    }
  }
}
```

### 🔤 查询字符串"范围"

你也可以对字符串类型（必须是 `keyword`类型）的字段进行范围查询，通常用于像版本号、等级（如"A"到"B"）这类有明确顺序的文本数据。

```
GET /user_info/_search
{
  "query": {
    "range": {
      "age_level": { // 该字段需为keyword类型
        "gte": "20",
        "lte": "30"
      }
    }
  }
}
```

### ⚙️ 高级参数

- **`boost`**：可以设置浮点数（如 `2.0`）来调整此查询条件在相关性评分中的权重。
- **`relation`**：适用于范围类型字段，指示范围查询如何匹配范围字段的值。有效值为 `INTERSECTS`（默认，相交）、`CONTAINS`（包含）、`WITHIN`（在...之内）。

### 💡 使用要点

1. **字段类型是关键**：确保要查询的字段是数值类型（如 `integer`、`float`）、日期类型（`date`）或适合范围比较的 `keyword`类型。
2. **日期边界注意舍入**：日期范围的包含与否与舍入规则有关。例如，`gt`会向上舍入到范围后的第一毫秒，而 `gte`会向下舍入到范围的第一毫秒。
3. **性能**：`range`查询在过滤条件下效率很高，尤其当它与 `bool`查询中的 `filter`子句结合使用时，因为过滤结果可以被缓存。



## 存在查询 exists

Elasticsearch DSL 中的 `exists`查询用于**查找指定字段存在非空值的文档**，也就是检查文档中是否包含某个字段，并且该字段的值不是 `null`或空数组 `[]`。

下表概括了 `exists`查询的核心特性和一个典型应用场景：

| 特性         | 说明                                                         |
| ------------ | ------------------------------------------------------------ |
| **核心用途** | 检查文档中是否存在某个字段且其值有效 。                      |
| **适用场景** | 数据质量检查、筛选有特定字段的文档、查找缺失字段的文档（需结合 `must_not`）。 |
| **必填参数** | `field`，指定要检查的字段名 。                               |

### 🔍 查询示例

**1. 基本查询**

查找 `user`字段存在的文档 ：

```
GET /_search
{
  "query": {
    "exists": {
      "field": "user"
    }
  }
}
```

**2. 查询字段不存在**

结合 `bool`查询的 `must_not`子句，可以找到 `user.id`字段不存在的文档 ：

```
GET /_search
{
  "query": {
    "bool": {
      "must_not": {
        "exists": {
          "field": "user.id"
        }
      }
    }
  }
}
```

### 💡 注意事项

使用 `exists`查询时，需要注意以下几点：

- **字段值为 `null`或空数组 `[]`时，不会被 `exists`查询匹配到** 。
- 空字符串（如 `""`）、包含 `null`和其他值的数组（如 `[null, "foo"]`）或在 mapping 中定义了自定义 null-value 的字段，**会被认为该字段是存在的** 。



## 组合查询（and/or/not/filter）

### 一、什么是 bool 查询（一句话版）

`bool` 查询就是 **把多个查询条件组合在一起**，并用
👉 **必须 / 应该 / 不能 / 过滤** 这四种“逻辑关系”来控制结果。

可以把它理解成 **SQL 里的 AND / OR / NOT / WHERE（不参与评分）**。

------

### 二、bool 查询的 4 个核心子句（重点）

| 子句       | 类似 SQL | 是否参与相关性评分 `_score` | 通俗解释                       |
| ---------- | -------- | --------------------------- | ------------------------------ |
| `must`     | AND      | ✅ 是                        | **必须满足**，而且会影响排序   |
| `should`   | OR       | ✅ 是                        | **可选满足**，满足越多越靠前   |
| `must_not` | NOT      | ❌ 否                        | **不能满足**                   |
| `filter`   | WHERE    | ❌ 否                        | **必须满足，但不算分**（最快） |

------

### 三、最基础 bool 查询示例

#### 1️⃣ must（必须满足）

```json
{
  "query": {
    "bool": {
      "must": [
        { "match": { "service": "order" } },
        { "match": { "level": "error" } }
      ]
    }
  }
}
```

**含义：**

- service 必须是 order
- level 必须是 error
- **同时满足（AND）**
- 会计算 `_score`

📌 常用于：**全文检索 + 排序**

------

#### 2️⃣ should（可选满足，OR）

```json
{
  "query": {
    "bool": {
      "should": [
        { "match": { "message": "timeout" } },
        { "match": { "message": "connection refused" } }
      ]
    }
  }
}
```

**含义：**

- 满足其中一个就行
- **满足越多，分数越高**

⚠️ 注意：

- 如果 **只有 should，没有 must/filter**
  - 至少匹配一个 should
- 如果 **有 must / filter**
  - should 就变成 **加分项**

------

#### 3️⃣ must_not（不能满足）

```json
{
  "query": {
    "bool": {
      "must_not": [
        { "match": { "status": "200" } }
      ]
    }
  }
}
```

**含义：**

- 排除 status=200 的文档
- 不参与评分

📌 常用于：**排除正常日志**

------

#### 4️⃣ filter（强烈推荐运维场景使用）

```json
{
  "query": {
    "bool": {
      "filter": [
        { "term": { "service.keyword": "order" } },
        { "range": { "@timestamp": { "gte": "now-1h" } } }
      ]
    }
  }
}
```

**特点：**

- 必须满足
- **不计算 `_score`**
- **支持缓存，性能最好**

📌 运维 / 日志 / 监控查询 **优先用 filter**

------

### 四、must vs filter（面试&实战必问）

| 对比         | must | filter |
| ------------ | ---- | ------ |
| 是否影响排序 | 是   | 否     |
| 是否计算分数 | 是   | 否     |
| 是否缓存     | 否   | 是     |
| 性能         | 较慢 | ⭐ 很快 |

👉 **结论：**

- **精确条件（状态、时间、ID）👉 filter**
- **全文搜索（message、desc）👉 must**

------

### 五、组合使用（真实生产查询）

### 场景：

> 查询 **最近 15 分钟**
> order 服务
> 错误日志
> 包含 timeout 或 slow

```json
{
  "query": {
    "bool": {
      "filter": [
        { "term": { "service.keyword": "order" } },
        { "range": { "@timestamp": { "gte": "now-15m" } } }
      ],
      "must": [
        { "match": { "level": "error" } }
      ],
      "should": [
        { "match": { "message": "timeout" } },
        { "match": { "message": "slow" } }
      ],
      "minimum_should_match": 1
    }
  }
}
```

### 关键点说明

- `filter`：快速缩小数据范围
- `must`：必须是 error
- `should`：错误类型加权
- `minimum_should_match: 1`：至少匹配一个 should

------

### 六、minimum_should_match（非常重要）

#### 1️⃣ 数字

```json
"minimum_should_match": 2
```

➡️ 至少命中 2 个 should 条件

------

#### 2️⃣ 百分比（大数据搜索常用）

```json
"minimum_should_match": "75%"
```

➡️ should 条件命中 75% 才返回

------

### 七、bool 嵌套（复杂查询必备）

```json
{
  "query": {
    "bool": {
      "must": [
        {
          "bool": {
            "should": [
              { "match": { "env": "prod" } },
              { "match": { "env": "staging" } }
            ]
          }
        }
      ],
      "must_not": [
        { "match": { "host": "test-host" } }
      ]
    }
  }
}
```

➡️ **(prod OR staging) AND NOT test-host**

------

### 八、运维 / 日志查询最佳实践（重点）

✅ **90% 场景模板**

```json
{
  "query": {
    "bool": {
      "filter": [
        { "term": { "service.keyword": "xxx" } },
        { "range": { "@timestamp": { "gte": "now-5m" } } }
      ],
      "must": [
        { "match": { "message": "error" } }
      ]
    }
  }
}
```

❌ **反例（性能差）**

```json
"must": [
  { "match": { "service": "xxx" } },
  { "match": { "@timestamp": "now-5m" } }
]
```

------

### 九、一句话总结

> **bool = 查询组合器**
> **filter 定范围，must 做全文，should 加权，must_not 排除**



# 排序

## 一、最基础的排序（单字段）

### 1️⃣ 按字段升序 / 降序

```json
{
  "query": {
    "match_all": {}
  },
  "sort": [
    { "age": { "order": "asc" } }
  ]
}
{
  "sort": [
    { "age": { "order": "desc" } }
  ]
}
```

- `asc`：升序（默认）
- `desc`：降序

------

## 二、多字段排序（最常用）

> **排序优先级从前到后**

```json
{
  "sort": [
    { "status": { "order": "asc" } },
    { "create_time": { "order": "desc" } }
  ]
}
```

含义：

1. 先按 `status` 排序
2. `status` 相同，再按 `create_time` 倒序

------

## 三、按 `_score` 排序（相关度）

### 1️⃣ 默认就是按 `_score desc`

```json
{
  "query": {
    "match": {
      "title": "elasticsearch"
    }
  }
}
```

### 2️⃣ 显式指定 `_score`

```json
{
  "sort": [
    { "_score": { "order": "desc" } }
  ]
}
```

⚠️ **注意**

- 一旦你指定了 `sort`，默认就 **不会再按 `_score` 排序**
- 如需相关度 + 时间排序，需显式写 `_score`

------

## 四、按时间排序（日志 / 运维最常见）

```json
{
  "sort": [
    { "@timestamp": { "order": "desc" } }
  ]
}
```

👉 日志分析、链路追踪、监控数据基本都这么用

------

## 五、缺失字段排序（missing）

当某些文档 **没有该字段** 时：

```json
{
  "sort": [
    {
      "age": {
        "order": "asc",
        "missing": "_last"
      }
    }
  ]
}
```

- `_first`：缺失值排最前
- `_last`：缺失值排最后（常用）

------

## 六、指定 unmapped 字段（防止报错）

当部分索引 **没有该字段**：

```json
{
  "sort": [
    {
      "age": {
        "order": "desc",
        "unmapped_type": "long"
      }
    }
  ]
}
```

👉 多索引查询（`index-*`）必备

------

## 七、字符串字段排序（keyword）

⚠️ **text 字段不能直接排序**

### ❌ 错误示例

```json
{
  "sort": [
    { "name": "asc" }
  ]
}
```

### ✅ 正确方式

```json
{
  "sort": [
    { "name.keyword": { "order": "asc" } }
  ]
}
```

------

## 八、脚本排序（高级，用得少）

```json
{
  "sort": [
    {
      "_script": {
        "type": "number",
        "script": {
          "lang": "painless",
          "source": "doc['price'].value * params.rate",
          "params": {
            "rate": 0.8
          }
        },
        "order": "desc"
      }
    }
  ]
}
```

⚠️ 性能开销大，**生产慎用**

------

## 九、geo 地理位置排序

```json
{
  "sort": [
    {
      "_geo_distance": {
        "location": {
          "lat": 39.9,
          "lon": 116.4
        },
        "order": "asc",
        "unit": "km"
      }
    }
  ]
}
```

------

## 十、与分页配合（from/size & search_after）

### 1️⃣ from / size（浅分页）

```json
{
  "from": 0,
  "size": 10,
  "sort": [
    { "create_time": "desc" }
  ]
}
```

### 2️⃣ search_after（深分页，推荐）

```json
{
  "size": 10,
  "sort": [
    { "create_time": "desc" },
    { "_id": "desc" }
  ],
  "search_after": ["2026-01-20T01:00:00", "abc123"]
}
```

👉 **search_after 必须配合唯一排序字段**

------

## 十一、运维 / 日志场景推荐排序模板

```json
{
  "sort": [
    { "@timestamp": "desc" },
    { "_id": "desc" }
  ]
}
```

------

## 十二、常见坑总结 ⚠️

| 问题              | 原因                   |
| ----------------- | ---------------------- |
| 排序报错          | text 字段直接排序      |
| 排序慢            | script / 非 doc_values |
| search_after 乱序 | 排序字段不唯一         |
| 多索引报错        | 未指定 `unmapped_type` |

------

# 分页

Elasticsearch DSL 提供了三种主要的分页方法，它们分别适用于不同的数据量和场景。下面这个表格能帮你快速把握它们的核心区别。

| 分页方式           | 实现原理                                                     | 优点                                     | 缺点                                                         | 适用场景                                                 |
| ------------------ | ------------------------------------------------------------ | ---------------------------------------- | ------------------------------------------------------------ | -------------------------------------------------------- |
| `from + size`  | 类似于数据库的 `LIMIT offset, size`。协调节点从每个分片获取 `from + size`条数据，合并排序后，再取从 `from`开始的 `size`条数据返回。 | 使用直观，可实现随机跳页。               | **深度分页性能差**，存在 `index.max_result_window`限制（默认10,000条）。 | 数据量小、顶部页码的简单分页。                           |
| `scroll`       | 首次查询创建数据**快照**，并返回一个 `scroll_id`。后续分页通过此 `scroll_id`按顺序获取数据，直至遍历完毕。 | 适合**深度遍历**大量数据，性能较稳定。   | 基于历史快照，**数据非实时**；占用资源直至超时释放；**只能顺序翻页**。 | 离线数据处理、全量导出、数据迁移等非实时性任务。         |
| `search_after` | 利用上一页结果中**最后一条记录的排序值**，作为下一页的查询起点。需要指定一个或多个唯一性字段作为排序依据。 | 适用于**深度分页的实时查询**，性能较好。 | **不支持随机跳页**，必须连续翻页；排序字段组合需能唯一确定一条记录。 | 需要深度分页且要求结果实时性的场景，如实时数据流式读取。 |

## 如何选择分页方式

你可以根据以下原则来选择最合适的分页方法：

- **数据量小、简单翻页**：使用 **`from + size`**。但务必留意 `index.max_result_window`的限制。
- **深度分页，且需要结果实时性**（如用户不断下拉加载）：使用 **`search_after`**。这是处理深度分页的推荐方式。
- **非实时批量处理大量数据**（如数据导出、索引备份）：使用 **`scroll`** 。



## from + size 

```
from：从第几条结果开始（偏移量，offset）
size：返回多少条结果（page size）
```

公式

```
from = (page_no - 1) * page_size
size = page_size
```



基本案例

```
GET my_index/_search
{
  "from": 0,
  "size": 10,
  "query": {
    "match_all": {}
  }
}
```

分页返回结果

```
{
  "hits": {
    "total": {
      "value": 1256,
      "relation": "eq"
    },
    "hits": [
      {
        "_id": "1",
        "_source": {
          "name": "test",
          "status": "success"
        }
      }
    ]
  }
}
```

| 字段               | 含义           |
| ------------------ | -------------- |
| `hits.total.value` | 总记录数       |
| `from + size`      | 当前页数据     |
| `hits.hits`        | 实际返回的数据 |



`from+size`限制

默认最大只能查 **10000 条**

```bash
from + size <= index.max_result_window
```

默认配置

```
"index.max_result_window": 10000
```

❌ 超过会报错：

```
Result window is too large
```



## search_after

### 一、为什么要用 `search_after`

#### 1️⃣ from + size 的问题

```json
{
  "from": 100000,
  "size": 10
}
```

**问题：**

* ES 必须先取出 `from + size` 条数据再丢弃前面的
* 深分页（>1w）性能急剧下降
* 默认 `index.max_result_window = 10000`

👉 **不适合大数据量分页**

---

#### 2️⃣ scroll 的问题

* 适合 **离线全量导出**
* 长时间占用资源
* 不适合用户实时查询

---

#### 3️⃣ search_after 的定位

✅ **实时分页**
✅ **无最大页数限制**
✅ **性能稳定**
❌ 不能跳页（只能“下一页”）

---

### 二、search_after 的核心思想

> **用“上一页最后一条文档的排序值”作为下一页的起点**

📌 不再告诉 ES「跳过多少条」
📌 而是告诉 ES「从哪条之后开始查」

---

### 三、基本使用示例（最重要）

#### 1️⃣ 第一页（不带 search_after）

```json
POST my_index/_search
{
  "size": 5,
  "query": {
    "match_all": {}
  },
  "sort": [
    { "timestamp": "asc" },
    { "_id": "asc" }
  ]
}
```

**返回结果（重点看 sort）**

```json
{
  "hits": {
    "hits": [
      {
        "_id": "a1",
        "sort": [1700000000000, "a1"]
      },
      {
        "_id": "a2",
        "sort": [1700000001000, "a2"]
      }
    ]
  }
}
```

---

#### 2️⃣ 第二页（带 search_after）

```json
POST my_index/_search
{
  "size": 5,
  "query": {
    "match_all": {}
  },
  "sort": [
    { "timestamp": "asc" },
    { "_id": "asc" }
  ],
  "search_after": [1700000001000, "a2"]
}
```

📌 `search_after` 的值 **必须与 sort 一一对应**

---

### 四、为什么一定要加 `_id`？

#### ❌ 错误写法

```json
"sort": [{ "timestamp": "asc" }]
```

**问题：**

* timestamp 可能重复
* 下一页会 **丢数据 / 重复数据**

---

#### ✅ 正确写法（唯一排序）

```json
"sort": [
  { "timestamp": "asc" },
  { "_id": "asc" }
]
```

规则：

* **最后一个排序字段必须唯一**
* 常用：`_id` / 业务主键

---

### 五、search_after 的完整执行流程

```
第一页
 └── 返回 sort = [1000, "a2"]
        ↓
第二页
 └── search_after = [1000, "a2"]
        ↓
第三页
 └── search_after = [1005, "b1"]
```

👉 类似数据库的 **游标（cursor）**

---

### 六、search_after vs scroll 对比

| 特性           | search_after | scroll   |
| -------------- | ------------ | -------- |
| 是否实时       | ✅            | ❌        |
| 是否支持深分页 | ✅            | ✅        |
| 是否占资源     | ❌            | ✅        |
| 是否可跳页     | ❌            | ❌        |
| 使用复杂度     | 中           | 高       |
| 推荐场景       | 用户查询     | 全量导出 |

---

### 七、常见坑（非常重要）

#### 1️⃣ sort 变化会导致分页错乱

```json
sort: [{ "timestamp": "desc" }]
```

下一页必须 **完全一致**

---

#### 2️⃣ 查询条件必须完全一致

❌ 不能翻页时改 query

---

#### 3️⃣ 不能和 from 同时使用

```json
❌ from + search_after （非法）
```

---

#### 4️⃣ 数据实时写入导致结果不稳定（可选解决）

如果你要 **绝对一致性分页**：

```json
"pit": {
  "id": "PIT_ID",
  "keep_alive": "1m"
}
```

（ES 7.10+ 推荐）

---

### 八、search_after + PIT（生产级推荐）

#### 创建 PIT

```json
POST /my_index/_pit?keep_alive=1m
```

#### 使用 PIT 分页

```json
POST /_search
{
  "size": 5,
  "pit": {
    "id": "PIT_ID",
    "keep_alive": "1m"
  },
  "sort": [
    { "timestamp": "asc" },
    { "_id": "asc" }
  ],
  "search_after": [1700000001000, "a2"]
}
```

📌 避免分页过程中数据新增/删除影响结果

---

### 九、Java / Python 使用示例（简化）

#### Java（RestHighLevelClient 思路）

```java
searchSourceBuilder.searchAfter(lastSortValues);
```

---

#### Python

```python
resp = es.search(
    index="my_index",
    size=5,
    sort=["timestamp:asc", "_id:asc"],
    search_after=last_sort
)
```

---

### 十、什么时候**必须**用 search_after？

✅ 日志查询
✅ APM / Trace 查询
✅ 运维平台时间线
✅ 大表分页
❌ 随机跳页（不适合）

---

### 十一、一句话总结

> **from + size 是“偏移量分页”，search_after 是“游标分页”**



## scorll

> **scroll 是干什么的、该不该用、怎么正确用**

---

### 一、scroll 是用来解决什么问题的？

#### scroll 的核心定位

> **稳定地、批量地遍历大量数据（全量扫描）**

📌 不是分页
📌 不是实时查询
📌 是 **“快照式游标扫描”**

---

#### 什么时候用 scroll？

✅ 全量数据导出
✅ 离线分析 / ETL
✅ 数据迁移
✅ Reindex 底层实现

---

#### 什么时候**不**要用 scroll？

❌ 用户列表分页
❌ 实时查询
❌ API 接口分页

---

### 二、scroll 的工作原理（非常重要）

#### 1️⃣ 创建查询快照（Snapshot）

* 第一次 search 时
* ES 固定当前索引状态
* 后续 **新增 / 删除 / 更新的数据都不可见**

---

#### 2️⃣ 返回 scroll_id

```json
"_scroll_id": "DXF1ZXJ5QW5kRmV0Y2gBAAAAAAA..."
```

👉 后续查询 **只认 scroll_id**

---

#### 3️⃣ 游标向前推进

```
scroll_id
   ↓
下一批数据
   ↓
新的 scroll_id
   ↓
直到 hits 为空
```

---

### 三、标准 DSL 使用流程（完整示例）

#### 1️⃣ 初始化 scroll 查询

```json
POST my_index/_search?scroll=2m
{
  "size": 1000,
  "query": {
    "match_all": {}
  },
  "sort": ["_doc"]
}
```

📌 关键点：

* `scroll=2m`：保持游标 2 分钟
* `size`：每批返回条数
* `sort: _doc`：**性能最优**

---

#### 2️⃣ 返回结果（关键字段）

```json
{
  "_scroll_id": "FGluY2x1ZGVfZGVsZXRlZA...",
  "hits": {
    "hits": [ ...1000条数据... ]
  }
}
```

---

#### 3️⃣ 获取下一批数据

```json
POST _search/scroll
{
  "scroll": "2m",
  "scroll_id": "FGluY2x1ZGVfZGVsZXRlZA..."
}
```

📌 返回：

* 新的 `_scroll_id`
* 下一批数据

---

#### 4️⃣ 结束条件

```json
"hits": []
```

👉 表示数据扫描完成

---

#### 5️⃣ 手动清理 scroll（强烈建议）

```json
DELETE _search/scroll
{
  "scroll_id": "FGluY2x1ZGVfZGVsZXRlZA..."
}
```

---

### 四、为什么 scroll 一定要用 `_doc` 排序？

#### ❌ 错误写法

```json
"sort": [{ "timestamp": "asc" }]
```

问题：

* 会触发全局排序
* 性能极差
* 违背 scroll 初衷

---

#### ✅ 正确写法

```json
"sort": ["_doc"]
```

优势：

* 按 segment 顺序扫描
* **最快**
* 内存占用最小

---

### 五、scroll vs search_after（核心对比）

| 对比项       | scroll      | search_after  |
| ------------ | ----------- | ------------- |
| 使用目的     | 全量遍历    | 实时分页      |
| 是否实时     | ❌           | ✅             |
| 是否固定数据 | ✅（快照）   | ❌（除非 PIT） |
| 是否占用资源 | 高          | 低            |
| 支持跳页     | ❌           | ❌             |
| 推荐程度     | ❌（新系统） | ✅             |

📌 **官方建议：新系统优先 PIT + search_after**

---

### 六、scroll 的致命问题（生产重点）

#### 1️⃣ 占用大量资源

* 每个 scroll 都持有搜索上下文
* 多用户 = 内存爆炸

---

#### 2️⃣ 无法横向扩展

* scroll 是“会话态”
* 不适合高并发 API

---

#### 3️⃣ scroll_id 过期即失效

* 超过 keep_alive
* 必须从头开始

---

### 七、scroll 的替代方案（ES 7.10+）

#### ✅ PIT + search_after（推荐）

| 场景     | 推荐               |
| -------- | ------------------ |
| 用户分页 | search_after       |
| 稳定分页 | PIT + search_after |
| 全量导出 | scroll / reindex   |

---

### 八、Java / Python 示例（简化）

#### Java（RestHighLevelClient）

```java
SearchRequest request = new SearchRequest("my_index");
request.scroll(TimeValue.timeValueMinutes(2));
```

---

#### Python

```python
resp = es.search(
    index="my_index",
    scroll="2m",
    size=1000,
    sort="_doc"
)
```

---

### 九、scroll 的最佳实践总结

✅ 只用于 **全量扫描**
✅ 批量 size 建议：500 ~ 5000
✅ 一定用 `_doc` 排序
✅ 用完立刻 clear scroll
❌ 不要做用户分页

---

### 十、一句话总结

> **scroll 是“数据库全表扫描”，不是分页工具**



# 数据聚合

## 字段（term）分组/分桶

### 一、terms 聚合是什么？

**terms 聚合**用于
👉 **按字段的“唯一值”进行分组统计**（类似 SQL 的 `GROUP BY`）

典型用途：

* 按 **服务名** 统计日志条数
* 按 **主机 IP** 统计请求量
* 按 **状态码** 统计分布
* Top N 统计（最常见）

---

### 二、最基础的 terms 聚合

#### 示例：按 `status` 分组统计文档数

```json
GET logs/_search
{
  "size": 0,
  "aggs": {
    "status_count": {
      "terms": {
        "field": "status"
      }
    }
  }
}
```

#### 返回结果（示意）

```json
"aggregations": {
  "status_count": {
    "buckets": [
      { "key": 200, "doc_count": 12345 },
      { "key": 404, "doc_count": 321 },
      { "key": 500, "doc_count": 87 }
    ]
  }
}
```

📌 **解释**

* `key`：字段的值
* `doc_count`：该值对应的文档数

---

### 三、terms 常用参数（非常重要）

#### 1️⃣ size（Top N）

```json
"terms": {
  "field": "service_name.keyword",
  "size": 5
}
```

👉 只返回 **出现次数最多的前 5 个值**

⚠️ 默认 `size = 10`

---

#### 2️⃣ field：一定要注意 keyword

❌ 错误用法：

```json
"field": "service_name"
```

✅ 正确用法（字符串字段）：

```json
"field": "service_name.keyword"
```

📌 原因：

* `text` 类型 **不能直接用于 terms**
* 需要 `keyword`（未分词）

---

#### 3️⃣ order：排序方式

#### 按 doc_count 降序（默认）

```json
"order": { "_count": "desc" }
```

#### 按 doc_count 升序

```json
"order": { "_count": "asc" }
```

#### 按 key 排序

```json
"order": { "_key": "asc" }
```

---

#### 4️⃣ missing：缺失值处理

```json
"terms": {
  "field": "host.keyword",
  "missing": "UNKNOWN"
}
```

👉 没有该字段的文档会被归为 `UNKNOWN`

---

#### 5️⃣ min_doc_count：过滤低频值

```json
"terms": {
  "field": "status",
  "min_doc_count": 10
}
```

👉 只统计 **出现 ≥10 次** 的值

---

### 四、terms + 子聚合（非常常用）

#### 示例：按服务分组，再统计平均响应时间

```json
GET apm/_search
{
  "size": 0,
  "aggs": {
    "by_service": {
      "terms": {
        "field": "service_name.keyword",
        "size": 10
      },
      "aggs": {
        "avg_rt": {
          "avg": {
            "field": "response_time"
          }
        }
      }
    }
  }
}
```

📌 典型 **运维/APM 场景**

---

### 五、terms + 查询条件（先过滤再聚合）

```json
GET logs/_search
{
  "size": 0,
  "query": {
    "range": {
      "@timestamp": {
        "gte": "now-1h"
      }
    }
  },
  "aggs": {
    "by_level": {
      "terms": {
        "field": "level.keyword"
      }
    }
  }
}
```

👉 **最近 1 小时** 不同日志级别分布

---

### 六、高基数字段 & 性能问题（重点）

#### ⚠️ terms 不适合字段

* traceId
* userId
* requestId
* UUID

原因：

* 唯一值太多（高基数）
* 内存消耗大
* 结果不准确

#### 替代方案

| 场景      | 建议                 |
| --------- | -------------------- |
| 去重数量  | `cardinality`        |
| 分页聚合  | `composite`          |
| 精确 TopN | `terms + shard_size` |

---

### 七、terms 精度问题（shard_size）

```json
"terms": {
  "field": "service.keyword",
  "size": 10,
  "shard_size": 100
}
```

📌 **原理**

* 每个 shard 先取 `shard_size`
* 再在 coordinator 合并

👉 shard 多、数据倾斜时必须调大

---

### 八、典型运维 / 日志分析案例

#### 1️⃣ Top 10 服务错误数

```json
"terms": {
  "field": "service.keyword",
  "size": 10
}
```

* query 中过滤 `status >= 500`

---

#### 2️⃣ 按主机统计请求量

```json
"terms": {
  "field": "host.ip",
  "size": 20
}
```

---

#### 3️⃣ 服务 + 状态码二维统计

```json
"aggs": {
  "by_service": {
    "terms": {
      "field": "service.keyword"
    },
    "aggs": {
      "by_status": {
        "terms": {
          "field": "status"
        }
      }
    }
  }
}
```

---

### 九、terms vs SQL 对照

```sql
SELECT status, COUNT(*)
FROM logs
GROUP BY status
ORDER BY COUNT(*) DESC
LIMIT 10;
```

对应 ES：

```json
"terms": {
  "field": "status",
  "size": 10,
  "order": { "_count": "desc" }
}
```



## 时间（date_histogram）分组/分桶

### 一、`date_histogram` 是什么？

`date_histogram` 是 **按时间字段进行分桶（Bucket）的聚合**，常用于：

* ⏱️ **按分钟 / 小时 / 天统计指标**
* 📊 **监控数据趋势（CPU、内存、QPS）**
* 📈 **日志量、错误数随时间变化**
* 🔍 **SkyWalking / APM / Trace 数据分析**

👉 本质：

> **把时间轴切成一个个时间桶，每个桶里再做统计**

---

### 二、最基本用法

#### 示例：按小时统计日志数量

```json
{
  "size": 0,
  "aggs": {
    "logs_per_hour": {
      "date_histogram": {
        "field": "@timestamp",
        "calendar_interval": "hour"
      }
    }
  }
}
```

#### 返回结果结构

```json
"aggregations": {
  "logs_per_hour": {
    "buckets": [
      {
        "key": 1705298400000,
        "key_as_string": "2024-01-15T10:00:00.000Z",
        "doc_count": 12345
      }
    ]
  }
}
```

#### 字段说明

| 字段            | 含义               |
| --------------- | ------------------ |
| `key`           | 时间戳（毫秒）     |
| `key_as_string` | 人类可读时间       |
| `doc_count`     | 该时间桶内的文档数 |

---

### 三、`calendar_interval` vs `fixed_interval`（⚠️重点）

#### 1️⃣ calendar_interval（**推荐用于自然时间**）

> 和日历有关，会考虑：

* 月份天数不同
* 闰年
* 夏令时（DST）

```json
"calendar_interval": "day"
```

支持：

* `minute`
* `hour`
* `day`
* `week`
* `month`
* `year`

📌 **日志 / 业务 / 报表首选**

---

#### 2️⃣ fixed_interval（**固定毫秒长度**）

> 严格按毫秒切分，不管日历

```json
"fixed_interval": "10m"
```

支持：

* `ms`
* `s`
* `m`
* `h`
* `d`

📌 **监控指标（Prometheus 风格）更合适**

---

#### 对比总结

| 场景                | 推荐                |
| ------------------- | ------------------- |
| 按天、按月报表      | `calendar_interval` |
| 精确 5 分钟 / 30 秒 | `fixed_interval`    |
| 存在 DST            | `calendar_interval` |

---

### 四、时间范围控制（query + aggs）

⚠️ **聚合本身不负责过滤时间**

```json
{
  "query": {
    "range": {
      "@timestamp": {
        "gte": "now-24h",
        "lte": "now"
      }
    }
  },
  "aggs": {
    "per_5m": {
      "date_histogram": {
        "field": "@timestamp",
        "fixed_interval": "5m"
      }
    }
  }
}
```

👉 **最佳实践**：

> **时间过滤一定放在 `query.range` 里**

---

### 五、`time_zone`（国内环境必用）

如果你看到时间桶偏 8 小时，原因就是 **UTC 默认时区**

```json
"time_zone": "+08:00"
```

完整示例：

```json
"date_histogram": {
  "field": "@timestamp",
  "calendar_interval": "hour",
  "time_zone": "+08:00"
}
```

📌 **国内日志、监控几乎必加**

---

### 六、空桶补齐（`min_doc_count` + `extended_bounds`）

#### 问题：

> 没数据的时间段直接消失？

#### 解决方案

```json
"date_histogram": {
  "field": "@timestamp",
  "fixed_interval": "5m",
  "min_doc_count": 0,
  "extended_bounds": {
    "min": "now-1h",
    "max": "now"
  }
}
```

效果：

* ⏳ 即使某 5 分钟没有数据
* 也会返回 `doc_count = 0`

📌 **画监控曲线必用**

---

### 七、子聚合（真正的威力）

#### 示例：每分钟错误数

```json
{
  "aggs": {
    "per_min": {
      "date_histogram": {
        "field": "@timestamp",
        "fixed_interval": "1m"
      },
      "aggs": {
        "error_count": {
          "filter": {
            "term": {
              "level": "ERROR"
            }
          }
        }
      }
    }
  }
}
```

返回结构：

```json
"buckets": [
  {
    "key_as_string": "2024-01-15T10:01:00",
    "doc_count": 200,
    "error_count": {
      "doc_count": 5
    }
  }
]
```

---

### 八、结合指标聚合（metrics）

#### 示例：每 5 分钟 CPU 平均值

```json
{
  "aggs": {
    "cpu_per_5m": {
      "date_histogram": {
        "field": "@timestamp",
        "fixed_interval": "5m"
      },
      "aggs": {
        "avg_cpu": {
          "avg": {
            "field": "cpu_usage"
          }
        }
      }
    }
  }
}
```

📌 **运维监控 / APM 标准用法**

---

### 九、性能与坑点（重要）

#### ❌ 常见坑

| 问题          | 原因                 |
| ------------- | -------------------- |
| 聚合很慢      | 时间范围太大         |
| 桶数量爆炸    | interval 太小        |
| 时间错 8 小时 | 没设置 `time_zone`   |
| 没有空桶      | 没加 `min_doc_count` |

---

#### ✅ 性能建议

* 时间范围 **<= 7 天**
* 分桶数量 **< 10,000**
* 大范围用 **rollup / downsample**
* 指标索引建议 **doc_values**

---

### 十、典型运维 / APM 场景速查

| 场景     | interval         |
| -------- | ---------------- |
| 实时监控 | `1m` / `30s`     |
| 日志趋势 | `1h`             |
| 日报     | `1d`             |
| 月报     | `1M`（calendar） |

---

### 十一、一句话总结

> **`date_histogram` = 时间序列分析核心**
>
> * `query` 控制范围
> * `interval` 决定精度
> * `time_zone` 决定正确性
> * 子聚合决定分析深度

---



## 区间自定义（range）分桶/分组

### 一、`range` 聚合是什么？

`range` 是一种 **Bucket 聚合**，用于：

> **按数值区间 / 时间区间 手动分桶**

📌 典型用途：

* 响应时间区间统计（RT <100ms、100–500ms、>1s）
* 请求大小分布
* 状态码范围
* 金额区间
* Trace / Span 耗时分布

---

### 二、最基础示例（数值区间）

#### 示例：请求耗时分布（单位 ms）

```json
{
  "size": 0,
  "aggs": {
    "latency_range": {
      "range": {
        "field": "duration",
        "ranges": [
          { "to": 100 },
          { "from": 100, "to": 500 },
          { "from": 500, "to": 1000 },
          { "from": 1000 }
        ]
      }
    }
  }
}
```

#### 返回结果

```json
"aggregations": {
  "latency_range": {
    "buckets": [
      { "key": "*-100.0", "to": 100, "doc_count": 1200 },
      { "key": "100.0-500.0", "from": 100, "to": 500, "doc_count": 860 },
      { "key": "500.0-1000.0", "from": 500, "to": 1000, "doc_count": 240 },
      { "key": "1000.0-*", "from": 1000, "doc_count": 35 }
    ]
  }
}
```

---

### 三、区间规则（非常重要）

| 写法              | 含义            |
| ----------------- | --------------- |
| `from`            | **包含**（>=）  |
| `to`              | **不包含**（<） |
| `{ "to": 100 }`   | `< 100`         |
| `{ "from": 100 }` | `>= 100`        |

⚠️ 不允许区间重叠，否则结果不可靠。

---

### 四、时间字段的 `range`（日期区间）

`range` **可以直接作用在 `date` 字段**

#### 示例：按时间区间统计请求数

```json
{
  "aggs": {
    "time_range": {
      "range": {
        "field": "@timestamp",
        "ranges": [
          { "to": "now-1h" },
          { "from": "now-1h", "to": "now" }
        ]
      }
    }
  }
}
```

📌 适合：

* 对比 最近 1 小时 vs 更早
* SLA 对比

---

### 五、自定义 key（强烈推荐）

```json
"ranges": [
  { "key": "fast", "to": 100 },
  { "key": "normal", "from": 100, "to": 500 },
  { "key": "slow", "from": 500 }
]
```

返回结果：

```json
"key": "fast"
```

📌 **用于前端 / 报表非常友好**

---

### 六、子聚合（核心用法）

#### 示例：各耗时区间的错误数

```json
{
  "aggs": {
    "latency": {
      "range": {
        "field": "duration",
        "ranges": [
          { "to": 200 },
          { "from": 200 }
        ]
      },
      "aggs": {
        "error_count": {
          "filter": {
            "term": {
              "status": "ERROR"
            }
          }
        }
      }
    }
  }
}
```

---

### 七、`range` vs `histogram` vs `date_histogram`

| 场景         | 用哪个           |
| ------------ | ---------------- |
| 自定义区间   | `range`          |
| 等宽数值分桶 | `histogram`      |
| 时间序列     | `date_histogram` |
| RT 分布      | `range`          |
| QPS 曲线     | `date_histogram` |

---

### 八、性能与注意事项

#### ❌ 常见坑

| 问题     | 原因            |
| -------- | --------------- |
| 统计不准 | 区间重叠        |
| 查询慢   | ranges 太多     |
| 时间错乱 | date 未统一时区 |

#### ✅ 建议

* 区间数量 **< 20**
* 数值字段必须是 `numeric`
* 时间字段注意 UTC / 本地时区
* 大数据量建议 **预聚合**

---

### 九、运维 / APM 经典案例

#### Trace 耗时分布

```json
"ranges": [
  { "key": "<100ms", "to": 100 },
  { "key": "100~300ms", "from": 100, "to": 300 },
  { "key": "300~1s", "from": 300, "to": 1000 },
  { "key": ">1s", "from": 1000 }
]
```

---

### 十、一句话总结

> **`range` = 人工定义分桶**
>
> * 精准控制区间
> * 非等宽
> * 非时间序列
> * 分布分析神器

## 等宽（histogram）分桶/分组

### 一、`histogram` 聚合是什么？

`histogram` 是 **数值型等宽分桶（Bucket）聚合**：

> **按照固定步长（interval）把数值字段切成连续区间**

📌 常见用途：

* 接口耗时分布（0–100、100–200、200–300 ms…）
* 请求大小分布
* 数值型指标分布（CPU、内存、磁盘 IO）
* Span / Trace duration 分布（等宽）

---

### 二、最基本用法

#### 示例：接口耗时（每 100ms 一个桶）

```json
{
  "size": 0,
  "aggs": {
    "latency_histogram": {
      "histogram": {
        "field": "duration",
        "interval": 100
      }
    }
  }
}
```

#### 返回结果示例

```json
"aggregations": {
  "latency_histogram": {
    "buckets": [
      { "key": 0.0,   "doc_count": 320 },
      { "key": 100.0, "doc_count": 860 },
      { "key": 200.0, "doc_count": 410 }
    ]
  }
}
```

#### `key` 的含义

> **桶的起始值**

* key = 100 → `[100, 200)`

---

### 三、`interval` 的工作方式（核心）

假设：

```json
"interval": 100
```

那么桶是：

| key  | 范围      |
| ---- | --------- |
| 0    | [0,100)   |
| 100  | [100,200) |
| 200  | [200,300) |

📌 **左闭右开**

---

### 四、`min_doc_count` + `extended_bounds`（补空桶）

#### 问题

没有数据的区间不会返回？

#### 解决

```json
"histogram": {
  "field": "duration",
  "interval": 100,
  "min_doc_count": 0,
  "extended_bounds": {
    "min": 0,
    "max": 1000
  }
}
```

📌 **画分布图 / 热力图必用**

---

### 五、`offset`（控制桶起点）

#### 场景

不想从 0 开始？

```json
"offset": 50
```

效果：

| key  | 区间      |
| ---- | --------- |
| 50   | [50,150)  |
| 150  | [150,250) |

📌 常用于：

* 金额区间
* 非 0 起始指标

---

### 六、子聚合（真正的分析能力）

#### 示例：每个耗时区间的错误数

```json
{
  "aggs": {
    "latency": {
      "histogram": {
        "field": "duration",
        "interval": 200
      },
      "aggs": {
        "error_count": {
          "filter": {
            "term": {
              "status": "ERROR"
            }
          }
        }
      }
    }
  }
}
```

---

### 七、`histogram` vs `range`（重点对比）

| 对比项   | histogram | range      |
| -------- | --------- | ---------- |
| 分桶方式 | 等宽      | 手动定义   |
| 区间数量 | 自动      | 手动       |
| 适合     | 分布分析  | SLA / 阈值 |
| 桶边界   | 自动      | 精确控制   |
| 常用场景 | RT 分布   | RT 等级    |

📌 **想看分布 → histogram**
📌 **想看区间等级 → range**

---

### 八、`histogram` vs `date_histogram`

| 项目     | histogram | date_histogram |
| -------- | --------- | -------------- |
| 字段类型 | numeric   | date           |
| interval | 数值      | 时间           |
| 时区     | 无        | 有             |
| 场景     | 数值分布  | 时间趋势       |

---

### 九、性能与坑点（重要）

#### ❌ 常见坑

| 问题     | 原因                 |
| -------- | -------------------- |
| 桶太多   | interval 太小        |
| 查询慢   | extended_bounds 太大 |
| 无法使用 | 字段不是 numeric     |
| 结果异常 | offset 用错          |

#### ✅ 建议

* 桶数量 **< 200**
* interval ≥ 数据最小精度
* 大范围数据考虑 **预聚合 / rollup**
* 指标字段必须有 `doc_values`

---

### 十、运维 / APM 典型案例

#### SkyWalking Span 耗时分布

```json
"histogram": {
  "field": "span.duration",
  "interval": 100
}
```

#### JVM GC 停顿时间分布

```json
"histogram": {
  "field": "gc_pause",
  "interval": 50
}
```

---

### 十一、一句话总结

> **`histogram` = 等宽数值分布分析利器**
>
> * 自动分桶
> * 连续区间
> * 非时间
> * 分布可视化首选



## min_doc_count` + `extended_bounds

### 一、它们分别解决什么问题？

#### 1️⃣ `min_doc_count`

> **控制是否返回“空桶”**

| 值          | 含义                   |
| ----------- | ---------------------- |
| `1`（默认） | 只返回有数据的桶       |
| `0`         | **即使没数据也返回桶** |

📌 但：**光有 `min_doc_count: 0` 还不够**

---

#### 2️⃣ `extended_bounds`

> **强制指定桶的起止范围**

```json
"extended_bounds": {
  "min": 0,
  "max": 1000
}
```

📌 否则 ES 只会根据 **已有数据** 决定桶范围

---

### 二、为什么一定要一起用？

#### ❌ 只有 `min_doc_count: 0`

```json
"histogram": {
  "field": "duration",
  "interval": 100,
  "min_doc_count": 0
}
```

❗ 问题：

* ES **不知道你要多大范围**
* 仍然不会补齐首尾空桶

---

#### ❌ 只有 `extended_bounds`

```json
"extended_bounds": {
  "min": 0,
  "max": 1000
}
```

❗ 问题：

* 默认 `min_doc_count = 1`
* **空桶仍然被过滤**

---

#### ✅ 正确姿势（必须同时存在）

```json
"histogram": {
  "field": "duration",
  "interval": 100,
  "min_doc_count": 0,
  "extended_bounds": {
    "min": 0,
    "max": 1000
  }
}
```

👉 **所有桶都会返回，包括 doc_count = 0**

---

### 三、`date_histogram` 示例（最常见）

#### 场景：最近 1 小时，每 5 分钟一条监控点

```json
{
  "size": 0,
  "query": {
    "range": {
      "@timestamp": {
        "gte": "now-1h",
        "lte": "now"
      }
    }
  },
  "aggs": {
    "per_5m": {
      "date_histogram": {
        "field": "@timestamp",
        "fixed_interval": "5m",
        "time_zone": "+08:00",
        "min_doc_count": 0,
        "extended_bounds": {
          "min": "now-1h",
          "max": "now"
        }
      }
    }
  }
}
```

📌 效果：

* 每 5 分钟一个点
* 即使没有数据，也会返回
* 画折线图 **不会断**

---

### 四、`histogram` 示例（数值分布）

#### 场景：RT 分布 0~2000ms，每 200ms 一档

```json
"histogram": {
  "field": "duration",
  "interval": 200,
  "min_doc_count": 0,
  "extended_bounds": {
    "min": 0,
    "max": 2000
  }
}
```

---

### 五、`extended_bounds` 的 min / max 写法

#### 1️⃣ 数值字段

```json
"extended_bounds": {
  "min": 0,
  "max": 1000
}
```

---

#### 2️⃣ 日期字段（date_histogram）

```json
"extended_bounds": {
  "min": "2024-01-01T00:00:00",
  "max": "2024-01-01T23:59:59"
}
```

或相对时间：

```json
"min": "now-24h",
"max": "now"
```

⚠️ **时间要和 `time_zone` 语义一致**

---

### 六、和 `hard_bounds` 的区别（容易混）

| 项目         | extended_bounds | hard_bounds |
| ------------ | --------------- | ----------- |
| 是否补桶     | ✅ 是            | ❌ 否        |
| 是否裁剪数据 | ❌ 不裁          | ✅ 裁        |
| 常用场景     | 补齐曲线        | 强制截断    |

📌 **画图用 extended_bounds**
📌 **限制统计范围用 hard_bounds**

---

### 七、常见坑总结（非常重要）

#### ❌ 坑 1：extended_bounds ≠ 过滤数据

```json
"extended_bounds": { "min": 0, "max": 100 }
```

👉 **不会过滤 >100 的数据**
👉 过滤必须用 `query.range`

---

#### ❌ 坑 2：桶数量爆炸

```json
interval = 1
range = 1 天
```

❗ 86400 个桶，ES 会很慢

---

#### ❌ 坑 3：date_histogram 没加 time_zone

导致：

* 数据对齐错
* 空桶位置不对

---

### 八、性能建议（实战经验）

* 桶数量控制在 **< 1000**
* 实时监控：

  * `fixed_interval`
  * `min_doc_count: 0`
* 报表分析：

  * `calendar_interval`
* 超大时间范围 → **downsample / rollup**

---

### 九、一句话总结

> **`min_doc_count + extended_bounds` = 补齐时间轴 / 数值轴**
>
> * 不会过滤数据
> * 只负责“补桶”
> * 画图不断线的关键



# 数据（统计）计算

## max/sum/avg/count

```
{
  "aggs": {
    "avg_latency": { "avg": { "field": "latency" }},
    "max_latency": { "max": { "field": "latency" }},
    "sum_bytes":  { "sum": { "field": "bytes" }},
    "req_count":  { "value_count": { "field": "trace_id" }}
  }
}
```

> stats(一次返回多个指标)

```
{
  "aggs": {
    "latency_stats": {
      "stats": {
        "field": "latency"
      }
    }
  }
}
```

返回结果示例

```
"aggregations" : {
    "all" : {
      "count" : 60,
      "min" : 0.0,
      "max" : 7.0,
      "avg" : 3.466666666666667,
      "sum" : 208.0
    }
  }

```



## Cardinality 去重统计

🔍 理解 Cardinality 聚合

cardinality 聚合用于计算字段中不同值的数量，也称为基数统计。它类似于 SQL 中的 COUNT(DISTINCT field)，能帮你快速了解数据的唯一性。

• 典型应用场景：统计网站的独立访客数（UV）、计算不同商品品牌的数量、分析每月有多少独立用户购买商品。

• 核心特点：cardinality 是一种近似算法，这意味着它可能会存在一定误差，但换来的是处理海量数据时的高性能和低资源消耗。

📝 基本语法与示例

一个最简单的 cardinality 聚合查询如下，它统计 color 字段有多少个不同的值：

```
GET /cars/_search
{
  "size": 0, 
  "aggs": {
    "distinct_colors": {
      "cardinality": {
        "field": "color"
      }
    }
  }
}
```

返回结果示例：

```
{
  ...,
  "aggregations": {
    "distinct_colors": {
      "value": 3
    }
  }
}
```


⚙️ 精度控制：precision_threshold 参数

precision_threshold 是控制精度与内存权衡的关键参数。

• 作用：设定一个阈值。当字段的唯一值数量低于此阈值时，统计结果接近 100% 准确；超过阈值时，会牺牲少量精度以节省内存。

• 取值范围：0 到 40000，默认值为 3000。

• 内存估算：内存消耗约为 precision_threshold * 8 字节。例如，设为 100 时，约消耗 800 字节。

```
GET /cars/_search
{
  "size": 0,
  "aggs": {
    "distinct_colors": {
      "cardinality": {
        "field": "color",
        "precision_threshold": 100 
      }
    }
  }
}
```


🧠 算法原理与性能优化

cardinality 聚合基于 HyperLogLog++ (HLL) 算法。该算法通过计算值的哈希值并进行概率估算，以固定的内存空间高效地统计超大基数的数据集。

你可以通过以下方式优化性能：

1. 预计算哈希值（索引时优化）
   对于非常大或非常高基数的字符串字段，可以在创建索引时使用 murmur3 插件预计算哈希值，将哈希计算从查询时转移到索引时，从而提升查询速度。

   步骤1：定义映射

   ```
   PUT /my_index/
   {
     "mappings": {
       "properties": {
         "my_large_field": {
           "type": "keyword",
           "fields": {
             "hash": { 
               "type": "murmur3"
             }
           }
         }
       }
     }
   }
   ```

   步骤2：查询哈希字段

   ```
   GET /my_index/_search
   {
     "size": 0,
     "aggs": {
       "distinct_values": {
         "cardinality": {
           "field": "my_large_field.hash"
         }
       }
     }
   }
   ```

   

2. 使用脚本过滤
   你可以使用 Painless 脚本对参与基数计算的值进行过滤，增加了灵活性。例如，只统计 age 字段大于 30 的唯一值数量：

   ```
   "aggs": {
     "distinct_colors": {
       "cardinality": {
         "field": "age",
         "script": "doc['age'].value > 30"
       }
     }
   }
   ```

   

🔄 组合其他聚合

cardinality 聚合常与其他聚合结合使用，实现更复杂的分析。

• 按时间统计唯一值（如：每月售出多少种颜色的汽车）

```
GET /sales/_search
  {
    "size": 0,
    "aggs": {
      "sales_per_month": {
        "date_histogram": {
          "field": "sold_date",
          "calendar_interval": "month"
        },
        "aggs": {
          "distinct_brands": {
            "cardinality": {
              "field": "brand"
            }
          }
        }
      }
    }
  }
```

• 先分组再统计唯一值（如：每个产品类别下有多少独立用户访问）

```
GET /products/_search
  {
    "size": 0,
    "aggs": {
      "by_category": {
        "terms": {
          "field": "category"
        },
        "aggs": {
          "unique_visitors": {
            "cardinality": {
              "field": "user_id"
            }
          }
        }
      }
    }
  }
```


💎 总结与选择

特性 说明

本质 基于 HLL 算法的近似去重计数

优点 速度快，占用内存固定，适合高基数场景

缺点 结果是近似值，存在微小误差

精度控制 使用 precision_threshold 参数平衡精度和内存

性能优化 对高基数长字符串字段可预计算哈希 (murmur3)

希望这份教程能帮助你熟练掌握 cardinality 聚合。如果你有特定的使用场景或遇到更具体的问题，我很乐意提供进一步的帮助。



## 百分位 percentiles

------

### 1️⃣ 基本概念

`percentiles` 聚合用于 **计算数值字段在不同百分位上的值**。

- 百分位 (Percentile) 是指数据集排序后 **某个位置的数据值**。
- 例如，`95th percentile` 表示 **95% 的数据都小于等于这个值**。

常用场景：

- 响应时间分析：`95th percentile` RT = 95% 请求都在这个时间以内。
- 日志统计：计算错误大小或延迟分布。

------

### 2️⃣ DSL 示例

假设索引 `logs`，字段 `response_time`：

```json
GET logs/_search
{
  "size": 0,
  "aggs": {
    "response_time_percentiles": {
      "percentiles": {
        "field": "response_time",
        "percents": [50, 75, 90, 95, 99]
      }
    }
  }
}
```

- `percents` 用来指定要计算的百分位。
- 默认返回的是近似值，内部算法是 **TDigest**。

返回示例：

```json
{
  "aggregations": {
    "response_time_percentiles": {
      "values": {
        "50.0": 120,
        "75.0": 200,
        "90.0": 350,
        "95.0": 400,
        "99.0": 500
      }
    }
  }
}
```

- 50th percentile = 120ms → 一半请求 <= 120ms
- 95th percentile = 400ms → 95% 请求 <= 400ms

------

### 3️⃣ 高精度控制

`percentiles` 聚合可以通过 `hdr` 或 `tdigest` 设置精度：

```json
GET logs/_search
{
  "size": 0,
  "aggs": {
    "response_time_percentiles": {
      "percentiles": {
        "field": "response_time",
        "percents": [50, 75, 90, 95, 99],
        "tdigest": {
          "compression": 100
        }
      }
    }
  }
}
```

- `compression` 越大 → 精度越高，但内存消耗增加。
- 默认 `tdigest.compression` = 100。

------

### 4️⃣ 与其他聚合结合

可以嵌套在 `terms` 或 `date_histogram` 中：

```json
GET logs/_search
{
  "size": 0,
  "aggs": {
    "by_service": {
      "terms": { "field": "service.keyword" },
      "aggs": {
        "response_time_percentiles": {
          "percentiles": {
            "field": "response_time",
            "percents": [50, 95]
          }
        }
      }
    }
  }
}
```

- 按 `service` 分组 → 每个服务计算 50th 和 95th 百分位响应时间。

------

### 5️⃣ 注意事项

1. **近似值**：
   - 大数据量下 `percentiles` 返回的是近似值，但误差可控。
2. **数值类型字段**：
   - `percentiles` 只能对 `long`、`float`、`double` 等数值字段使用。
3. **tdigest vs hdr**：
   - `tdigest`（默认）适合大多数场景
   - `hdr` 高精度，适合需要严格准确百分位的场景

## Pipeline 

> 一句话先定调：**Pipeline 聚合 = 对“聚合结果”再做聚合**，而不是对原始文档动手。

---

### 1️⃣ Pipeline 聚合是什么？

普通聚合（`terms` / `avg` / `sum` …）
👉 **作用在文档上**

Pipeline 聚合
👉 **作用在“其他聚合的结果上”**

所以它**一定是子聚合**，而且必须指定 `buckets_path`。

典型使用场景：

* 计算增长率 / 环比 / 同比
* 对每个 bucket 的指标再求 max / avg
* 过滤 bucket（类似 SQL 的 `having`）
* 累加、移动平均、导数

---

### 2️⃣ Pipeline 聚合两大类（重点）

#### 🔹 ① Parent Pipeline Aggregations

**对同一个 bucket 里的多个值做计算**

常见的有：

| 聚合                         | 作用           |
| -------------------------- | ------------ |
| `bucket_script`            | 自定义脚本计算      |
| `bucket_selector`          | 按条件过滤 bucket |
| `bucket_sort`              | 排序 / 分页      |
| `derivative`               | 导数（变化量）      |
| `moving_avg` / `moving_fn` | 移动计算         |
| `cumulative_sum`           | 累计和          |

---

#### 🔹 ② Sibling Pipeline Aggregations

**对一组 bucket 的结果做整体计算**

常见的有：

| 聚合             | 作用             |
| -------------- | -------------- |
| `avg_bucket`   | 所有 bucket 的平均值 |
| `sum_bucket`   | 所有 bucket 求和   |
| `max_bucket`   | 最大值            |
| `min_bucket`   | 最小值            |
| `stats_bucket` | 统计             |

---

### 3️⃣ 核心参数：`buckets_path`

这是 Pipeline 聚合的**灵魂**👇

```text
buckets_path: <聚合名>[>子聚合名][.metric]
```

示例结构：

```json
aggs
 └─ per_day
     └─ avg_rt
 └─ max_rt   ← pipeline
```

对应路径：

```json
"buckets_path": "per_day>avg_rt"
```

---

### 4️⃣ 常见 Pipeline 聚合实战

#### ✅ 1）bucket_script（算增长率 / 比率）

👉 计算 **错误率 = error_count / total_count**

```json
"error_rate": {
  "bucket_script": {
    "buckets_path": {
      "error": "error_count",
      "total": "total_count"
    },
    "script": "params.error / params.total"
  }
}
```

---

#### ✅ 2）bucket_selector（类似 SQL HAVING）

👉 只保留 **请求数 > 1000 的 bucket**

```json
"filter_bucket": {
  "bucket_selector": {
    "buckets_path": {
      "cnt": "_count"
    },
    "script": "params.cnt > 1000"
  }
}
```

⚠️ 注意：

* **只是过滤结果**，并不会减少 ES 的计算量

---

#### ✅ 3）derivative（变化量 / 增长）

👉 每分钟请求数的变化量

```json
"req_increase": {
  "derivative": {
    "buckets_path": "req_count"
  }
}
```

---

#### ✅ 4）cumulative_sum（累计值）

👉 请求数累计和

```json
"total_req": {
  "cumulative_sum": {
    "buckets_path": "req_count"
  }
}
```

---

#### ✅ 5）avg_bucket（所有 bucket 的平均值）

👉 每天平均请求量

```json
"avg_daily_req": {
  "avg_bucket": {
    "buckets_path": "per_day>req_count"
  }
}
```

---

### 5️⃣ 完整示例（date_histogram + pipeline）

```json
GET logs/_search
{
  "size": 0,
  "aggs": {
    "per_minute": {
      "date_histogram": {
        "field": "@timestamp",
        "fixed_interval": "1m"
      },
      "aggs": {
        "req_count": {
          "value_count": {
            "field": "_id"
          }
        },
        "req_increase": {
          "derivative": {
            "buckets_path": "req_count"
          }
        }
      }
    }
  }
}
```

📈 用于看 **QPS 增长趋势**

---

### 6️⃣ 常见坑（很重要）

1. ❌ Pipeline 聚合不能单独存在

   * **必须依赖其他聚合**

2. ❌ 不能跨层级随便引用

   * `buckets_path` 路径必须合法

3. ⚠️ `bucket_selector` 不省资源

   * 它是 **后处理**

4. ⚠️ `derivative` 需要有序 bucket

   * 一般配合 `date_histogram`

---

### 7️⃣ 一句话总结

> **普通聚合算“数”，Pipeline 聚合算“数的关系”**



# 结果高亮（highlight）

> **Elasticsearch DSL 的高亮（highlight）**，那就是用来在搜索结果里把**命中的关键词标出来**的功能，常用于日志、全文检索、运维排障这类场景 👍

下面给你一个**最常用、最好理解的示例**。

---

### 一、最简单的 DSL 高亮示例

```json
GET my_index/_search
{
  "query": {
    "match": {
      "message": "error timeout"
    }
  },
  "highlight": {
    "fields": {
      "message": {}
    }
  }
}
```

👉 返回结果里会多一个 `highlight` 字段：

```json
"highlight": {
  "message": [
    "request <em>error</em> cause <em>timeout</em>"
  ]
}
```

---

### 二、自定义高亮标签（前后缀）

前端一般不想用 `<em>`，可以自定义：

```json
"highlight": {
  "pre_tags": ["<span class='hl'>"],
  "post_tags": ["</span>"],
  "fields": {
    "message": {}
  }
}
```

前端 CSS：

```css
.hl {
  color: red;
  font-weight: bold;
}
```

---

### 三、多个字段同时高亮

```json
"highlight": {
  "fields": {
    "message": {},
    "exception": {},
    "traceId": {}
  }
}
```

只要字段被 query 命中，就会返回对应高亮内容。

---

### 四、常见高亮参数（很实用）

```json
"highlight": {
  "fields": {
    "message": {
      "fragment_size": 150,
      "number_of_fragments": 3,
      "no_match_size": 50
    }
  }
}
```

参数说明：

| 参数                    | 作用         |
| --------------------- | ---------- |
| `fragment_size`       | 每个高亮片段长度   |
| `number_of_fragments` | 返回几个片段     |
| `no_match_size`       | 没命中也返回部分内容 |

---

### 五、highlight 类型（重点）

```json
"highlight": {
  "fields": {
    "message": {
      "type": "unified"
    }
  }
}
```

常见三种：

| 类型        | 说明                      |
| --------- | ----------------------- |
| `unified` | **默认 & 推荐**，效果最好        |
| `plain`   | 简单高亮，性能一般               |
| `fvh`     | 需要 `term_vector`，大文本性能好 |

> 日志、运维检索：**unified 就够了**

---

### 六、在日志 / 运维场景的一个典型例子

```json
GET logs-*/_search
{
  "query": {
    "bool": {
      "must": [
        { "match": { "level": "ERROR" }},
        { "match": { "message": "timeout" }}
      ]
    }
  },
  "highlight": {
    "pre_tags": ["<mark>"],
    "post_tags": ["</mark>"],
    "fields": {
      "message": {}
    }
  }
}
```

---

### 七、几个**坑点提醒**（很重要）

1. **keyword 类型不能高亮**

   * 必须是 `text`
2. `match` / `query_string` 才会触发高亮
3. `_source` 被关闭，高亮可能异常
4. 中文分词效果取决于分词器（ik / smart）

---
