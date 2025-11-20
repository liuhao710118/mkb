# **Linux `nslookup` 命令详解**

## 一、`nslookup` 是什么？

`nslookup`（Name Server Lookup）用于查询 DNS 信息，主要作用是：

- 查询域名对应的 IP 地址（A 记录）
- 查询 IP 对应的域名（PTR 反向解析）
- 指定 DNS 服务器进行查询
- 查看域名的各种 DNS 记录：A、AAAA、MX、TXT、CNAME 等
- 排查 DNS 故障

常用于判断 **域名无法访问** 或 **DNS 解析异常** 时的排查。

------

# **二、基本语法**

```
nslookup [选项] [域名/IP] [DNS服务器]
```

如：

```
nslookup www.google.com 8.8.8.8
```

------

# **三、工作模式**

### 1. **非交互模式（常用）**

直接查询单个域名/IP：

```
nslookup www.baidu.com
```

### 2. **交互模式**

输入 `nslookup` 后进入子命令环境：

```
> server 8.8.8.8
> set type=mx
> google.com
```

退出：

```
> exit
```

------

# **四、常用查询示例**

------

## **1. 查询域名对应的 IP**

```
nslookup www.baidu.com
```

输出示例：

```
Non-authoritative answer:
Name:    www.baidu.com
Address: 220.181.38.148
```

------

## **2. 使用指定 DNS 服务器解析**

```
nslookup www.google.com 8.8.8.8
```

------

## **3. 查询 MX（邮件服务器）记录**

```
nslookup -query=mx gmail.com
```

或交互模式下：

```
> set type=mx
> gmail.com
```

------

## **4. 查询 TXT 记录（如 SPF 信息）**

```
nslookup -query=txt google.com
```

------

## **5. 查询 CNAME 记录**

```
nslookup -type=CNAME mail.google.com
```

------

## **6. 查询 AAAA（IPv6）记录**

```
nslookup -type=AAAA google.com
```

------

## **7. 反向解析 IP 地址（PTR 记录）**

```
nslookup 8.8.8.8
```

输出示例：

```
Name: dns.google
```

------

# **五、交互模式常用指令**

进入：

```
nslookup
```

常用命令：

| 命令                   | 功能                               |
| ---------------------- | ---------------------------------- |
| `server <dns>`         | 切换 DNS 服务器                    |
| `set type=A`           | 查询类型（A/MX/TXT/AAAA/CNAME 等） |
| `set debug`            | 查看详细调试信息                   |
| `set recursion=yes/no` | 是否允许递归查询                   |
| `set timeout=5`        | 设置超时秒数                       |

示例：

```
> server 1.1.1.1
> set type=MX
> google.com
```

------

# **六、典型 DNS 故障排查**

------

## **1. 域名无法访问**：先检查 DNS 是否正常解析

```
nslookup example.com
```

如果报错：

```
** server can't find example.com: NXDOMAIN
```

说明域名不存在。

------

## **2. 本地 DNS 有问题？试试公共 DNS**

```
nslookup www.google.com 8.8.8.8
```

------

## **3. 检查 DNS 返回是否被篡改**

```
nslookup test.com 8.8.8.8
nslookup test.com 1.1.1.1
```

两个结果不一致 → 可能存在 DNS 污染或缓存问题。

------

## **4. 检查 MX 记录是否正常（邮件异常排查）**

```
nslookup -type=mx domain.com
```

------

## **5. 反向解析失败**

如果 PTR 记录不存在，会看到：

```
** server can't find 8.8.8.8: NXDOMAIN
```

------

# **七、常见参数速查表**

| 参数                      | 说明                |
| ------------------------- | ------------------- |
| `nslookup 域名`           | 普通解析            |
| `nslookup 域名 DNS服务器` | 用指定 DNS 查询     |
| `-type=A`                 | 查询 A 记录（默认） |
| `-type=AAAA`              | 查询 IPv6 地址      |
| `-type=MX`                | 查询邮件记录        |
| `-type=CNAME`             | 查询别名记录        |
| `-type=TXT`               | 查询 TXT 记录       |
| `-query=<类型>`           | 与 `-type=` 等同    |
| `-timeout=<秒>`           | 设置超时            |

------

# **八、与 dig 对比（dig 更强大）**

| 项目     | nslookup         | dig        |
| -------- | ---------------- | ---------- |
| 功能     | 基本解析         | **更强大** |
| 输出     | 简单             | **更详细** |
| 脚本使用 | 一般             | **推荐**   |
| 是否过时 | 部分系统标记为旧 | 最新工具   |

dig 示例：

```
dig google.com +trace
```

