# 🧾 datetime 常用速查表

------

## 一、常用导入方式（推荐）

```python
from datetime import datetime, date, time, timedelta
```

------

## 二、获取当前时间

| 需求          | 写法                         |
| ------------- | ---------------------------- |
| 当前本地时间  | `datetime.now()`             |
| 当前 UTC 时间 | `datetime.utcnow()`          |
| 今天日期      | `date.today()`               |
| 当前时间戳    | `datetime.now().timestamp()` |

------

## 三、创建时间对象

### 1️⃣ date

```python
date(2025, 12, 18)
```

### 2️⃣ datetime

```python
datetime(2025, 12, 18, 16, 30, 0)
```

------

## 四、datetime 常用属性

```python
dt = datetime.now()
```

| 属性              | 含义           |
| ----------------- | -------------- |
| `dt.year`         | 年             |
| `dt.month`        | 月             |
| `dt.day`          | 日             |
| `dt.hour`         | 时             |
| `dt.minute`       | 分             |
| `dt.second`       | 秒             |
| `dt.microsecond`  | 微秒           |
| `dt.weekday()`    | 星期（0=周一） |
| `dt.isoweekday()` | 星期（1=周一） |

------

## 五、字符串 ↔ datetime

### 1️⃣ datetime → 字符串（strftime）

```python
dt.strftime("%Y-%m-%d %H:%M:%S")
```

#### 常用格式符速查

| 格式 | 含义     | 示例      |
| ---- | -------- | --------- |
| `%Y` | 年       | 2025      |
| `%m` | 月       | 12        |
| `%d` | 日       | 18        |
| `%H` | 24小时   | 16        |
| `%I` | 12小时   | 04        |
| `%M` | 分       | 30        |
| `%S` | 秒       | 05        |
| `%f` | 微秒     | 123456    |
| `%a` | 星期简写 | Wed       |
| `%A` | 星期全称 | Wednesday |

------

### 2️⃣ 字符串 → datetime（strptime）

```python
datetime.strptime("2025-12-18 16:30:00", "%Y-%m-%d %H:%M:%S")
```

------

## 六、时间加减（timedelta）

```python
timedelta(days=1, hours=2, minutes=30, seconds=10)
```

| 需求         | 示例                         |
| ------------ | ---------------------------- |
| 明天         | `now + timedelta(days=1)`    |
| 昨天         | `now - timedelta(days=1)`    |
| 5 分钟后     | `now + timedelta(minutes=5)` |
| 最近 24 小时 | `now - timedelta(hours=24)`  |

------

## 七、时间差计算

```python
delta = end - start
```

| 用法                    | 说明           |
| ----------------------- | -------------- |
| `delta.days`            | 天数           |
| `delta.seconds`         | 秒（不含天）⚠️  |
| `delta.total_seconds()` | 总秒数（推荐） |

------

## 八、时间戳（timestamp）

### 1️⃣ datetime → 时间戳

```python
dt.timestamp()
```

### 2️⃣ 时间戳 → datetime

```python
datetime.fromtimestamp(1702886400)
```

------

## 九、时间比较

```python
dt1 < dt2
dt1 > dt2
dt1 == dt2
```

👉 可直接比较，无需转换

------

## 十、常用 replace 技巧

### 1️⃣ 今天 00:00:00

```python
datetime.now().replace(hour=0, minute=0, second=0, microsecond=0)
```

### 2️⃣ 今天 23:59:59

```python
datetime.now().replace(hour=23, minute=59, second=59, microsecond=999999)
```

------

## 十一、判断是否超时

```python
if datetime.now() - last_time > timedelta(minutes=5):
    print("超时")
```

------

## 十二、date 与 datetime 互转

```python
# date → datetime
datetime.combine(date.today(), time.min)

# datetime → date
dt.date()
```

------

## 十三、常见坑速记 ⚠️

| 坑                       | 正确方式             |
| ------------------------ | -------------------- |
| 字符串直接比较时间       | 先 `strptime`        |
| `utcnow()` 当作带时区    | ❌（无 tzinfo）       |
| `delta.seconds` 当总秒数 | 用 `total_seconds()` |
| 毫秒时间戳               | ÷ 1000               |

