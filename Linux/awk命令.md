

------

# 🧾 awk 命令使用文档（GNU awk 实用指南）

------

## 一、命令简介

**awk** 是一种强大的文本处理工具，名字来源于三位创作者的姓氏首字母：**Aho、Weinberger、Kernighan**。
 它擅长按“行 + 列”的模式读取文本数据，并对每行按规则执行操作。
 `awk` 可用于：

- 文本格式化输出
- 日志分析、数据筛选
- 表格字段统计、求和、平均值
- 文件内容过滤与转换

在 Linux 中常用的实现是 **GNU awk（gawk）**。

------

## 二、基本语法

```bash
awk [选项] 'pattern { action }' 文件名
```

或：

```bash
awk [选项] -f 脚本文件 [文件...]
```

------

## 三、常用选项（来自 `awk --help`）

| 选项                      | 说明                                   |
| ------------------------- | -------------------------------------- |
| `-F fs`                   | 设置输入字段分隔符（默认空格或制表符） |
| `-v var=value`            | 在执行前设置变量值                     |
| `-f file`                 | 从指定文件读取 awk 程序                |
| `-e 'program-text'`       | 在命令行中直接指定程序文本             |
| `--posix`                 | 以 POSIX 模式运行，限制 GNU 扩展       |
| `--lint[=fatal]`          | 检查可移植性问题                       |
| `--dump-variables[=file]` | 输出所有变量（调试用）                 |
| `--profile[=file]`        | 输出性能分析数据                       |
| `--sandbox`               | 禁止系统命令调用（安全模式）           |
| `--version`               | 显示版本信息                           |
| `--help`                  | 显示帮助信息                           |

------

## 四、基本结构

一个 awk 程序由 3 个部分组成：

```awk
BEGIN { 初始化操作 }
模式 { 对匹配行执行的操作 }
END { 处理结束后执行的操作 }
```

三个部分都可选。

------

## 五、字段与内置变量

| 变量          | 含义                           |
| ------------- | ------------------------------ |
| `$0`          | 当前整行内容                   |
| `$1`、`$2`... | 各字段（第1列、第2列…）        |
| `NF`          | 当前行字段数                   |
| `$NF`         | 当前行的最后一个字段           |
| `NR`          | 当前读到的行号（所有文件累计） |
| `FNR`         | 当前文件内的行号               |
| `FS`          | 输入字段分隔符（默认空格）     |
| `OFS`         | 输出字段分隔符                 |
| `RS`          | 输入记录分隔符（默认换行符）   |
| `ORS`         | 输出记录分隔符                 |
| `FILENAME`    | 当前文件名                     |

------

## 六、常用操作示例

### 1️⃣ 打印字段

```bash
awk '{print $1, $3}' data.txt
```

打印每行的第1列和第3列。

### 2️⃣ 指定分隔符

```bash
awk -F ':' '{print $1}' /etc/passwd
```

以冒号作为分隔符，打印用户名。

### 3️⃣ 条件筛选

```bash
awk '$3 > 100' data.txt
```

打印第三列大于 100 的行。

### 4️⃣ 模式匹配（正则）

```bash
awk '/error/' logfile.txt
awk '!/INFO/' logfile.txt
```

打印包含或不包含指定关键字的行。

### 5️⃣ BEGIN / END 用法

```bash
awk 'BEGIN {print "开始处理"} {print $1} END {print "处理完成"}' file.txt
```

------

## 七、计算统计类示例

### 求和

```bash
awk '{sum += $2} END {print "总和:", sum}' data.txt
```

### 平均值

```bash
awk '{sum += $2; count++} END {print "平均值:", sum / count}' data.txt
```

### 条件统计

```bash
awk '$3=="OK" {ok++} END {print "成功行数:", ok}' log.txt
```

------

## 八、字符串与格式化

### 格式化输出

```bash
awk '{printf "%-10s %5d\n", $1, $2}' data.txt
```

### 转换大小写

