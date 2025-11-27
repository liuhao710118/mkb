# Python 正则表达式(re模块)完整教程

正则表达式是处理文本的强大工具，Python通过`re`模块提供正则表达式支持。

## 1. 基本用法

### 导入模块

```python
import re
```

## 2. 常用函数

### 2.1 匹配查找

#### `re.match()`- 从字符串开头匹配

```python
import re

# 从开头匹配数字
result = re.match(r'\d+', '123abc')
if result:
    print("匹配成功:", result.group())  # 输出: 123
else:
    print("匹配失败")

# 匹配失败示例
result = re.match(r'\d+', 'abc123')
print(result)  # 输出: None
```

#### `re.search()`- 搜索第一个匹配

```python
text = "我的电话是123-456-7890，另一个是098-765-4321"
result = re.search(r'\d{3}-\d{3}-\d{4}', text)
if result:
    print("找到电话:", result.group())  # 输出: 123-456-7890
    print("匹配位置:", result.start(), "-", result.end())  # 匹配的起止位置
```

#### `re.findall()`- 查找所有匹配

```python
text = "价格: $10, $20, $30"
prices = re.findall(r'\$\d+', text)
print(prices)  # 输出: ['$10', '$20', '$30']

# 提取分组内容
text = "姓名: 张三, 年龄: 25; 姓名: 李四, 年龄: 30"
info = re.findall(r'姓名: (\w+), 年龄: (\d+)', text)
print(info)  # 输出: [('张三', '25'), ('李四', '30')]
```

#### `re.finditer()`- 返回迭代器

```python
text = "苹果10元, 香蕉20元, 橙子15元"
for match in re.finditer(r'(\w+)(\d+)元', text):
    print(f"商品: {match.group(1)}, 价格: {match.group(2)}")
# 输出:
# 商品: 苹果, 价格: 10
# 商品: 香蕉, 价格: 20
# 商品: 橙子, 价格: 15
```

### 2.2 替换操作

#### `re.sub()`- 基本替换

```python
text = "2023-01-15"
# 替换日期格式
result = re.sub(r'(\d{4})-(\d{2})-(\d{2})', r'\2/\3/\1', text)
print(result)  # 输出: 01/15/2023

# 使用函数进行复杂替换
def double_numbers(match):
    num = int(match.group())
    return str(num * 2)

text = "数字: 1, 2, 3, 4"
result = re.sub(r'\d+', double_numbers, text)
print(result)  # 输出: 数字: 2, 4, 6, 8
```

#### `re.subn()`- 替换并返回次数

```python
text = "hello world, hello python"
result, count = re.subn(r'hello', 'Hi', text)
print(f"结果: {result}, 替换次数: {count}")
# 输出: 结果: Hi world, Hi python, 替换次数: 2
```

### 2.3 分割字符串

#### `re.split()`- 正则分割

```python
# 按多种分隔符分割
text = "苹果,香蕉;橙子 西瓜"
result = re.split(r'[,; ]', text)
print(result)  # 输出: ['苹果', '香蕉', '橙子', '西瓜']

# 包含分隔符
text = "1+2-3 * 4"
result = re.split(r'([+-])', text)  # 括号保留分隔符
print(result)  # 输出: ['1', '+', '2', '-', '3 * 4']
```

## 3. 正则表达式语法

### 3.1 基本元字符

```python
# 测试函数
def test_pattern(pattern, text):
    matches = re.findall(pattern, text)
    print(f"模式: {pattern} -> 匹配: {matches}")

text = "abc123 XYZ!@# 测试"

# 常用元字符
test_pattern(r'.', text)        # 匹配任意字符(除换行符)
test_pattern(r'\d', text)       # 匹配数字: [0-9]
test_pattern(r'\D', text)       # 匹配非数字: [^0-9]
test_pattern(r'\w', text)       # 匹配单词字符: [a-zA-Z0-9_]
test_pattern(r'\W', text)       # 匹配非单词字符
test_pattern(r'\s', text)       # 匹配空白字符
test_pattern(r'\S', text)       # 匹配非空白字符
```

### 3.2 量词和重复

```python
text = "a ab abbb abbbb abc"

test_pattern(r'ab', text)        # 匹配ab
test_pattern(r'ab?', text)      # ?: 0次或1次
test_pattern(r'ab*', text)      # *: 0次或多次
test_pattern(r'ab+', text)      # +: 1次或多次
test_pattern(r'ab{2}', text)    # {n}: 恰好n次
test_pattern(r'ab{2,}', text)   # {n,}: 至少n次
test_pattern(r'ab{2,3}', text)  # {n,m}: n到m次
```

### 3.3 字符类和边界

```python
text = "cat bat hat rat at @at"

# 字符类
test_pattern(r'[cb]at', text)   # 匹配cat或bat
test_pattern(r'[^cb]at', text)  # 匹配不以c或b开头的at
test_pattern(r'[a-z]at', text)  # 匹配小写字母开头的at

# 边界匹配
text = "the theme then they the"
test_pattern(r'the', text)           # 所有the
test_pattern(r'\bthe\b', text)       # 单词边界
test_pattern(r'\Bthe\B', text)       # 非单词边界
```

### 3.4 分组和捕获

