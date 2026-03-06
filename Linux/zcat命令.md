`zcat` 是 Linux/Unix 系统中用来 **查看压缩文件内容的命令**，和 `cat` 很像，但它能 **直接读取 gzip 压缩文件（`.gz`）** 并输出到标准输出（终端或管道），无需先解压。📄

------

## 1️⃣ 基本语法

```bash
zcat 文件名.gz
```

示例：

```bash
zcat access.log.gz
```

作用：

- 输出 `access.log.gz` 的内容到终端
- 可以配合管道命令处理，例如 `grep`、`less`

------

## 2️⃣ 常见用法示例

### 1. 分页查看

```bash
zcat access.log.gz | less
```

- 可以分页浏览大文件

------

### 2. 搜索内容

```bash
zcat access.log.gz | grep "ERROR"
```

- 查找所有包含 `ERROR` 的行

------

### 3. 与管道统计

```bash
zcat access.log.gz | wc -l
```

- 统计压缩日志总行数

------

### 4. 查看多个压缩文件

```bash
zcat access1.log.gz access2.log.gz | grep "500"
```

- 支持一次输出多个 `.gz` 文件的内容

------

## 3️⃣ 与相关命令对比

| 命令                      | 功能                                 |
| ------------------------- | ------------------------------------ |
| `cat file`                | 查看普通文本文件                     |
| `zcat file.gz`            | 查看单个 gzip 压缩文件               |
| `zless file.gz`           | 分页查看 gzip 文件                   |
| `zgrep "PATTERN" file.gz` | 在 gzip 文件中搜索内容               |
| `tar -O -xzf file.tar.gz` | 查看 `.tar.gz` 内容流（可结合 grep） |

------

💡 **总结**

- `zcat` = `cat + gzip`
- 输出到终端或管道
- 不解压到磁盘

