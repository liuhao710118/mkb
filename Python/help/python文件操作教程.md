# 🐍 Python 文件操作教程

## 一、打开文件（open）

```python
f = open("test.txt", "r")   # r 只读
f = open("test.txt", "w")   # w 写入（覆盖）
f = open("test.txt", "a")   # a 追加
f = open("test.txt", "rb")  # rb 以二进制方式读取
f = open("test.txt", "wb")  # wb 以二进制方式写入
```

**推荐使用 with 自动关闭文件：**

```python
with open("test.txt", "r") as f:
    data = f.read()
```

------

## 二、读取文件

### 1. 一次性读取所有内容

```python
with open("test.txt", "r") as f:
    content = f.read()
    print(content)
```

### 2. 按行读取

```python
with open("test.txt", "r") as f:
    lines = f.readlines()
    print(lines)
```

### 3. 逐行读取（大文件推荐）

```python
with open("test.txt", "r") as f:
    for line in f:
        print(line.strip())
```

------

## 三、写入文件

### 1. 覆盖写入

```python
with open("test.txt", "w") as f:
    f.write("Hello World")
```

### 2. 追加写入

```python
with open("test.txt", "a") as f:
    f.write("\nNew Line")
```

------

## 四、文件定位（指针）

```python
with open("test.txt", "r") as f:
    f.seek(5)       # 移动到第 5 个字节
    print(f.read()) # 从第 5 个字节开始读
```

查看当前指针位置：

```python
f.tell()
```

------

## 五、判断文件是否存在（os / pathlib）

### 使用 `os.path`

```python
import os

if os.path.exists("test.txt"):
    print("文件存在")
```

### 使用 pathlib（推荐）

```python
from pathlib import Path

p = Path("test.txt")
print(p.exists())
```

------

## 六、删除、重命名、创建文件夹

```python
import os

os.remove("test.txt")                  # 删除文件
os.rename("old.txt", "new.txt")        # 重命名文件
os.mkdir("dir")                        # 创建目录
os.rmdir("dir")                        # 删除空目录
os.makedirs("a/b/c")                   # 创建多层
```

**删除非空目录（慎用！）**

```python
import shutil
shutil.rmtree("dir")
```

------

## 七、读取/写入二进制文件（如图片、压缩包）

读取图片：

```python
with open("image.jpg", "rb") as f:
    data = f.read()
```

写入二进制：

```python
with open("copy.jpg", "wb") as f:
    f.write(data)
```

------

## 八、Pathlib 更优雅的路径操作（强烈推荐）

```python
from pathlib import Path

p = Path("data/test.txt")

# 父目录
print(p.parent)

# 文件名
print(p.name)

# 后缀
print(p.suffix)

# 遍历目录
for file in Path("logs").iterdir():
    print(file)
```

------

## 九、读取大文件技巧（逐块读取）

```python
with open("bigfile.log", "r") as f:
    while chunk := f.read(1024):  # 每次 1KB
        print(chunk)
```

------

## 十、JSON 文件读写（常用）

写入 JSON：

```python
import json

data = {"name": "Tom", "age": 20}

with open("data.json", "w", encoding="utf-8") as f:
    json.dump(data, f, ensure_ascii=False, indent=4)
```

读取 JSON：

```python
with open("data.json", "r", encoding="utf-8") as f:
    data = json.load(f)
```

------

