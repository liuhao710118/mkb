`pgrep` 是 Linux 中 **按条件查找进程 PID 的工具**，它属于 `procps` 工具集。
常用于 **脚本、运维排障、自动化管理进程**。

简单理解：

```
pgrep = ps + grep + 提取PID
```

它可以直接找到进程 **PID**，比 `ps -ef | grep` 更简洁。

------

# 一、基本语法

### **一、命令基本语法**

```markdown
pgrep [选项] <进程名模式>
```

- 

  **进程名模式**：用于匹配进程名的字符串，支持正则表达式。

# **二、常用选项详解**

| 选项   | 全称          | 作用与示例                                                   |
| ------ | ------------- | ------------------------------------------------------------ |
| **-l** | `--list-name` | **同时列出 PID 和进程名**。 示例：`pgrep -l nginx`会输出 `1234 nginx`。 |
| **-f** | `--full`      | **匹配完整的命令行**，而不仅仅是进程名。 示例：`pgrep -f "python app.py"`可以找到运行该脚本的进程。 |
| **-u** | `--euid`      | **匹配指定有效用户（EUID）的进程**。 示例：`pgrep -u root`查找所有 root 用户运行的进程。 |
| **-U** | `--uid`       | **匹配指定真实用户（UID）的进程**。                          |
| **-x** | `--exact`     | **进行精确匹配**，进程名必须与模式完全一致。 示例：`pgrep -x bash`只匹配名为 “bash” 的进程，不会匹配 “bashrc”。 |
| **-n** | `--newest`    | **只显示最新（最近启动）的匹配进程**。                       |
| **-o** | `--oldest`    | **只显示最旧（最早启动）的匹配进程**。                       |
| **-c** | `--count`     | **仅输出匹配到的进程数量**，而不显示 PID。                   |
| **-v** | `--inverse`   | **反向匹配**，显示**不**符合模式的进程。                     |
| **-d** | `--delimiter` | **指定输出分隔符**，默认为换行符。 示例：`pgrep -d , sshd`输出类似 `1122,2233`。 |

------

### 最常用示例

## 1 查询某个进程

```bash
pgrep nginx
```

查看 nginx 的 PID。

等价于：

```bash
ps -ef | grep nginx
```

但 `pgrep` 更干净。

------

## 2 显示 PID + 进程名

```bash
pgrep -l nginx
```

输出：

```
1234 nginx
1235 nginx
```

参数：

```
-l  显示进程名
```

------

## 3 显示完整命令

```bash
pgrep -a java
```

输出：

```
2345 java -jar app.jar
```

参数：

```
-a 显示完整命令行
```

这个在 **查 Java 程序** 时非常常用。

------

# 三、按完整命令行匹配

默认 `pgrep` 只匹配 **进程名**。

如果要匹配 **完整命令行**：

```
-f
```

例如：

```bash
pgrep -f app.jar
```

输出：

```
2345
```

查看详情：

```bash
pgrep -af app.jar
```

输出：

```
2345 java -jar app.jar
```

------

# 四、按用户查询

查询某用户的进程：

```bash
pgrep -u username
```

例如：

```bash
pgrep -u tom
```

输出：

```
1234
2234
5567
```

------

显示进程名：

```bash
pgrep -lu tom
```

输出：

```
1234 bash
2234 java
5567 python
```

------

# 五、按父进程查询

```bash
pgrep -P PID
```

例如：

```bash
pgrep -P 1000
```

查看某个进程的 **子进程**。

------

# 六、匹配多个进程

`pgrep` 支持 **正则表达式**。

例如：

```bash
pgrep 'nginx|apache'
```

输出：

```
1234
5678
```

------

# 七、精确匹配

```bash
pgrep -x nginx
```

说明：

```
-x 完全匹配
```

只匹配：

```
nginx
```

不匹配：

```
nginx-worker
```

------

# 八、查找最新或最老进程

### 查找最新进程

```bash
pgrep -n nginx
```

`-n` = newest

------

### 查找最老进程

```bash
pgrep -o nginx
```

`-o` = oldest

------

# 九、限制返回数量

例如：

```bash
pgrep -n java
```

只返回最新一个 Java 进程。

------

# 十、结合 kill 使用

`pgrep` 常和 `kill` 一起用。

例如：

### 杀死 nginx

```bash
kill $(pgrep nginx)
```

------

### 杀死某个 jar

```bash
kill $(pgrep -f app.jar)
```

------

# 十一、Shell脚本中常用

### 判断进程是否存在

```bash
if pgrep nginx > /dev/null
then
    echo "nginx running"
else
    echo "nginx stopped"
fi
```

------

### 自动启动服务

```bash
pgrep nginx > /dev/null || nginx
```

含义：

```
如果 nginx 没运行，就启动
```

------

# 十二、运维最常用组合

### 查看 Java 进程

```bash
pgrep -af java
```

------

### 查看某个 jar

```bash
pgrep -af app.jar
```

------

### 查某端口程序

```bash
pgrep -af 8080
```

------

# 十三、pgrep vs ps

| 命令      | 作用         |
| --------- | ------------ |
| ps        | 查看所有进程 |
| ps + grep | 过滤         |
| pgrep     | 直接匹配进程 |

示例：

传统方式：

```bash
ps -ef | grep nginx
```

推荐方式：

```bash
pgrep -af nginx
```

------

# 十四、pgrep + pkill 组合

这两个命令通常一起出现：

```
pgrep = 查进程
pkill = 杀进程
```

例如：

查看：

```bash
pgrep -af app.jar
```

杀掉：

```bash
pkill -f app.jar
```

------

# 十五、生产环境建议

**永远先 pgrep 再 pkill**

例如：

```
pgrep -af app.jar
pkill -f app.jar
```

避免误杀。

------

如果你做 **Linux运维 / Java部署 / Kubernetes / 中间件**，我可以再给你整理一份：

**《Linux进程管理最常用20个命令（非常实战）》**

包括：

```
ps
top
htop
pgrep
pkill
kill
killall
lsof
netstat
ss
```

基本是 **运维 / 面试 / 生产环境最常用的一套**。