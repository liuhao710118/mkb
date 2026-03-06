`zless` 是 Linux/Unix 系统中用来 **查看压缩文件内容（通常是 `.gz`）的分页命令**。它类似 `less`，但可以 **直接读取 gzip 压缩文件而不需要先解压**。

## 一、基本用法

```bash
zless 文件名.gz
```

例如：

```bash
zless access.log.gz
```

作用：

- 直接查看 `access.log.gz`
- 不需要 `gunzip`
- 支持 **分页浏览**

------

## 二、常见操作键（和 `less` 一样）

| 按键    | 作用       |
| ------- | ---------- |
| Space   | 向下翻一页 |
| Enter   | 向下翻一行 |
| b       | 向上翻一页 |
| /关键字 | 搜索       |
| n       | 下一个匹配 |
| q       | 退出       |

示例：

```
/ERROR
```

搜索日志中的 `ERROR`

------

## 三、常见使用场景

### 1 查看压缩日志

```bash
zless /var/log/nginx/access.log.1.gz
```

------

### 2 搜索日志

```
/500
```

查找 HTTP 500。

------

### 3 和 `grep` 配合

如果想过滤：

```bash
zcat access.log.gz | grep 500
```

或者

```bash
zgrep 500 access.log.gz
```

------

## 四、相关命令对比

| 命令    | 作用                   |
| ------- | ---------------------- |
| `zcat`  | 直接输出 gzip 内容     |
| `zless` | 分页查看 gzip          |
| `zgrep` | 在 gzip 中搜索         |
| `zmore` | 类似 zless，但功能较少 |

示例：

```bash
zgrep ERROR access.log.gz
```

------

✅ **总结**

`zless` = **less + gzip**

特点：

- 不需要解压
- 支持分页
- 适合查看 `.gz` 日志

------

💡 **程序员常用日志技巧：**

如果你经常看日志，这三个命令非常有用：

```bash
zless access.log.gz
zgrep ERROR access.log.gz
zcat access.log.gz | less
```