```bash
awk '{print toupper($1), tolower($2)}' words.txt
```

### 字符串连接

```bash
awk '{print $1 "-" $2}' data.txt
```

------

## 九、控制结构

awk 语法支持类似 C 语言的控制语句：

```awk
if (条件) { ... } else { ... }
for (i=1; i<=NF; i++) { ... }
while (条件) { ... }
break; continue;
```

示例：

```bash
awk '{if ($3 > 80) print $1, "及格"; else print $1, "不及格"}' score.txt
```

------

## 🔟 文件与输入控制

### 同时处理多个文件

```bash
awk '{print FILENAME, NR, $0}' file1 file2
```

### 只打印文件名

```bash
awk 'BEGIN {FS=":"} {print FILENAME}' /etc/passwd /etc/group
```

------

## 🧠 十一、awk 的强大输出控制

| 控制变量 | 说明                        |
| -------- | --------------------------- |
| `OFS`    | 输出字段分隔符（默认空格）  |
| `ORS`    | 输出记录分隔符（默认换行）  |
| `OFMT`   | 输出数字格式（默认 "%.6g"） |

示例：

```bash
awk 'BEGIN {OFS=","} {print $1, $2}' data.txt
```

------

## 🧩 十二、数组与循环示例

awk 支持**关联数组（哈希表）**：

```bash
awk '{count[$1]++} END {for (key in count) print key, count[key]}' access.log
```

👉 统计日志中每个 IP 出现次数。

------

## ⚡ 十三、实用案例

### 1️⃣ 统计访问日志中各 IP 出现次数

```bash
awk '{ip[$1]++} END {for (i in ip) print i, ip[i]}' access.log | sort -k2 -nr | head
```

### 2️⃣ 处理 CSV 数据

```bash
awk -F',' '{sum+=$3} END {print "总金额:", sum}' data.csv
```

### 3️⃣ 过滤并导出新文件

```bash
awk '$3 > 100 {print $1,$2,$3}' data.txt > filtered.txt
```

------

## 🧰 十四、调试与性能分析

| 选项                        | 用途                         |
| --------------------------- | ---------------------------- |
| `--lint`                    | 检查潜在的移植性问题         |
| `--dump-variables=vars.txt` | 输出所有变量值（调试）       |
| `--profile=prof.txt`        | 生成性能分析报告             |
| `-t`                        | 调试输出执行的命令           |
| `--sandbox`                 | 禁用外部命令调用（安全模式） |

------

## ⚙️ 十五、awk 执行原理总结

1️⃣ awk 逐行读取输入文件
 2️⃣ 按 `FS` 分隔成字段
 3️⃣ 判断该行是否匹配指定“模式”
 4️⃣ 如果匹配，则执行对应“动作”
 5️⃣ 所有行处理完后，执行 `END` 模块

------

## ✅ 十六、常见命令速查表

| 任务               | 命令                                  |
| ------------------ | ------------------------------------- |
| 打印第1列          | `awk '{print $1}' file`               |
| 打印第1、3列       | `awk '{print $1,$3}' file`            |
| 以逗号分隔         | `awk -F',' '{print $1}' file.csv`     |
| 条件筛选           | `awk '$2>100' file`                   |
| 统计行数           | `awk 'END{print NR}' file`            |
| 计算总和           | `awk '{sum+=$2} END{print sum}' file` |
| 删除空行           | `awk NF file`                         |
| 输出包含关键字的行 | `awk '/error/' file`                  |
| 打印文件最后一列   | `awk '{print $NF}' file`              |

------

## 📚 十七、结语

`awk` 是文本处理的瑞士军刀，灵活且强大。
 熟练掌握它后，你可以在不写 Python/Perl 的情况下完成大多数文本分析任务。

------

是否希望我帮你把这份文档导出成一份
 ✅ **Markdown + PDF 格式**（带格式、可打印、可离线保存）？
 可以直接保存为《awk 使用指南（GNU awk 中文版）》方便查阅。