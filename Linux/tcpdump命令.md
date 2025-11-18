`tcpdump`是 Linux 系统下的**网络数据包分析工具**，相当于网络的"窃听器"或"抓包工具"。它允许你捕获和分析流经网络接口的数据包。

## 核心功能：网络世界的"监听设备"

想象一下：

- 

  **网络数据包** = 邮寄的信件

- 

  **网络接口** = 邮局

- 

  **tcpdump** = 邮局里的信件检查员，可以查看每封信的内容

## 基本使用

### 安装 tcpdump

```
# CentOS/Rocky Linux
sudo dnf install tcpdump

# Ubuntu/Debian  
sudo apt install tcpdump
```

### 最简单的使用

```
# 捕获所有网络接口的数据包（按 Ctrl+C 停止）
sudo tcpdump

# 捕获指定接口的数据包
sudo tcpdump -i eth0
sudo tcpdump -i wlan0
```

## 常用参数详解

### 接口选择

```
# 查看可用接口
tcpdump -D

# 捕获特定接口
sudo tcpdump -i eth0
sudo tcpdump -i any          # 所有接口

# 无线网络监控
sudo tcpdump -i wlan0
```

### 输出控制

```
# 不解析主机名（显示IP而不是域名，更快）
sudo tcpdump -n

# 更详细的输出
sudo tcpdump -v
sudo tcpdump -vv             # 更详细
sudo tcpdump -vvv            # 最详细

# 十六进制和ASCII显示数据包内容
sudo tcpdump -XX
sudo tcpdump -A              # 只显示ASCII

# 限制捕获包数量
sudo tcpdump -c 10           # 只捕获10个包
```

### 保存和读取捕获文件

```
# 保存到文件（不显示在屏幕）
sudo tcpdump -w capture.pcap

# 从文件读取分析
sudo tcpdump -r capture.pcap

# 同时保存到文件并显示在屏幕
sudo tcpdump -w capture.pcap -l | tee capture.txt
```

## 强大的过滤表达式

这是 tcpdump 最强大的功能！你可以精确捕获特定流量。

### 基于主机的过滤

```
# 捕获与特定主机相关的所有流量
sudo tcpdump host 192.168.1.100

# 只捕获源IP是192.168.1.100的流量
sudo tcpdump src host 192.168.1.100

# 只捕获目标IP是192.168.1.1的流量  
sudo tcpdump dst host 192.168.1.1

# 捕获两个主机之间的流量
sudo tcpdump host 192.168.1.100 and host 192.168.1.1
```

### 基于网络的过滤

```
# 捕获整个网段的流量
sudo tcpdump net 192.168.1.0/24

# 捕获源网络
sudo tcpdump src net 192.168.1.0/24

# 捕获目标网络
sudo tcpdump dst net 10.0.0.0/8
```

### 基于端口的过滤

```
# 捕获特定端口的流量
sudo tcpdump port 80
sudo tcpdump port 443

# 端口范围
sudo tcpdump portrange 20-25

# 源端口或目标端口
sudo tcpdump src port 53       # DNS查询
sudo tcpdump dst port 80       # 目标Web服务器
```

### 基于协议的过滤

```
# 常见协议
sudo tcpdump tcp               # 只捕获TCP流量
sudo tcpdump udp               # 只捕获UDP流量  
sudo tcpdump icmp              # 只捕获ping包
sudo tcpdump arp               # ARP请求/响应

# IP协议版本
sudo tcpdump ip6               # IPv6流量
sudo tcpdump ip                 # IPv4流量
```

## 实际应用场景示例

### 场景1：监控Web流量

```
# 捕获HTTP流量（端口80）
sudo tcpdump -i eth0 -n port 80

# 捕获HTTPS流量（端口443），显示ASCII内容
sudo tcpdump -i eth0 -A port 443

# 捕获特定网站的流量
sudo tcpdump -i eth0 -n host google.com and port 80
```

### 场景2：诊断网络连接问题

```
# 监控与特定服务器的所有连接
sudo tcpdump -i eth0 -n host 8.8.8.8

# 监控DNS查询
sudo tcpdump -i eth0 -n port 53

# 监控ping包（ICMP）
sudo tcpdump -i eth0 -n icmp
```

### 场景3：分析特定协议

```
# SSH连接分析
sudo tcpdump -i eth0 -n port 22

# 数据库连接监控
sudo tcpdump -i eth0 -n port 3306    # MySQL
sudo tcpdump -i eth0 -n port 5432   # PostgreSQL
```

## 复杂过滤表达式组合

