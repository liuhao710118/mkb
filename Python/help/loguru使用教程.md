# Loguru 详细使用教程

## 1. 安装

```
pip install loguru
```

## 2. 基本使用

### 最简单的用法

```
from loguru import logger

logger.debug("这是一条调试信息")
logger.info("这是一条普通信息")
logger.success("这是一条成功信息")
logger.warning("这是一条警告信息")
logger.error("这是一条错误信息")
logger.critical("这是一条严重错误信息")
```

### 输出示例

```
2024-01-15 10:30:15.123 | INFO     | __main__:<module>:5 - 这是一条普通信息
2024-01-15 10:30:15.124 | SUCCESS  | __main__:<module>:6 - 这是一条成功信息
```

## 3. 配置和自定义

### 添加控制台输出配置

```
import sys
from loguru import logger

# 移除默认配置
logger.remove()

# 添加控制台输出，自定义格式
logger.add(
    sys.stdout,
    format="<green>{time:YYYY-MM-DD HH:mm:ss}</green> | <level>{level: <8}</level> | <cyan>{name}</cyan>:<cyan>{function}</cyan>:<cyan>{line}</cyan> - <level>{message}</level>",
    level="DEBUG"
)

logger.info("自定义格式的日志")
```

### 添加文件输出

```
# 添加文件输出
logger.add(
    "app.log",
    format="{time:YYYY-MM-DD HH:mm:ss} | {level: <8} | {name}:{function}:{line} - {message}",
    level="DEBUG",
    encoding="utf-8"
)

logger.info("这条日志会同时输出到控制台和文件")
```

## 4. 高级功能

### 日志轮转和保留

```
logger.add(
    "file_{time}.log",  # 按时间分割文件
    rotation="10 MB",   # 每10MB轮转一次
    retention="30 days", # 保留30天的日志
    compression="zip",   # 压缩旧日志
    level="DEBUG"
)

# 或者按时间轮转
logger.add(
    "app.log",
    rotation="00:00",    # 每天午夜轮转
    retention="1 month",
    level="INFO"
)
```

### 异常记录

```
# 自动记录异常堆栈
def risky_function():
    try:
        1 / 0
    except Exception:
        logger.exception("发生了一个异常")

risky_function()

# 或者使用装饰器自动捕获异常
@logger.catch
def another_risky_function():
    return 1 / 0

another_risky_function()
```

### 结构化日志

```
# 记录结构化数据
user_data = {"name": "张三", "age": 25, "city": "北京"}
logger.info("用户信息: {}", user_data)

# 使用上下文信息
logger.bind(user_id=123, action="login").info("用户登录")
```

## 5. 过滤器

### 按级别过滤

```
# 只记录ERROR及以上级别的日志到文件
logger.add(
    "error.log",
    level="ERROR",
    filter=lambda record: record["level"].name == "ERROR"
)
```

### 自定义过滤器

```
def my_filter(record):
    # 只记录包含特定关键词的日志
    return "important" in record["message"].lower()

logger.add("important.log", filter=my_filter, level="INFO")
logger.info("这是一条重要的日志")
logger.info("这是普通日志")  # 这条不会被记录到important.log
```

## 6. 多日志文件配置

```
# 配置不同的日志文件用于不同用途
# 应用日志
app_logger = logger.add(
    "application.log",
    format="{time:YYYY-MM-DD HH:mm:ss} | {level} | {message}",
    level="INFO",
    rotation="10 MB"
)

# 错误日志
error_logger = logger.add(
    "errors.log",
    format="{time:YYYY-MM-DD HH:mm:ss} | {level} | {file}:{line} - {message}",
    level="ERROR",
    rotation="1 MB"
)

# 调试日志（仅在开发环境启用）
debug_logger = logger.add(
    "debug.log",
    format="{time:YYYY-MM-DD HH:mm:ss.SSS} | {level} | {thread} | {file}:{function}:{line} - {message}",
    level="DEBUG",
    rotation="50 MB"
)
```

## 7. 异步日志记录

```
import asyncio
from loguru import logger

# 配置异步写入
logger.add(
    "async_app.log",
    enqueue=True,  # 启用异步队列
    level="INFO"
)

async def async_task():
    for i in range(10):
        logger.info(f"异步任务执行中: {i}")
        await asyncio.sleep(0.1)

# 运行异步任务
asyncio.run(async_task())
```

