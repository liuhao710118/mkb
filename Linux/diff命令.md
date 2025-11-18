# **Linux `diff` 命令教程文档**



> 常用的是git风格的比较



diff -u 文件1 文件2

输出

```
diff --git a/file.txt b/file.txt
index 7898192..6fc4957 100644
--- a/file.txt
+++ b/file.txt
@@ -1,5 +1,6 @@
 Hello World
-This line was removed
+This line was added
+This is another new line
 Some existing content
 Another existing line
```



**头部信息：**

- `--- a/file.txt`：原始文件（a 版本）
- `+++ b/file.txt`：修改后的文件（b 版本）

**变更块标识：**

- `@@ -1,5 +1,6 @@`：表示变更的位置
  - `-1,5`：原始文件从第1行开始，共5行
  - `+1,6`：新文件从第1行开始，共6行

**变更内容：**

- `-`开头的行：**被删除**的内容（红色显示）
- `+`开头的行：**被添加**的内容（绿色显示）
- **没有符号**的行：未变更的上下文内容



`diff`（difference）用于 **比较文件或目录之间的差异**，并以标准格式输出变更内容。
 常用于：

- 配置文件对比
- 代码变更分析
- 备份文件一致性校验
- 自动化补丁生成

------

# **一、基本语法**

```
diff [选项] <文件1> <文件2>
```

目录比较：

```
diff [选项] <目录1> <目录2>
```

------

# **二、常用选项说明**

| 选项             | 说明                                           |
| ---------------- | ---------------------------------------------- |
| **-u**           | 以 unified（统一格式）输出，最常用（git 风格） |
| **-c**           | context 格式输出                               |
| **-r**           | 递归比较目录                                   |
| **-q**           | 只显示文件是否不同，不显示具体差异             |
| **-i**           | 忽略大小写                                     |
| **-w**           | 忽略所有空白字符（空格、换行）                 |
| **--color=auto** | 彩色显示差异（更易读）                         |

------

# **三、基础用法示例**

------

## **1. 比较两个文件（最常用）**

```
diff file1.txt file2.txt
```

输出示例：

```
3c3
< apple
---
> banana
```

表示：file1 第 3 行与 file2 第 3 行不一致。

------

## **2. 统一格式（开发者最常用，类似 git diff）**

```
diff -u file1 file2
```

输出：

```
--- file1
+++ file2
@@ -1,3 +1,3 @@
 apple
-orange
+banana
 grape
```

特点：清晰标明前后差异。

------

## **3. 忽略大小写比较**

```
diff -i a.txt b.txt
```

------

## **4. 忽略所有空白字符**

```
diff -w a.txt b.txt
```

用于忽略程序格式不一致例如缩进、空格等。

------

# **四、目录比较**

------

## **1. 递归比较两个目录**

```
diff -r dir1 dir2
```

输出示例：

```
Only in dir2: config.json
Files dir1/app.py and dir2/app.py differ
```

------

## **2. 仅显示是否不同（不显示具体差异）**

```
diff -qr dir1 dir2
```

输出：

```
Files dir1/a.conf and dir2/a.conf differ
```

适用于目录同步检测。

------

# **五、生成补丁（patch）文件**

`diff` 可生成补丁文件，供 `patch` 命令使用。

------

## **1. 生成补丁（统一格式）**

```
diff -u old.conf new.conf > update.patch
```

------

## **2. 使用补丁文件自动更新**

```
patch < update.patch
```

适用于自动配置下发、版本升级。

------

# **六、输出解释说明（重要）**

`diff` 输出中符号含义：

| 符号 | 含义           |
| ---- | -------------- |
| `<`  | 来自第一个文件 |
| `>`  | 来自第二个文件 |
| `a`  | 添加（add）    |
| `d`  | 删除（delete） |
| `c`  | 修改（change） |

示例：

```
4a5
```

表示：在 file1 的第 4 行之后添加 file2 第 5 行。

------

# **七、结合管道使用（高级技巧）**

------

## **1. 对比命令输出差异**

```
diff <(ls dir1) <(ls dir2)
```

使用进程替代符，比较目录内容，非常方便。

------

## **2. 对比两个程序输出是否一致**

```
diff <(curl http://api1) <(curl http://api2)
```

------

# **八、实用场景示例**

------

## **1. 配置文件变更对比**

```
diff -u /etc/nginx/nginx.conf.bak /etc/nginx/nginx.conf
```

------

## **2. 检查部署目录是否一致**

```
diff -qr /opt/app1 /opt/app2
```

------

## **3. 生成版本升级补丁**

```
diff -urN old-version/ new-version/ > update.patch
```

------

# **九、总结**

`diff` 是 Linux 文本比较的核心工具，可用于文件与目录的差异对比、补丁生成等。

最常用：

```
diff -u file1 file2      # 显示详细差异
diff -r dir1 dir2        # 比较目录
diff -qr dir1 dir2       # 只显示是否不同
diff -u old new > patch  # 生成补丁
```

