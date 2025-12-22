下面给你一份 **Python `datetime` 模块的系统使用教程**，从基础到常用实战，适合日常开发直接查用 👍

------

## 一、`datetime` 模块简介

`datetime` 是 Python 标准库中用于 **日期和时间处理** 的模块，常用于：

- 获取当前时间
- 时间格式化 / 解析
- 时间加减（计算间隔）
- 时间戳转换
- 比较时间大小

```python
import datetime
```

------

## 二、`datetime` 模块的核心类

| 类名        | 作用                        |
| ----------- | --------------------------- |
| `date`      | 只表示日期（年/月/日）      |
| `time`      | 只表示时间（时/分/秒/微秒） |
| `datetime`  | 日期 + 时间（最常用）       |
| `timedelta` | 时间间隔                    |
| `tzinfo`    | 时区信息（进阶）            |

------

## 三、获取当前日期和时间

### 1️⃣ 当前日期

```python
from datetime import date

today = date.today()
print(today)          # 2025-12-18
print(today.year)     # 年
print(today.month)    # 月
print(today.day)      # 日
```

------

### 2️⃣ 当前时间（推荐）

```python
from datetime import datetime

now = datetime.now()
print(now)
# 2025-12-18 16:03:25.123456
```

👉 常用属性：

```python
now.year
now.month
now.day
now.hour
now.minute
now.second
```

------

### 3️⃣ 当前 UTC 时间

```python
datetime.utcnow()
```

⚠️ **无时区信息，只是数值不同**

------

## 四、创建指定时间

### 1️⃣ 创建 `date`

```python
from datetime import date

d = date(2025, 12, 18)
```

------

### 2️⃣ 创建 `datetime`

```python
from datetime import datetime

dt = datetime(2025, 12, 18, 16, 30, 0)
```

------

## 五、时间格式化（非常常用）

### 1️⃣ datetime → 字符串（`strftime`）

```python
now = datetime.now()

formatted = now.strftime("%Y-%m-%d %H:%M:%S")
print(formatted)
# 2025-12-18 16:05:30
```

#### 常见格式符

| 格式 | 含义        |
| ---- | ----------- |
| `%Y` | 年          |
| `%m` | 月          |
| `%d` | 日          |
| `%H` | 小时（24h） |
| `%M` | 分钟        |
| `%S` | 秒          |
| `%f` | 微秒        |
| `%A` | 星期        |
| `%w` | 星期（0-6） |

------

### 2️⃣ 字符串 → datetime（`strptime`）

```python
from datetime import datetime

s = "2025-12-18 16:30:00"
dt = datetime.strptime(s, "%Y-%m-%d %H:%M:%S")
```

------

## 六、时间加减（`timedelta`）

### 1️⃣ 基本用法

```python
from datetime import datetime, timedelta

now = datetime.now()

# 加 1 天
now + timedelta(days=1)

# 减 2 小时
now - timedelta(hours=2)
```

------

### 2️⃣ 常用参数

```python
timedelta(
    days=1,
    hours=2,
    minutes=30,
    seconds=10
)
```

------

### 3️⃣ 计算两个时间差

```python
t1 = datetime(2025, 12, 18, 12, 0)
t2 = datetime(2025, 12, 18, 16, 0)

delta = t2 - t1
print(delta)              # 4:00:00
print(delta.seconds)      # 14400
print(delta.total_seconds())  # 14400.0
```

------

## 七、时间戳（timestamp）

### 1️⃣ 当前时间戳

```python
datetime.now().timestamp()
```

👉 返回 **秒（float）**

------

### 2️⃣ 时间戳 → datetime

```python
ts = 1702886400
dt = datetime.fromtimestamp(ts)
```

------

### 3️⃣ datetime → 时间戳

```python
dt.timestamp()
```

------

## 八、时间比较

```python
t1 = datetime(2025, 12, 18, 10, 0)
t2 = datetime(2025, 12, 18, 12, 0)

print(t1 < t2)   # True
print(t1 == t2)  # False
```

------

## 九、常见实战示例

### ✅ 获取今天的开始和结束时间

```python
from datetime import datetime

start = datetime.now().replace(hour=0, minute=0, second=0, microsecond=0)
end = datetime.now().replace(hour=23, minute=59, second=59, microsecond=999999)
```

------

### ✅ 最近 24 小时（你之前做监控统计时会用到）

```python
now = datetime.now()
last_24h = now - timedelta(hours=24)
```

------

### ✅ 判断是否超时（例如 5 分钟）

```python
if datetime.now() - last_time > timedelta(minutes=5):
    print("已超时")
```

------

## 十、容易踩坑的点 ⚠️

1. **字符串 ≠ datetime**（比较前一定要转）
2. `utcnow()` ≠ 带时区的 UTC
3. 时间戳是 **秒，不是毫秒**
4. `timedelta.seconds` ≠ 总秒数（用 `total_seconds()`）

------

## 十一、总结一句话

> **记住三板斧：**
>
> - 获取时间：`datetime.now()`
> - 格式化：`strftime / strptime`
> - 计算间隔：`timedelta`

------