## 8. 自定义日志级别

```
from loguru import logger

# 添加自定义级别
logger.level("PERFORMANCE", no=25, color="<magenta>")

# 使用自定义级别
logger.log("PERFORMANCE", "性能指标: 处理时间 150ms")

# 或者使用装饰器创建快捷方式
performance = logger.__class__.performance

def performance(self, message, *args, **kwargs):
    self.log("PERFORMANCE", message, *args, **kwargs)

logger.__class__.performance = performance

logger.performance("另一个性能日志")
```

## 9. 实际项目配置示例

```
# config/logger.py
import sys
from pathlib import Path
from loguru import logger

# 创建日志目录
LOG_DIR = Path("logs")
LOG_DIR.mkdir(exist_ok=True)

def setup_logger(level="INFO", rotation="10 MB", retention="30 days"):
    """设置项目日志配置"""
    
    # 移除默认处理器
    logger.remove()
    
    # 控制台输出配置
    logger.add(
        sys.stdout,
        format="<green>{time:YYYY-MM-DD HH:mm:ss}</green> | "
               "<level>{level: <8}</level> | "
               "<cyan>{name}</cyan>:<cyan>{function}</cyan>:<cyan>{line}</cyan> - "
               "<level>{message}</level>",
        level=level,
        colorize=True
    )
    
    # 主日志文件
    logger.add(
        LOG_DIR / "app.log",
        format="{time:YYYY-MM-DD HH:mm:ss} | {level: <8} | {name}:{function}:{line} - {message}",
        level=level,
        rotation=rotation,
        retention=retention,
        encoding="utf-8",
        enqueue=True  # 异步写入
    )
    
    # 错误日志文件
    logger.add(
        LOG_DIR / "error.log",
        format="{time:YYYY-MM-DD HH:mm:ss} | {level} | {name}:{function}:{line} - {message}\n{exception}",
        level="ERROR",
        rotation="5 MB",
        retention="90 days",
        encoding="utf-8",
        enqueue=True
    )
    
    return logger

# 初始化日志配置
app_logger = setup_logger(level="INFO")
```

## 10. 在模块中的使用

```
# module1.py
from loguru import logger

class DatabaseManager:
    def __init__(self):
        self.logger = logger.bind(module="DatabaseManager")
    
    def connect(self):
        self.logger.info("连接数据库")
        # 连接逻辑...
        self.logger.success("数据库连接成功")
    
    def query(self, sql):
        self.logger.debug("执行SQL查询: {}", sql)
        # 查询逻辑...
        return {"result": "data"}

# module2.py
from loguru import logger

class APIService:
    def __init__(self):
        self.logger = logger.bind(module="APIService")
    
    def process_request(self, request_data):
        self.logger.info("处理API请求", request_data=request_data)
        
        try:
            # 处理逻辑...
            result = {"status": "success"}
            self.logger.success("请求处理完成")
            return result
        except Exception as e:
            self.logger.error("请求处理失败: {}", e)
            raise
```

## 11. 性能优化配置

```
# 高性能场景下的配置
logger.add(
    "high_perf.log",
    format="{time:YYYY-MM-DD HH:mm:ss} | {level} | {message}",
    level="INFO",
    rotation="100 MB",
    compression="zip",
    enqueue=True,  # 启用异步
    backtrace=False,  # 禁用回溯（提高性能）
    diagnose=False   # 禁用诊断（提高性能）
)
```

## 12. 常用配置参数说明

| 参数          | 说明             | 示例值                    |
| ------------- | ---------------- | ------------------------- |
| `rotation`    | 日志轮转条件     | "10 MB", "1 day", "00:00" |
| `retention`   | 日志保留时间     | "30 days", "1 week"       |
| `compression` | 压缩格式         | "zip", "gz"               |
| `format`      | 日志格式         | 格式化字符串              |
| `level`       | 日志级别         | "DEBUG", "INFO", "ERROR"  |
| `filter`      | 过滤器函数       | lambda record: condition  |
| `enqueue`     | 是否异步         | True/False                |
| `serialize`   | 是否序列化为JSON | True/False                |
| `encoding`    | 文件编码         | "utf-8"                   |

这个教程涵盖了 loguru 的主要功能，从基础使用到高级配置都有详细示例。loguru 的设计哲学是"零配置开箱即用"，但同时也提供了丰富的自定义选项来满足复杂需求。