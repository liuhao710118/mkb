# 📝 `uniq` 是什么？

`uniq` 用于对文本 **相邻重复的行** 进行处理：

- 过滤掉重复行（去重）
- 显示重复行
- 统计重复次数
- 只显示唯一行

⚠️ **关键点：uniq 只能去除“相邻的”重复行**
 这意味着如果文件未排序，你可能得不到预期结果。

因此 `uniq` 常与 `sort` 一起使用：

```bash
sort file.txt | uniq
```

------

# 📌 一、基本用法

### ✔ 1. 去除相邻重复行

```bash
uniq file.txt
```

例：

输入：

```
a
a
b
a
```

输出：

```
a
b
a   ← 这一行仍然存在，因为它并不与前一行相同
```

------

# 📌 二、常用选项（最重要的几个）

| 选项      | 功能                                  |
| --------- | ------------------------------------- |
| `-d`      | 只输出重复行                          |
| `-u`      | 只输出不重复的行                      |
| `-c`      | 统计每行重复次数                      |
| `-i`      | 忽略大小写                            |
| `-f N`    | 跳过前 N 个字段比较                   |
| `-s N`    | 跳过前 N 个字符比较                   |
| `--group` | 分组显示唯一/重复行（新版 coreutils） |

------

# 📌 三、常用示例

------

## 1️⃣ **标准去重（必须排序）**

```bash
sort file.txt | uniq
```

------

## 2️⃣ **统计每行出现次数**

```bash
sort file.txt | uniq -c
```

示例结果：

```
  2 apple
  5 banana
  1 orange
```

------

## 3️⃣ **只显示重复行**

```bash
sort file.txt | uniq -d
```

------

## 4️⃣ **只显示不重复的行**

（唯一出现一次的行）

```bash
sort file.txt | uniq -u
```

------

## 5️⃣ **忽略大小写进行去重**

```bash
sort -f file.txt | uniq -i
```

------

## 6️⃣ **跳过前 N 个字段比较（-f）**

文本：

```
a 10
a 20
b 10
b 10
```

按第 2 列去重：

```bash
uniq -f 1 file.txt
```

输出：

```
a 10
a 20
b 10
```

------

## 7️⃣ **跳过前 N 个字符比较（-s）**

```bash
uniq -s 2 file.txt
```

------

## 8️⃣ **判断哪些行是独一无二并分组（--group）**

输入：

```
a
a
b
c
c
sort file.txt | uniq --group
```

输出：

```
repeat:
a
a

unique:
b

repeat:
c
c
```

------

# 📌 四、uniq 常见误区（非常重要）

### ❌ 误区：uniq 可以直接对文件去重

✔ 实际上 **只能处理相邻重复行**

错误例子：

```
a
b
a
uniq file.txt
```

输出还是：

```
a
b
a
```

必须先排序：

```bash
sort file.txt | uniq
```

------

# 📌 五、uniq 与 sort 的组合

| 目的                 | 推荐命令   |
| -------------------- | ---------- |
| 去重（不关心顺序）   | `sort file |
| 统计每行次数（升序） | `sort file |
| 统计并按次数排序     | `sort file |
| 找最大重复行         | `sort file |
| 仅输出重复行         | `sort file |
| 仅输出唯一行         | `sort file |

------

# 📌 六、实战示例

## ✔ 统计 Nginx 访问最多的 IP

```bash
cut -d ' ' -f 1 access.log | sort | uniq -c | sort -nr | head
```

------

## ✔ 统计某文件有多少行是重复的

```bash
sort file | uniq -d | wc -l
```

------

## ✔ 查看某字段是否有重复（如 CSV 第 3 列）

```bash
cut -d ',' -f 3 data.csv | sort | uniq -d
```

------

# 🎯 总结

`uniq` 非常简单但强大，核心：

- 只处理 **相邻** 行
- 常与 `sort` 搭配
- 常用选项：`-c`、`-d`、`-u`

常见用途：

- 统计重复
- 过滤重复
- 查找唯一值
- 大文件日志分析

