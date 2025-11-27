# Python 字典(dict)完整教程

字典是Python中非常重要且常用的数据结构，用于存储键值对(key-value)的映射关系。

## 1. 字典基础

### 创建字典

```
# 空字典
empty_dict = {}
empty_dict2 = dict()

# 直接创建字典
person = {"name": "Alice", "age": 25, "city": "New York"}

# 使用dict()构造函数
person2 = dict(name="Bob", age=30, city="London")

# 从键值对列表创建
person3 = dict([("name", "Charlie"), ("age", 35), ("city", "Paris")])

print(person)  # {'name': 'Alice', 'age': 25, 'city': 'New York'}
```

### 字典特性

- 键必须是不可变类型（字符串、数字、元组等）
- 值可以是任意类型
- 字典是无序的（Python 3.7+保持插入顺序）
- 键是唯一的

## 2. 基本操作

### 访问元素

```
person = {"name": "Alice", "age": 25, "city": "New York"}

# 使用键访问
print(person["name"])  # Alice

# 使用get()方法（避免KeyError）
print(person.get("age"))      # 25
print(person.get("country"))  # None
print(person.get("country", "USA"))  # 指定默认值：USA
```

### 添加和修改元素

```
person = {"name": "Alice", "age": 25}

# 添加新键值对
person["city"] = "New York"

# 修改现有键的值
person["age"] = 26

# 使用update()批量更新
person.update({"age": 27, "country": "USA"})
print(person)  # {'name': 'Alice', 'age': 27, 'city': 'New York', 'country': 'USA'}
```

### 删除元素

```
person = {"name": "Alice", "age": 25, "city": "New York"}

# del语句
del person["age"]

# pop()方法（返回被删除的值）
city = person.pop("city")
print(city)  # New York

# popitem()删除最后插入的键值对（Python 3.7+）
last_item = person.popitem()

# 清空字典
person.clear()

# 删除整个字典
del person
```

## 3. 字典方法

### 常用方法示例

```
person = {"name": "Alice", "age": 25, "city": "New York"}

# 获取所有键
keys = person.keys()
print(list(keys))  # ['name', 'age', 'city']

# 获取所有值
values = person.values()
print(list(values))  # ['Alice', 25, 'New York']

# 获取所有键值对
items = person.items()
print(list(items))  # [('name', 'Alice'), ('age', 25), ('city', 'New York')]

# 检查键是否存在
print("name" in person)  # True
print("country" not in person)  # True

# 获取字典长度
print(len(person))  # 3

# 设置默认值
person.setdefault("country", "USA")
print(person["country"])  # USA
```

## 4. 字典遍历

### 遍历键值对

```
person = {"name": "Alice", "age": 25, "city": "New York"}

# 遍历所有键
for key in person:
    print(key)

# 遍历所有值
for value in person.values():
    print(value)

# 遍历所有键值对
for key, value in person.items():
    print(f"{key}: {value}")
```

## 5. 字典推导式

### 创建新字典

```
# 基本字典推导式
numbers = [1, 2, 3, 4, 5]
squared = {x: x**2 for x in numbers}
print(squared)  # {1: 1, 2: 4, 3: 9, 4: 16, 5: 25}

# 带条件的字典推导式
even_squares = {x: x**2 for x in numbers if x % 2 == 0}
print(even_squares)  # {2: 4, 4: 16}

# 转换现有字典
person = {"name": "Alice", "age": 25}
uppercase_person = {k.upper(): v for k, v in person.items()}
print(uppercase_person)  # {'NAME': 'Alice', 'AGE': 25}
```

## 6. 嵌套字典

### 多层字典结构

```
# 嵌套字典
company = {
    "employee1": {
        "name": "Alice",
        "age": 25,
        "position": "Developer"
    },
    "employee2": {
        "name": "Bob", 
        "age": 30,
        "position": "Manager"
    }
}

# 访问嵌套字典
print(company["employee1"]["name"])  # Alice

# 添加嵌套数据
company["employee3"] = {"name": "Charlie", "age": 35}
```

## 7. 高级技巧

### defaultdict

```
from collections import defaultdict

# 为不存在的键提供默认值
word_count = defaultdict(int)
words = ["apple", "banana", "apple", "orange"]

for word in words:
    word_count[word] += 1

print(dict(word_count))  # {'apple': 2, 'banana': 1, 'orange': 1}
```

### Counter

```
from collections import Counter

# 快速计数
words = ["apple", "banana", "apple", "orange"]
word_count = Counter(words)
print(word_count)  # Counter({'apple': 2, 'banana': 1, 'orange': 1})
```

### 字典合并

```
# Python 3.5+ 使用 ** 操作符
dict1 = {"a": 1, "b": 2}
dict2 = {"b": 3, "c": 4}
merged = {**dict1, **dict2}
print(merged)  # {'a': 1, 'b': 3, 'c': 4}

# Python 3.9+ 使用 | 操作符
merged = dict1 | dict2
print(merged)  # {'a': 1, 'b': 3, 'c': 4}
```

## 8. 实际应用示例

### 配置文件管理

```
config = {
    "database": {
        "host": "localhost",
        "port": 5432,
        "name": "mydb"
    },
    "server": {
        "host": "0.0.0.0", 
        "port": 8000
    }
}

# 安全获取嵌套配置
db_port = config.get("database", {}).get("port", 5432)
```

### 数据分组

```
# 按类别分组数据
students = [
    {"name": "Alice", "grade": "A"},
    {"name": "Bob", "grade": "B"},
    {"name": "Charlie", "grade": "A"}
]

grade_groups = {}
for student in students:
    grade = student["grade"]
    if grade not in grade_groups:
        grade_groups[grade] = []
    grade_groups[grade].append(student["name"])

print(grade_groups)  # {'A': ['Alice', 'Charlie'], 'B': ['Bob']}
```

## 总结

字典是Python中最灵活和强大的数据结构之一，具有以下特点：

- O(1)时间复杂度的查找、插入、删除操作
- 灵活的键值对存储
- 丰富的内置方法
- 支持多种高级用法

掌握字典的使用对于编写高效的Python代码至关重要。