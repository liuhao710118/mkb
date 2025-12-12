下面给你一份**超详细、超实用的 Python JSON 教程**，从入门到常见坑一次讲清楚。内容适合快速学习，也适合做参考手册。

------

# 🐍 Python JSON 教程（超详细）

Python 标准库内置了 `json` 模块，用于 **JSON 与 Python 对象之间的相互转换**。

------

# 1. JSON 与 Python 数据类型对应关系

| JSON 类型 | Python 类型 |
| --------- | ----------- |
| object    | dict        |
| array     | list        |
| string    | str         |
| number    | int / float |
| true      | True        |
| false     | False       |
| null      | None        |

------

# 2. json 模块常用函数

| 函数           | 说明                       |
| -------------- | -------------------------- |
| `json.dumps()` | Python → JSON 字符串       |
| `json.dump()`  | Python → JSON 写入文件     |
| `json.loads()` | JSON 字符串 → Python       |
| `json.load()`  | 从文件中读取 JSON → Python |

------

# 3. Python → JSON

## 3.1 `json.dumps()`：对象变 JSON 字符串

```python
import json

data = {
    "name": "Tom",
    "age": 20,
    "skills": ["Python", "Docker"]
}

json_str = json.dumps(data)
print(json_str)
```

### 输出

```json
{"name": "Tom", "age": 20, "skills": ["Python", "Docker"]}
```

------

## 3.2 美化输出（pretty print）

```python
json_str = json.dumps(data, indent=4, ensure_ascii=False)
print(json_str)
```

### 常用参数：

- `indent=4`：缩进 4 格
- `ensure_ascii=False`：允许中文正常显示（否则会变 Unicode）

------

## 3.3 `json.dump()`：写入文件

```python
with open("data.json", "w", encoding="utf-8") as f:
    json.dump(data, f, indent=4, ensure_ascii=False)
```

------

# 4. JSON → Python

## 4.1 `json.loads()`：解析字符串

```python
json_text = '{"name": "Tom", "age": 20}'
data = json.loads(json_text)
print(data)
```

### 输出：

```python
{'name': 'Tom', 'age': 20}
```

------

## 4.2 `json.load()`：从文件加载

```python
with open("data.json", "r", encoding="utf-8") as f:
    data = json.load(f)
```

------

# 5. JSON 保存与读取的完整案例

```python
import json

student = {
    "name": "Alice",
    "score": 95,
    "passed": True,
    "tags": ["excellent", "top"]
}

# 保存
with open("student.json", "w", encoding="utf-8") as f:
    json.dump(student, f, indent=4, ensure_ascii=False)

# 读取
with open("student.json", "r", encoding="utf-8") as f:
    data = json.load(f)

print(data)
```

------

# 6. JSON 编码常见问题

## ❗ 6.1 Python 的 set 不能直接转 JSON

```python
data = {"tags": {"A", "B"}}
json.dumps(data)  # 会报错
```

解决：手工转换成 list：

```python
json.dumps({"tags": list({"A", "B"})})
```

------

## ❗ 6.2 datetime 不能直接序列化

```python
import datetime
json.dumps({"time": datetime.datetime.now()})  # 报错
```

解决方案：

### 方法 1：转字符串

```python
json.dumps({"time": datetime.datetime.now().isoformat()})
```

### 方法 2：自定义 JSONEncoder

```python
from json import JSONEncoder
import datetime

class MyEncoder(JSONEncoder):
    def default(self, obj):
        if isinstance(obj, datetime.datetime):
            return obj.isoformat()
        return super().default(obj)

json.dumps({"time": datetime.datetime.now()}, cls=MyEncoder)
```

------

# 7. 将 JSON 映射为 Python 对象（class）

一般 JSON 映射到 dict，若想映射到 class，可以用：

## 方法 1：手动解析

```python
class User:
    def __init__(self, name, age):
        self.name = name
        self.age = age

data = json.loads('{"name": "Tom", "age": 20}')
user = User(**data)
```

## 方法 2：用 dataclass

```python
from dataclasses import dataclass
import json

@dataclass
class User:
    name: str
    age: int

data = json.loads('{"name": "Tom", "age": 20}')
user = User(**data)
```

------

# 8. JSON Lines（逐行 JSON）

很多日志、Kafka 使用 JSONL：

```
{"a":1}
{"b":2}
```

读取方法：

```python
with open("data.jsonl") as f:
    for line in f:
        obj = json.loads(line)
        print(obj)
```

------

# 9. 小结

你现在掌握了：

✔ JSON 与 Python 类型对应
 ✔ dumps / dump / loads / load
 ✔ JSON 美化输出
 ✔ 读取/写入文件
 ✔ 解决无法序列化问题（datetime、set）
 ✔ JSON 映射到类
 ✔ 读取 JSON Lines

如果你愿意，我还能帮你做：

📌 JSON 教程 PDF
 📌 JSON 高级技巧（如 obj_hook、parse_float、parse_int）
 📌 实用项目实例（如解析 API 返回值、日志转 JSON、SkyWalking Trace 解码相关示例）

需要继续吗？