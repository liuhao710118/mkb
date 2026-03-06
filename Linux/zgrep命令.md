`zgrep` 是 Linux/Unix 系统中用来 **在压缩文件（通常是 `.gz`）中搜索文本的命令**，本质上是 **`grep` + `gzip`**。它可以直接在压缩文件里查找，而无需先解压。🔍

------

## 1️⃣ 基本语法

```bash
zgrep [选项] PATTERN 文件名.gz
```

- `PATTERN`：你要搜索的文本或正则表达式
- `文件名.gz`：gzip 压缩文件

### 示例

```bash
zgrep ERROR /var/log/nginx/access.log.1.gz
```

作用：

- 在 `access.log.1.gz` 中搜索所有包含 `ERROR` 的行
- 不需要解压文件

------

## 2️⃣ 常用选项

| 选项   | 说明                  |
| ------ | --------------------- |
| -i     | 忽略大小写            |
| -n     | 显示行号              |
| -c     | 只显示匹配行数        |
| -v     | 显示不匹配的行        |
| -A NUM | 匹配行之后显示 NUM 行 |
| -B NUM | 匹配行之前显示 NUM 行 |
| -C NUM | 显示匹配行前后 NUM 行 |

### 示例

```bash
zgrep -i "error" access.log.gz
```

忽略大小写搜索 `error`

```bash
zgrep -n "timeout" access.log.gz
```

显示行号

```bash
zgrep -C 3 "500" access.log.gz
```

显示匹配行及前后 3 行

------

## 3️⃣ 高级用法

### 1. 多个文件搜索

```bash
zgrep "ERROR" *.log.gz
```

### 2. 和管道组合

```bash
zgrep "timeout" access.log.gz | wc -l
```

统计匹配行数

```bash
zgrep "ERROR" access.log.gz | less
```

分页查看

### 3. 使用正则

```bash
zgrep -E "ERROR|FAIL" access.log.gz
```

匹配 `ERROR` 或 `FAIL`

------

## 4️⃣ 和相关命令对比

| 命令          | 作用             |
| ------------- | ---------------- |
| grep          | 普通文本搜索     |
| zgrep         | 压缩文件搜索     |
| zcat          | 查看压缩文件内容 |
| zless / zmore | 分页查看压缩文件 |

------

✅ **总结**

`zgrep` = `grep` + `gzip`，**直接在 `.gz` 文件里搜索，非常适合日志分析**。

