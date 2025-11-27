# Python字符串(str)完整教程

字符串是Python中最常用的数据类型，用于表示文本信息。

## 1. 字符串基础

### 创建字符串

```python
# 单引号
s1 = 'Hello World'

# 双引号
s2 = "Hello World"

# 三引号（多行字符串）
s3 = '''这是一个
多行
字符串'''

s4 = """这也是一个
多行
字符串"""

# 空字符串
empty_str = ""
```

### 字符串特性

- 不可变对象（创建后不能修改）
- 有序的字符序列
- 支持索引和切片操作
- 支持各种编码（UTF-8、ASCII等）

## 2. 基本操作

### 字符串连接

```python
# 使用 + 操作符
greeting = "Hello" + " " + "World"
print(greeting)  # Hello World

# 使用 join() 方法（更高效）
words = ["Hello", "World", "Python"]
sentence = " ".join(words)
print(sentence)  # Hello World Python

# 使用 * 操作符重复字符串
repeat = "ha" * 3
print(repeat)  # hahaha
```

### 字符串长度

```python
s = "Hello World"
length = len(s)
print(length)  # 11
```

## 3. 字符串索引和切片

### 索引访问

```python
s = "Python"

# 正向索引（从0开始）
print(s[0])   # P
print(s[2])   # t
print(s[-1])  # n（倒数第一个）
print(s[-2])  # o（倒数第二个）
```

### 切片操作

```python
s = "Hello World"

# 基本切片 [start:end]
print(s[0:5])    # Hello
print(s[6:])     # World
print(s[:5])     # Hello
print(s[-5:])    # World

# 带步长的切片 [start:end:step]
print(s[::2])    # HloWrd（每隔一个字符）
print(s[::-1])   # dlroW olleH（反转字符串）
print(s[0:5:2])  # Hlo（前5个字符，步长为2）
```

## 4. 字符串方法

### 大小写转换

```python
s = "Hello World"

print(s.upper())       # HELLO WORLD
print(s.lower())       # hello world
print(s.capitalize())  # Hello world
print(s.title())       # Hello World
print(s.swapcase())    # hELLO wORLD
```

### 查找和替换

```
s = "Hello World, Hello Python"

# 查找
print(s.find("World"))     # 6（返回索引，找不到返回-1）
print(s.index("World"))    # 6（返回索引，找不到抛出异常）
print(s.rfind("Hello"))    # 13（从右边开始查找）
print(s.count("Hello"))    # 2（出现次数）

# 替换
print(s.replace("Hello", "Hi"))  # Hi World, Hi Python
print(s.replace("Hello", "Hi", 1))  # 只替换第一个：Hi World, Hello Python
```

### 字符串判断

```python
s = "Hello123"

print(s.isalpha())      # False（是否全为字母）
print(s.isdigit())      # False（是否全为数字）
print(s.isalnum())      # True（是否全为字母或数字）
print(s.islower())      # False（是否全小写）
print(s.isupper())      # False（是否全大写）
print(s.isspace())      # False（是否全为空白字符）
print(s.startswith("Hello"))  # True
print(s.endswith("123"))      # True
```

### 去除空白字符

```python
s = "   Hello World   "

print(s.strip())     # "Hello World"（去除两端空白）
print(s.lstrip())    # "Hello World   "（去除左边空白）
print(s.rstrip())    # "   Hello World"（去除右边空白）

# 去除特定字符
s2 = "xxxHello Worldxxx"
print(s2.strip('x'))  # "Hello World"
```

### 分割和连接

```python
# 分割字符串
s = "apple,banana,orange"
fruits = s.split(",")
print(fruits)  # ['apple', 'banana', 'orange']

# 按行分割
multiline = "Line1\nLine2\nLine3"
lines = multiline.splitlines()
print(lines)  # ['Line1', 'Line2', 'Line3']

# 连接字符串
words = ["Hello", "World", "Python"]
sentence = " ".join(words)
print(sentence)  # Hello World Python
```

### 对齐和填充

```python
s = "Hello"

print(s.ljust(10, "-"))  # Hello-----（左对齐）
print(s.rjust(10, "-"))  # -----Hello（右对齐）
print(s.center(10, "-")) # --Hello---（居中对齐）

# 用0填充数字
num = "42"
print(num.zfill(5))  # 00042
```

## 5. 字符串格式化

### % 格式化（传统方式）

