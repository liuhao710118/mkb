# # **Linux `awk` 命令教程（超详细）**

## 一、`awk` 是什么？

`awk` 是 Linux 中最强大的文本处理工具之一，用于：

- 按列处理文本（如按空格、逗号等分隔）
- 条件筛选（类似 SQL where）
- 格式化输出
- 文本统计（求和、平均、最大值）
- 批处理日志文件

它像是 shell 中的 **超强版 Excel + SQL + 脚本语言**。

------

# **二、基本语法**

```
awk 'pattern { action }' file
```

例子：

```
awk '{print $1}' file.txt
```

------

# **三、默认行为**

- 默认分隔符是空白（空格、Tab）
- `$1` 表示第一列，`$2` 第二列，`$0` 为整行
- `NR` 行号
- `NF` 列数

------

# **四、常用参数**

| 参数 | 说明                |
| ---- | ------------------- |
| `-F` | 指定分隔符          |
| `-v` | 传入变量            |
| `-f` | 从文件加载 awk 脚本 |

------

# **五、最常用操作示例**

## 1. 打印第一列

```
awk '{print $1}' file.txt
```

## 2. 打印第 1 列 和 第 3 列

```
awk '{print $1, $3}' file.txt
```

## 3. 指定分隔符（如：冒号）

```
awk -F: '{print $1}' /etc/passwd
```

## 4. 打印行号 + 内容

```
awk '{print NR, $0}' file.txt
```

## 5. 打印列数

```
awk '{print NF}' file.txt
```

------

# **六、按条件过滤（就像 SQL 的 WHERE）**

## 1. 打印第二列大于 100 的行

```
awk '$2 > 100' file.txt
```

## 2. 第二列等于 "root"

```
awk '$2 == "root"' file.txt
```

## 3. 只显示包含某字符串的行

```
awk '/error/' logfile
```

## 4. 指定范围（行区间）

```
awk 'NR>=10 && NR<=20' file.txt
```

------

# **七、统计（sum、avg、max）**

## 1. 求第二列之和

```
awk '{sum += $2} END {print sum}' file.txt
```

## 2. 求平均值

```
awk '{sum += $2} END {print sum/NR}' file.txt
```

## 3. 求最大值

```
awk 'NR==1 || $2 > max {max=$2} END {print max}' file.txt
```

------

# **八、格式化输出（类似 printf）**

```
awk '{printf "%-10s %-10s\n", $1, $2}' file.txt
```

------

# **九、修改列内容**

## 1. 第 2 列乘以 100

```
awk '{$2 = $2 * 100; print}' file.txt
```

## 2. 拼接字符串

```
awk '{print $1 "_suffix"}' file.txt
```

------

# **十、自定义分隔符输出 OFS**

```
awk 'BEGIN{OFS=","} {print $1,$2,$3}' file.txt
```

------

# **十一、使用变量**

## 1. 向 awk 传递变量

```
awk -v threshold=100 '$2 > threshold' file.txt
```

------

# **十二、数组（统计频率）**

例如统计第 1 列出现次数：

```
awk '{count[$1]++} END {for (k in count) print k, count[k]}' file.txt
```

------

# **十三、BEGIN 与 END**

```
awk 'BEGIN {print "start"} {print $1} END {print "done"}' file
```

流程：

1. BEGIN 最先执行
2. 对每行执行 `{}`
3. END 最后执行

------

# **十四、典型实战案例**

## **1. 查看磁盘使用情况最大项**

```
df -h | awk 'NR>1 {print $5, $6}' | sort -nr | head -1
```

## **2. 统计 Nginx 中每个 IP 的访问次数**

```
awk '{count[$1]++} END{for (ip in count) print ip, count[ip]}' access.log
```

## **3. 日志中统计 error 的数量**

```
awk '/error/ {count++} END{print count}' logfile
```

## **4. 按 cpu 使用率排序**

```
ps aux | awk '{print $3, $11}' | sort -nr
```

------

# **十五、Cheat Sheet（速查表）**

| 功能       | 命令                                            |
| ---------- | ----------------------------------------------- |
| 打印列     | `awk '{print $1,$2}'`                           |
| 分隔符     | `awk -F"," '{print $1}'`                        |
| 条件过滤   | `awk '$3>100'`                                  |
| 匹配字符串 | `awk '/nginx/'`                                 |
| 求和       | `awk '{sum+=$2} END{print sum}'`                |
| 求平均值   | `awk '{sum+=$2} END{print sum/NR}'`             |
| 统计频率   | `awk '{a[$1]++} END{for(i in a) print i,a[i]}'` |
| 格式化输出 | `awk '{printf "%s\t%s\n",$1,$2}'`               |

