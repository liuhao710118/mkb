# 📘 Python 时间教程（完整版）

## 目录

1. `time` 模块（时间戳、结构化时间）
2. `datetime` 模块（日期时间对象）
3. 时间格式化与解析（`strftime` / `strptime`）
4. 时区处理（`zoneinfo` / `pytz`）
5. 时间运算（加减时间）
6. 常用案例（日志、计时器、日期范围等）

------

# 1️⃣ `time` 模块

适合低层系统时间处理。

### 1.1 获取当前时间戳（秒）

```python
import time
print(time.time())  # 例如 1733383103.123456
```

### 1.2 时间戳 → struct_time

```python
t = time.localtime(time.time())
print(t)
```

### 1.3 struct_time → 时间戳

```python
ts = time.mktime(t)
```

### 1.4 获取格式化时间字符串

```python
time_str = time.strftime("%Y-%m-%d %H:%M:%S", time.localtime())
```

### 1.5 时间字符串 → struct_time

```python
t = time.strptime("2025-12-04 10:20:30", "%Y-%m-%d %H:%M:%S")
```

### 1.6 延时

```python
time.sleep(2)  # 休眠2秒
```

------

# 2️⃣ `datetime` 模块（推荐）

更高层、更方便。

### 2.1 当前时间

```python
from datetime import datetime
now = datetime.now()
print(now)
```

### 2.2 指定日期时间

```python
dt = datetime(2025, 12, 4, 10, 20, 30)
```

### 2.3 转换为时间戳

```python
timestamp = dt.timestamp()
```

### 2.4 时间戳 → datetime

```python
datetime.fromtimestamp(1733383103)
```

------

# 3️⃣ 时间格式化/解析

### 3.1 datetime → 字符串（格式化）

```python
now.strftime("%Y-%m-%d %H:%M:%S")
```

常用格式：

| 符号 | 含义          |
| ---- | ------------- |
| `%Y` | 4位年份       |
| `%m` | 月            |
| `%d` | 日            |
| `%H` | 小时(24h)     |
| `%M` | 分钟          |
| `%S` | 秒            |
| `%f` | 微秒          |
| `%A` | 星期名称      |
| `%w` | 星期数字(0-6) |

### 3.2 字符串 → datetime

```python
datetime.strptime("2025-12-04 16:00:00", "%Y-%m-%d %H:%M:%S")
```

------

# 4️⃣ 时区处理（Python 3.9+ 推荐 zoneinfo）

### 4.1 获取本地时间（含时区）

```python
from datetime import datetime
from zoneinfo import ZoneInfo

now = datetime.now(ZoneInfo("Asia/Shanghai"))
```

### 4.2 时区转换

```python
dt = datetime.now(ZoneInfo("Asia/Shanghai"))
new_dt = dt.astimezone(ZoneInfo("UTC"))
```

常见时区名称：

- `"UTC"`
- `"Asia/Shanghai"`
- `"America/New_York"`
- `"Europe/London"`

------

# 5️⃣ 时间加减（timedelta）

### 5.1 加减时间

```python
from datetime import timedelta

now = datetime.now()
print(now + timedelta(days=7))     # 加 7 天
print(now - timedelta(hours=3))    # 减 3 小时
```

### 5.2 两个时间差值

```python
delta = datetime(2025, 12, 4) - datetime(2025, 1, 1)
print(delta.days)
```

------

# 6️⃣ 常用案例

## 📌 案例 1：统计代码运行耗时

```python
import time

start = time.perf_counter()
# 执行某些代码
end = time.perf_counter()
print("耗时:", end - start)
```

## 📌 案例 2：获取某天的 0 点时间

```python
from datetime import datetime

today_zero = datetime.combine(datetime.today(), datetime.min.time())
```

## 📌 案例 3：生成日期范围（用于报表、任务调度）

```python
from datetime import datetime, timedelta

start = datetime(2025, 1, 1)
end = datetime(2025, 1, 10)

while start <= end:
    print(start.strftime("%Y-%m-%d"))
    start += timedelta(days=1)
```

## 📌 案例 4：获取最近 24 小时每小时的时间段

```python
from datetime import datetime, timedelta

now = datetime.now()
for i in range(24):
    print((now - timedelta(hours=i)).strftime("%Y-%m-%d %H:00:00"))
```

## 📌 案例 5：将“2025-12-04 13:20:10”转换为 UTC 时间

```python
from datetime import datetime
from zoneinfo import ZoneInfo

dt = datetime.strptime("2025-12-04 13:20:10", "%Y-%m-%d %H:%M:%S")
dt = dt.replace(tzinfo=ZoneInfo("Asia/Shanghai"))
print(dt.astimezone(ZoneInfo("UTC")))
```

------

# 📚 总结

Python 时间模块你需要记住三件事：

1. **系统时间（结构化时间） → `time` 模块**
2. **日期时间对象 → `datetime` 模块（最常用）**
3. **时区 → `zoneinfo`（3.9+）或 `pytz`**

