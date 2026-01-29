`ss` 是 Linux 下用于**查看网络连接状态**的核心命令，比 `netstat` 更快、更推荐（基于内核 `sock_diag` 接口）。

------

## 一、ss 是什么

- **全称**：Socket Statistics
- **作用**：查看 TCP / UDP / UNIX Socket / RAW Socket 等状态
- **优点**：速度快、信息全、内核级接口
- **位置**：`iproute2` 工具包

------

## 二、常用参数速查表

| 参数 | 含义                         |
| ---- | ---------------------------- |
| `-l` | 只显示监听状态 (LISTEN)      |
| `-n` | 不解析域名，直接显示 IP      |
| `-t` | TCP                          |
| `-u` | UDP                          |
| `-x` | UNIX Socket                  |
| `-p` | 显示进程信息                 |
| `-a` | 所有状态                     |
| `-s` | 统计汇总                     |
| `-o` | 显示定时器                   |
| `-e` | 显示扩展信息                 |
| `-i` | TCP 内部信息（窗口、拥塞等） |
| `-r` | 显示路由信息                 |

------

## 三、最常用命令组合 ⭐

### 1️⃣ 查看所有 TCP 连接

```bash
ss -t
```

### 2️⃣ 查看所有 TCP 监听端口（等价 netstat -lnt）

```bash
ss -lnt
```

### 3️⃣ 查看监听端口 + 进程

```bash
ss -lntp
```

示例输出：

```
LISTEN 0 128 0.0.0.0:22  0.0.0.0:* users:(("sshd",pid=1234,fd=3))
```

------

### 4️⃣ 查看所有连接（TCP + UDP）

```bash
ss -tunap
```

------

### 5️⃣ 查看指定端口

```bash
ss -lnt sport = :80
ss -ant dport = :3306
```

------

### 6️⃣ 查看某 IP 的连接

```bash
ss -ant dst 192.168.1.10
```

------

## 四、TCP 状态排查（非常重要）🔥

### 查看各 TCP 状态数量

```bash
ss -ant | awk '{++S[$1]} END {for (i in S) print i, S[i]}'
```

常见状态含义：

| 状态           | 说明               |
| -------------- | ------------------ |
| `LISTEN`       | 监听               |
| `ESTAB`        | 已建立             |
| `TIME-WAIT`    | 等待关闭           |
| `CLOSE-WAIT`   | 对端已关，本端未关 |
| `SYN-SENT`     | 正在建立           |
| `SYN-RECV`     | 半连接             |
| `FIN-WAIT-1/2` | 主动关闭           |

👉 **大量 TIME-WAIT**：端口复用 / 短连接问题
👉 **大量 CLOSE-WAIT**：程序未正确 close socket（代码问题）

------

## 五、ss 高级用法 🚀

### 1️⃣ 查看 TCP 内核参数（窗口、RTT、拥塞）

```bash
ss -ti
```

------

### 2️⃣ 显示 socket 缓冲区

```bash
ss -tm
```

------

### 3️⃣ 查看 UNIX Socket

```bash
ss -xlp
```

------

### 4️⃣ 统计汇总（快速看系统网络状态）

```bash
ss -s
```

示例：

```
TCP:   123 (estab 80, closed 30, orphaned 2, timewait 10)
```

------

## 六、ss vs netstat 对照表

| netstat         | ss         |
| --------------- | ---------- |
| `netstat -lntp` | `ss -lntp` |
| `netstat -ant`  | `ss -ant`  |
| `netstat -s`    | `ss -s`    |
| `netstat -x`    | `ss -x`    |

📌 **结论：以后优先用 ss**

------

## 七、运维排障常用一行命令合集 ⭐⭐⭐

```bash
# 看谁占了端口
ss -lntp | grep :8080

# 看某服务连接数
ss -ant sport = :80 | wc -l

# 看半连接
ss -ant state syn-recv

# 看 TIME-WAIT
ss -ant state time-wait
```