```python
text = "2023-01-15 2024-12-25"

# 分组捕获
matches = re.findall(r'(\d{4})-(\d{2})-(\d{2})', text)
print(matches)  # 输出: [('2023', '01', '15'), ('2024', '12', '25')]

# 非捕获分组 (?:pattern)
matches = re.findall(r'(?:\d{4})-(?:\d{2})-(?:\d{2})', text)
print(matches)  # 输出: ['2023-01-15', '2024-12-25']

# 命名分组 (?P<name>pattern)
match = re.search(r'(?P<year>\d{4})-(?P<month>\d{2})-(?P<day>\d{2})', text)
if match:
    print(f"年: {match.group('year')}, 月: {match.group('month')}, 日: {match.group('day')}")
```

### 3.5 贪婪与非贪婪匹配

```python
text = "<div>内容1</div><div>内容2</div>"

# 贪婪匹配(默认)
greedy = re.findall(r'<div>.*</div>', text)
print("贪婪匹配:", greedy)  # 匹配整个字符串

# 非贪婪匹配
non_greedy = re.findall(r'<div>.*?</div>', text)
print("非贪婪匹配:", non_greedy)  # 匹配两个div标签
```

## 4. 标志参数(flags)

```python
text = "Hello WORLD\nhello python"

# 忽略大小写
result = re.findall(r'hello', text, re.IGNORECASE)
print("忽略大小写:", result)  # 输出: ['Hello', 'hello']

# 多行模式
result = re.findall(r'^hello', text, re.IGNORECASE | re.MULTILINE)
print("多行模式:", result)  # 输出: ['Hello', 'hello']

# 点号匹配换行
text = "line1\nline2"
result = re.findall(r'line1.line2', text, re.DOTALL)
print("点号匹配换行:", result)  # 输出: ['line1\nline2']
```

## 5. 编译正则表达式

对于需要重复使用的正则表达式，可以先编译提高效率：

```python
# 编译正则表达式
pattern = re.compile(r'\b\w+\b')  # 匹配单词

text = "Hello world, welcome to Python!"

# 使用编译后的模式
matches = pattern.findall(text)
print(matches)  # 输出: ['Hello', 'world', 'welcome', 'to', 'Python']

# 其他方法
result = pattern.sub('WORD', text)
print(result)  # 输出: WORD WORD, WORD WORD WORD!
```

## 6. 实用示例

### 6.1 验证邮箱格式

```python
def validate_email(email):
    pattern = r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'
    return bool(re.match(pattern, email))

emails = ["test@example.com", "invalid.email", "user@domain.co.uk"]
for email in emails:
    print(f"{email}: {'有效' if validate_email(email) else '无效'}")
```

### 6.2 提取网页链接

```python
html = '''
<a href="https://example.com">链接1</a>
<a href="http://test.org/page.html">链接2</a>
<img src="image.jpg">
'''

# 提取所有链接
links = re.findall(r'href="(https?://[^"]+)"', html)
print("网页链接:", links)

# 提取图片链接
images = re.findall(r'src="([^"]+\.(jpg|png|gif))"', html)
print("图片链接:", images)
```

### 6.3 数据清洗

```python
def clean_text(text):
    # 移除多余空格
    text = re.sub(r'\s+', ' ', text)
    # 移除特殊字符(保留中文、英文、数字和常用标点)
    text = re.sub(r'[^\u4e00-\u9fa5a-zA-Z0-9\s.,!?;:]', '', text)
    # 标准化标点
    text = re.sub(r'[，,]+', ',', text)
    text = re.sub(r'[。.]+', '.', text)
    return text.strip()

dirty_text = "这是  一个,,测试文本！！包含@#特殊字符。。。"
clean = clean_text(dirty_text)
print("清洗后:", clean)  # 输出: 这是 一个,测试文本!包含特殊字符.
```

### 6.4 日志分析

```python
log_data = '''
2023-01-15 10:30:15 INFO User login successful
2023-01-15 10:31:20 ERROR Database connection failed
2023-01-15 10:32:05 WARNING Disk space low
'''

# 提取错误日志
pattern = r'(\d{4}-\d{2}-\d{2} \d{2}:\d{2}:\d{2}) (ERROR) (.+)'
errors = re.findall(pattern, log_data)
for error in errors:
    print(f"时间: {error[0]}, 级别: {error[1]}, 信息: {error[2]}")
```

## 7. 常见问题与技巧

### 7.1 转义特殊字符

```python
import re

# 错误方式(需要转义)
text = "1 + 2 = 3"
# result = re.findall('1 + 2 = 3', text)  # 错误!

# 正确方式
result = re.findall(r'1 \+ 2 = 3', text)  # 使用原始字符串和转义
print(result)  # 输出: ['1 + 2 = 3']

# 或者使用re.escape自动转义
pattern = re.escape('1 + 2 = 3')
result = re.findall(pattern, text)
print(result)  # 输出: ['1 + 2 = 3']
```

### 7.2 性能优化

```python
# 不好的做法(每次编译)
def slow_search(text, pattern):
    return re.findall(pattern, text)

# 好的做法(预编译)
class TextProcessor:
    def __init__(self):
        self.patterns = {
            'email': re.compile(r'\b[\w.-]+@[\w.-]+\.\w+\b'),
            'phone': re.compile(r'\b\d{3}-\d{3}-\d{4}\b'),
            'url': re.compile(r'https?://[^\s]+')
        }
    
    def extract_info(self, text):
        results = {}
        for key, pattern in self.patterns.items():
            results[key] = pattern.findall(text)
        return results

processor = TextProcessor()
text = "联系: email@example.com, 电话: 123-456-7890, 网站: https://example.com"
print(processor.extract_info(text))
```



