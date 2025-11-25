# 📌 **iotop 命令是什么？**

`iotop` 用于实时监控系统中 **进程/线程的磁盘 I/O 使用情况**，类似 `top` 查看 CPU 和内存，但它专门看：

- **读 I/O**
- **写 I/O**
- **I/O 利用率**
- 哪些进程正在占用磁盘

适用于定位：

- 高磁盘占用
- I/O 阻塞
- 性能瓶颈

> ⚠ 注意：iotop 需要内核支持 I/O 统计（CONFIG_TASKSTATS、CONFIG_TASK_DELAY_ACCT 等），并一般需要 root 权限。

------

# 📝 基本用法

```bash
sudo iotop
```

会显示所有进程的实时 I/O 情况。

------

# ⭐ 常用选项

| 参数      | 说明                                      |
| --------- | ----------------------------------------- |
| `-o`      | **仅显示有实际 I/O 活动的进程**（最常用） |
| `-b`      | 批处理模式（适合脚本、日志）              |
| `-n NUM`  | 刷新 NUM 次后退出                         |
| `-d SEC`  | 每 SEC 秒刷新一次                         |
| `-p PID`  | 只监控指定 PID                            |
| `-u USER` | 只监控某用户                              |
| `-k`      | 按 kB/s 显示（默认就使用 kB/s）           |
| `-q`      | 和 `-b` 一起使用去掉标题栏                |

------

# 📊 输出字段含义详解

默认界面示例：

```
Total DISK READ : 5.54 M/s | Total DISK WRITE : 2.34 M/s
TID  PRIO  USER     DISK READ  DISK WRITE  SWAPIN    IO    COMMAND
1234 be/4  root        0.00 K/s   500.00 K/s  0.00%  10.00%  dd if=/dev/zero of=/tmp/test
```

字段解释：

### 第一行

- **Total DISK READ**：系统总读速率
- **Total DISK WRITE**：系统总写速率

### 表格字段

| 字段       | 含义                                             |
| ---------- | ------------------------------------------------ |
| TID        | 线程 ID（iotop 默认在“线程模式”）                |
| PRIO       | I/O 优先级（be=best-effort，rt=real-time）       |
| USER       | 执行该进程的用户                                 |
| DISK READ  | 磁盘读速率                                       |
| DISK WRITE | 磁盘写速率                                       |
| SWAPIN%    | 进程因 swap 引起的 I/O 比例                      |
| IO%        | 进程等待 I/O 造成阻塞的 CPU 时间占比（I/O wait） |
| COMMAND    | 进程命令                                         |

💡 **IO% 高说明进程因磁盘 I/O 而阻塞。**

------

# 🧪 使用示例

### ✔ 只查看正在进行 I/O 的进程（最常用）

```bash
sudo iotop -o
```

------

### ✔ 每 2 秒刷新，输出 10 次

```bash
sudo iotop -d 2 -n 10
```

------

### ✔ 显示指定进程 PID 的 I/O

```bash
sudo iotop -p 1234
```

------

### ✔ 批处理模式（用于日志）

```bash
sudo iotop -b -n 5 > io.log
```

------

### ✔ 配合 grep 查看特定命令 I/O

```bash
sudo iotop -b | grep mysql
```

------

### ✔ 查看某个用户的 IO（如 Apache）

```bash
sudo iotop -u www-data
```

------

# 🛠️ 常见诊断场景

### 1️⃣ **系统变慢，怀疑磁盘 I/O 高**

```bash
sudo iotop -o
```

➜ 找出正在疯狂读写的进程

### 2️⃣ **排查数据库磁盘瓶颈**

```bash
sudo iotop -p $(pidof mysqld)
```

### 3️⃣ **找出谁导致高 IO wait（iowait 上升）**

看 IO% 字段，值高代表进程被阻塞在 I/O 上。

### 4️⃣ **长期记录 I/O 日志**

```bash
sudo iotop -b -d 5 -n 1000 >> iolog.txt
```

------

# 🎯 总结

`iotop` 是 Linux 下查看 **实时磁盘 I/O 的首选工具**，重点关注：

- **DISK READ / WRITE**：谁在高读写
- **IO%**：进程是否被 I/O 阻塞
- 常用参数：`-o`、`-b`、`-d`、`-n`