```python
name = "Alice"
age = 25
print("My name is %s and I am %d years old." % (name, age))

# 常用格式化符号：
# %s - 字符串
# %d - 整数
# %f - 浮点数
# %.2f - 保留两位小数
```

### str.format() 方法

```python
name = "Alice"
age = 25

# 位置参数
print("My name is {} and I am {} years old.".format(name, age))

# 索引参数
print("My name is {0} and I am {1} years old. Hello {0}!".format(name, age))

# 命名参数
print("My name is {name} and I am {age} years old.".format(name=name, age=age))

# 格式规范
pi = 3.14159
print("Pi is approximately {:.2f}".format(pi))  # 保留两位小数
```

### f-string（Python 3.6+ 推荐）

```python
name = "Alice"
age = 25

# 基本用法
print(f"My name is {name} and I am {age} years old.")

# 表达式
print(f"Next year I will be {age + 1} years old.")

# 方法调用
print(f"My name in uppercase: {name.upper()}")

# 格式规范
pi = 3.14159
print(f"Pi: {pi:.2f}")          # 保留两位小数
print(f"Number: {1000:,}")      # 千分位分隔符
print(f"Percent: {0.25:.1%}")   # 百分比
```

## 6. 转义字符

### 常用转义字符

```python
# 换行符
print("Hello\nWorld")

# 制表符
print("Name:\tAlice")

# 反斜杠
print("This is a backslash: \\")

# 引号
print('She said: "Hello"')
print("She said: \"Hello\"")
print('It\'s a beautiful day')

# 原始字符串（忽略转义）
path = r"C:\Users\Name\Folder"
print(path)  # C:\Users\Name\Folder
```

## 7. 字符串编码

### 编码和解码

```python
# 字符串编码为字节
s = "Hello, 世界!"
utf8_bytes = s.encode('utf-8')
print(utf8_bytes)  # b'Hello, \xe4\xb8\x96\xe7\x95\x8c!'

# 字节解码为字符串
decoded_str = utf8_bytes.decode('utf-8')
print(decoded_str)  # Hello, 世界!

# 不同编码
ascii_bytes = s.encode('ascii', errors='ignore')  # 忽略无法编码的字符
print(ascii_bytes)  # b'Hello, !'
```

## 8. 实用技巧和示例

### 检查回文

```python
def is_palindrome(s):
    s = s.lower().replace(" ", "")
    return s == s[::-1]

print(is_palindrome("A man a plan a canal Panama"))  # True
```

### 统计字符频率

```python
from collections import Counter

s = "hello world"
char_count = Counter(s)
print(char_count)  # Counter({'l': 3, 'o': 2, 'h': 1, 'e': 1, ' ': 1, 'w': 1, 'r': 1, 'd': 1})
```

### 字符串验证

```python
def is_valid_email(email):
    return '@' in email and '.' in email and len(email) > 5

print(is_valid_email("test@example.com"))  # True
```

### 文本处理

```python
# 清理和标准化文本
text = "  Hello, World!  "
cleaned = text.strip().replace(",", "").lower()
print(cleaned)  # hello world!

# 提取数字
s = "Price: $123.45"
import re
numbers = re.findall(r'\d+\.\d+', s)
print(numbers)  # ['123.45']
```

## 9. 性能考虑

### 字符串连接效率比较

```python
# 低效：每次 + 操作都创建新字符串
result = ""
for i in range(1000):
    result += str(i)  # 不推荐

# 高效：使用列表和join()
parts = []
for i in range(1000):
    parts.append(str(i))
result = "".join(parts)  # 推荐
```

## 10. 实际应用示例

### 配置文件解析

```python
config_text = """
host=localhost
port=8080
debug=true
"""

config = {}
for line in config_text.strip().split('\n'):
    if '=' in line:
        key, value = line.split('=', 1)
        config[key.strip()] = value.strip()

print(config)  # {'host': 'localhost', 'port': '8080', 'debug': 'true'}
```

### 模板生成

```python
def generate_email_template(name, product):
    template = """
Dear {name},

Thank you for purchasing {product}!
We hope you enjoy it.

Best regards,
The Team
"""
    return template.format(name=name, product=product)

print(generate_email_template("Alice", "Python Course"))
```

## 总结

字符串是Python编程中最基础也是最重要的数据类型之一：

- 丰富的内置方法满足各种文本处理需求
- 多种格式化方式让字符串操作更加灵活
- 理解字符串的不可变性有助于编写高效代码
- 掌握编码知识对于处理多语言文本至关重要

熟练掌握字符串操作是成为Python开发者的必备技能。