### 逻辑运算符

```
# AND（与）
sudo tcpdump host 192.168.1.100 and port 80

# OR（或）
sudo tcpdump port 80 or port 443

# NOT（非）
sudo tcpdump not port 22            # 排除SSH流量
sudo tcpdump host 192.168.1.100 and not port 22
```

### 分组过滤

```
# 使用括号分组（需要转义）
sudo tcpdump 'host 192.168.1.100 and (port 80 or port 443)'

# 复杂的业务逻辑过滤
sudo tcpdump 'src net 192.168.1.0/24 and (tcp port 80 or udp port 53)'
```

## 高级技巧和实战案例

### 案例1：分析HTTP请求

```
# 捕获HTTP GET/POST请求
sudo tcpdump -i eth0 -A -s 0 'tcp port 80 and (((ip[2:2] - ((ip[0]&0xf)<<2)) - ((tcp[12]&0xf0)>>2)) != 0)'

# 简化版：捕获包含HTTP方法的包
sudo tcpdump -i eth0 -A -s 0 'tcp port 80 and (tcp[((tcp[12]&0xf0)>>2):4] = 0x47455420)'
```

### 案例2：监控FTP密码（安全测试）

```
# FTP密码是明文传输的
sudo tcpdump -i eth0 -A -s 0 port 21
```

### 案例3：网络性能分析

```
# 捕获TCP标志位分析连接问题
sudo tcpdump -i eth0 -n 'tcp[tcpflags] & (tcp-syn|tcp-fin) != 0'

# 监控TCP重传
sudo tcpdump -i eth0 -n 'tcp[13] & 4 != 0'
```

## 读懂 tcpdump 输出

### 典型输出示例

```
13:45:30.123456 IP 192.168.1.100.54321 > 93.184.216.34.80: Flags [S], seq 123456789, win 64240, options [mss 1460,sackOK,TS val 100 ecr 0,nop,wscale 7], length 0
```

**分解解读：**

- 

  `13:45:30.123456`- 时间戳

- 

  `IP`- 协议类型（IPv4）

- 

  `192.168.1.100.54321`- 源IP和端口

- 

  `93.184.216.34.80`- 目标IP和端口（80是HTTP）

- 

  `Flags [S]`- TCP标志（S=SYN，握手开始）

- 

  `seq 123456789`- 序列号

- 

  `win 64240`- 窗口大小

### TCP标志位含义

```
[S] - SYN（建立连接）
[F] - FIN（结束连接）  
[P] - PUSH（推送数据）
[R] - RST（重置连接）
[.] - ACK（确认）
```

## 实用脚本示例

### 实时监控脚本

```
#!/bin/bash
# 实时网络监控脚本

INTERFACE="eth0"
LOG_FILE="/tmp/network_monitor_$(date +%Y%m%d_%H%M%S).log"

echo "开始网络监控，接口: $INTERFACE"
echo "日志文件: $LOG_FILE"

sudo tcpdump -i $INTERFACE -n -l | tee $LOG_FILE
```

### 异常连接检测

```
#!/bin/bash
# 检测异常外网连接

echo "监控异常外网连接..."
sudo tcpdump -i eth0 -n 'dst net not 192.168.0.0/16 and not 10.0.0.0/8 and not 172.16.0.0/12' | head -20
```

## 注意事项

### 权限和道德

```
# tcpdump需要root权限
sudo tcpdump

# 重要：只在你自己管理的网络或获得授权的网络中使用
# 未经授权捕获他人流量可能违法！
```

### 性能考虑

```
# 在高流量网络上使用过滤，避免系统过载
sudo tcpdump -i eth0 -c 1000 port 80    # 限制包数量

# 使用BPF过滤器提高性能
sudo tcpdump -i eth0 'tcp port 80'
```

## 总结

**tcpdump 是网络管理员、开发者和安全专家的瑞士军刀，主要用途：**

| 用途             | 命令示例                   |
| ---------------- | -------------------------- |
| **网络故障诊断** | `tcpdump -n host 目标IP`   |
| **协议分析**     | `tcpdump -A port 80`       |
| **安全监控**     | `tcpdump -w capture.pcap`  |
| **性能分析**     | `tcpdump -n 'tcp port 80'` |
| **应用调试**     | `tcpdump -i lo port 3306`  |

**核心价值**：tcpdump 让你能够"看到"网络上实际流动的数据，是理解网络通信、诊断问题、分析协议不可或缺的工具。

掌握 tcpdump 就像获得了网络的"X光视力"，能够深入理解数据如何在不同系统间流动！