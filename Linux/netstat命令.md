

# 📘 Linux `netstat` 命令使用文档

## 一、命令简介

`netstat`（Network Statistics）是一个常用的网络状态监控工具，用于显示：

- 网络连接（TCP/UDP 等）
- 路由表
- 网络接口信息
- 网络协议统计信息
- 网络伪装（NAT）连接等

在系统排查网络连通性、端口占用、连接状态、路由等问题时非常有用。

> ⚠️ 注意：在某些现代 Linux 发行版中，`netstat` 已被 `ss` 命令取代（位于 `iproute2` 包中），但 `netstat` 仍广泛用于传统系统与脚本中。

------

## 二、命令语法

```bash
netstat [OPTIONS] [delay]
```

或：

```bash
netstat [<Socket> ...]
```

或：

```bash
netstat [-vWeenNcCF] [<Af>] -r
```

------

## 三、常用功能分类

| 功能              | 对应选项             | 说明                                    |
| ----------------- | -------------------- | --------------------------------------- |
| 查看路由表        | `-r`, `--route`      | 显示系统的路由表（类似 `route -n`）     |
| 查看接口信息      | `-i`, `--interfaces` | 显示所有网络接口统计信息                |
| 查看指定接口      | `-I <Iface>`         | 显示指定接口的统计信息                  |
| 查看多播组        | `-g`, `--groups`     | 显示多播组成员关系                      |
| 查看协议统计      | `-s`, `--statistics` | 显示各协议（TCP/UDP/ICMP 等）的统计信息 |
| 查看 NAT 伪装连接 | `-M`, `--masquerade` | 显示被 NAT 伪装的连接信息               |

------

## 四、显示控制选项

| 选项                 | 说明                                       |
| -------------------- | ------------------------------------------ |
| `-v`, `--verbose`    | 输出详细信息                               |
| `-W`, `--wide`       | 不截断 IP 地址显示                         |
| `-n`, `--numeric`    | 以数字形式显示，不解析主机名、端口、用户名 |
| `--numeric-hosts`    | 仅主机名不解析                             |
| `--numeric-ports`    | 仅端口名不解析                             |
| `--numeric-users`    | 仅用户名不解析                             |
| `-N`, `--symbolic`   | 解析硬件接口名                             |
| `-e`, `--extend`     | 显示扩展信息（如 UID、Inode）              |
| `-p`, `--programs`   | 显示每个连接的 PID/进程名                  |
| `-o`, `--timers`     | 显示连接的定时器信息                       |
| `-c`, `--continuous` | 持续输出网络状态（类似实时监控）           |
| `-l`, `--listening`  | 显示处于监听状态的 socket                  |
| `-a`, `--all`        | 显示所有 socket，包括监听和非监听          |
| `-F`, `--fib`        | 显示 FIB（转发表）信息                     |
| `-C`, `--cache`      | 显示路由缓存而不是 FIB                     |
| `-Z`, `--context`    | 显示 SELinux 安全上下文信息                |

------

## 五、协议和地址族选项

### 🔹 Socket 协议类型

| 选项              | 说明                 |
| ----------------- | -------------------- |
| `-t`, `--tcp`     | 显示 TCP 连接        |
| `-u`, `--udp`     | 显示 UDP 连接        |
| `-U`, `--udplite` | 显示 UDP-Lite 连接   |
| `-S`, `--sctp`    | 显示 SCTP 连接       |
| `-w`, `--raw`     | 显示 RAW socket 连接 |
| `-x`, `--unix`    | 显示 UNIX 域套接字   |
| `--ax25`          | 显示 AX.25 连接      |
| `--ipx`           | 显示 IPX 连接        |
| `--netrom`        | 显示 NET/ROM 连接    |

------

### 🔹 地址族（Address Family）

可通过以下参数指定网络协议族（默认：`inet`）：

| 参数                  | 说明             |
| --------------------- | ---------------- |
| `-4`                  | 仅显示 IPv4 连接 |
| `-6`                  | 仅显示 IPv6 连接 |
| `-A <af>` 或 `--<af>` | 指定其他协议族   |

支持的地址族包括：

```
inet (IPv4)
inet6 (IPv6)
ax25 (AMPR AX.25)
netrom (AMPR NET/ROM)
ipx (Novell IPX)
ddp (Appletalk DDP)
x25 (CCITT X.25)
```

------

## 六、常用示例

### 1️⃣ 查看所有连接（TCP/UDP）

```bash
netstat -an
```

> 显示所有网络连接，使用数字地址与端口。

------

### 2️⃣ 查看监听端口

```bash
netstat -tuln
```

> 显示所有正在监听的 TCP/UDP 端口。

------

### 3️⃣ 查看进程对应端口

```bash
netstat -tulnp
```

> 显示每个监听端口对应的进程 ID 与程序名称。

------

### 4️⃣ 实时监控网络连接变化

```bash
netstat -c
```

> 每秒刷新一次网络状态，实时显示连接变化。

------

### 5️⃣ 查看路由表

```bash
netstat -rn
```

> 类似于 `route -n`，以数字形式显示系统的路由信息。

------

### 6️⃣ 查看接口统计信息

```bash
netstat -i
```

> 显示所有网络接口的状态与收发包统计。

------

### 7️⃣ 查看协议统计信息

```bash
netstat -s
```

> 输出 TCP、UDP、ICMP 等各层协议的详细统计数据。

------

### 8️⃣ 查看 UNIX 套接字

```bash
netstat -x
```

> 显示本地进程间通信（UNIX domain socket）状态。

------

### 9️⃣ 显示 NAT 伪装连接

```bash
netstat -M
```

> 查看被 NAT（IP 伪装）处理的连接情况。

------

## 七、输出字段说明（以 `netstat -tulnp` 为例）

| 字段             | 含义                                   |
| ---------------- | -------------------------------------- |
| Proto            | 协议类型（TCP/UDP）                    |
| Recv-Q           | 接收队列中的字节数                     |
| Send-Q           | 发送队列中的字节数                     |
| Local Address    | 本地 IP 和端口                         |
| Foreign Address  | 远程 IP 和端口                         |
| State            | 当前连接状态（LISTEN、ESTABLISHED 等） |
| PID/Program name | 进程 ID 及对应程序名（需 `-p` 选项）   |

------

## 八、常见用途总结

| 场景                   | 命令示例        |
| ---------------------- | --------------- |
| 查看服务器监听端口     | `netstat -tlnp` |
| 查看某个端口的占用进程 | `netstat -tulnp |
| 查看网络连接状态变化   | `netstat -c`    |
| 检查路由配置           | `netstat -rn`   |
| 检查接口状态           | `netstat -i`    |
| 查看协议统计           | `netstat -s`    |

------

## 九、注意事项

1. 某些信息（如进程名）需要 root 权限才能查看。

2. 若系统无 `netstat` 命令，可安装：

   ```bash
   yum install net-tools -y
   ```

3. 在较新系统（如 CentOS 8、Ubuntu 20.04+）中推荐使用：

   ```bash
   ss -tuln
   ```

   代替 `netstat`。

------

## 十、更多帮助

```bash
man netstat
netstat --help
```

