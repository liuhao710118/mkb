`sed`（**Stream Editor**）是 Linux/Unix 中非常常用的**文本流编辑工具**，主要用于对文本进行**查找、替换、删除、插入、过滤**等操作，常用于日志处理、配置文件修改、脚本自动化等场景。

它通常和 `grep`、`awk` 一起被称为 **Linux 文本处理三大神器**。

------

# 一、sed 基本语法

```bash
sed [选项] '操作命令' 文件
```

例如：

```bash
sed 's/old/new/' file.txt
```

含义：

- `s` ：替换（substitute）
- `old` ：旧内容
- `new` ：新内容

------

# 二、最常用 sed 命令

## 1 替换字符串

```bash
sed 's/old/new/' file.txt
```

只替换 **每一行的第一个匹配**

例子：

```bash
echo "hello world hello" | sed 's/hello/hi/'
```

结果

```
hi world hello
```

------

## 2 全局替换

```bash
sed 's/old/new/g' file.txt
```

`g` = global（全局）

例子

```bash
echo "hello hello hello" | sed 's/hello/hi/g'
```

输出

```
hi hi hi
```

------

## 3 只替换指定行

### 替换第2行

```bash
sed '2s/old/new/' file.txt
```

### 替换2-5行

```bash
sed '2,5s/old/new/g' file.txt
```

------

## 4 删除行

### 删除第3行

```bash
sed '3d' file.txt
```

### 删除2-5行

```bash
sed '2,5d' file.txt
```

### 删除包含某字符串的行

```bash
sed '/error/d' file.txt
```

------

## 5 打印指定行

默认 sed 会输出全部内容

只打印第3行：

```bash
sed -n '3p' file.txt
```

打印包含 error 的行

```bash
sed -n '/error/p' file.txt
```

------

## 6 插入行

### 在第3行前插入

```bash
sed '3i new line' file.txt
```

### 在第3行后插入

```bash
sed '3a new line' file.txt
```

------

## 7 修改文件（直接写入）

默认 sed **不会修改原文件**

需要 `-i`

```bash
sed -i 's/old/new/g' file.txt
```

例如：

```bash
sed -i 's/localhost/127.0.0.1/g' config.txt
```

------

# 三、常见实战案例

## 1 删除空行

```bash
sed '/^$/d' file.txt
```

解释

```
^  行开头
$  行结尾
^$ 空行
```

------

## 2 删除注释

```bash
sed '/^#/d' file.txt
```

删除以 `#` 开头的行

------

## 3 替换 IP

```bash
sed 's/192.168.1.1/10.0.0.1/g' file.txt
```

------

## 4 只保留第1-10行

```bash
sed -n '1,10p' file.txt
```

------

## 5 提取日志中的错误

```bash
sed -n '/ERROR/p' app.log
```

------

# 四、sed 常用选项

| 参数 | 作用           |
| ---- | -------------- |
| -n   | 不自动打印     |
| -i   | 直接修改文件   |
| -e   | 多个命令       |
| -f   | 从脚本读取命令 |

------

## 多命令示例

```bash
sed -e 's/error/ERROR/g' -e 's/warn/WARN/g' log.txt
```

------

# 五、sed 正则表达式

> 参考链接：[sed命令正则基本用法.md](./concept/sed命令正则基本用法.md)

# 六、程序员最常用的 sed 命令

### 替换配置

```bash
sed -i 's/port=8080/port=9090/g' config.ini
```

### 批量修改

```bash
sed -i 's/http/https/g' *.conf
```

### 处理日志

```bash
sed -n '/ERROR/p' app.log
```

------

# 七、sed + 管道

sed 很多时候 **配合管道**

```bash
cat log.txt | sed 's/error/ERROR/g'
```

或者

```bash
grep ERROR log.txt | sed 's/ERROR/[ERROR]/'
```

------

# 八、sed 和 awk 的区别

| 工具 | 特点   |
| ---- | ------ |
| sed  | 行编辑 |
| awk  | 列处理 |
| grep | 查找   |

------

