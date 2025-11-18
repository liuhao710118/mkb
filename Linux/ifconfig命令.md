# **Linux `ifconfig` 命令教程文档**

`ifconfig`（interface configure）用于 **查看和配置网络接口**，包括 IP、MAC、网卡启停、MTU 等。

常用于：

- 查看网卡信息
- 配置临时 IP 地址
- 启用/关闭网络接口
- 配置广播地址、子网掩码
- 排查网络问题

------

# **一、基本语法**

```
ifconfig [接口] [参数]
```

如果不带参数，列出所有活动的网络接口。

------

# **二、查看网络接口信息**

------

## **1. 查看所有启用的网络接口**

```
ifconfig
```

示例输出（部分）：

```
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  
        inet 192.168.1.10  netmask 255.255.255.0  broadcast 192.168.1.255
        inet6 fe80::20c:29ff:fe37:abcd  prefixlen 64
        ether 00:0c:29:37:ab:cd  txqueuelen 1000  (Ethernet)
```

常见字段说明：

| 字段           | 含义                      |
| -------------- | ------------------------- |
| **inet**       | IPv4 地址                 |
| **inet6**      | IPv6 地址                 |
| **ether**      | MAC 地址                  |
| **netmask**    | 子网掩码                  |
| **broadcast**  | 广播地址                  |
| **UP/RUNNING** | 网卡已启动并正常工作      |
| **MTU**        | 最大传输单元（默认 1500） |

------

## **2. 查看指定网卡**

```
ifconfig eth0
```

------

# **三、配置 IP 地址（临时）**

⚠ 重要说明：使用 `ifconfig` 设置 IP 是临时的，重启后失效。

------

## **1. 设置 IP 地址和子网掩码**

```
ifconfig eth0 192.168.1.100 netmask 255.255.255.0
```

------

## **2. 设置广播地址**

```
ifconfig eth0 broadcast 192.168.1.255
```

------

## **3. 同时配置 IP、子网掩码、广播地址**

```
ifconfig eth0 192.168.1.100 netmask 255.255.255.0 broadcast 192.168.1.255
```

------

# **四、启用/禁用网络接口**

------

## **1. 启用网卡（up）**

```
ifconfig eth0 up
```

------

## **2. 禁用网卡（down）**

```
ifconfig eth0 down
```

适用于：

- 临时断网
- 重置接口
- 调试网络

------

# **五、配置网卡 MAC 地址**

⚠ 需要 root 权限，部分系统不允许随意修改。

```
ifconfig eth0 hw ether 00:11:22:33:44:55
```

------

# **六、配置 MTU**

默认 MTU = 1500，可在 VPN、隧道、特殊网络中调整。

```
ifconfig eth0 mtu 1400
```

------

# **七、查看发送与接收统计**

```
ifconfig eth0
```

输出包含：

```
RX packets
TX packets
errors
dropped
overruns
carrier
```

用于网络性能排查。

------

# **八、启用混杂模式（抓包时用）**

混杂模式允许网卡接收所有经过的报文，而不仅是发给本机的。

```
ifconfig eth0 promisc
```

取消混杂模式：

```
ifconfig eth0 -promisc
```

------

# **九、和 `ip` 命令的区别（重要）**

现代系统推荐使用 `ip`：

| 功能         | ifconfig     | ip   |
| ------------ | ------------ | ---- |
| 配置 IP      | ✔            | ✔    |
| 修改路由     | ✖            | ✔    |
| 显示详细信息 | 较少         | 完整 |
| 已维护       | 否（已废弃） | ✔    |
| 推荐使用     | ❌            | ✔✔✔  |

例如 `ifconfig` 的替代：

```
ip addr show
ip link set eth0 up
ip addr add 192.168.1.10/24 dev eth0
```

------

# **十、常用排错命令示例**

------

## **1. 检查 IP 配置**

```
ifconfig
```

------

## **2. 重启网卡**

```
ifconfig eth0 down
ifconfig eth0 up
```

------

## **3. 检查是否 UP**

```
ifconfig eth0 | grep UP
```

------

# **十一、总结**

`ifconfig` 是传统的网络接口管理工具，用于：

- 查看网卡状态
- 配置临时 IP
- 启动/关闭网卡
- 修改 MAC、MTU
- 开启混杂模式

最常用命令：

```
ifconfig                # 查看网卡信息
ifconfig eth0           # 查看网卡 eth0
ifconfig eth0 up        # 启动网卡
ifconfig eth0 down      # 关闭网卡
ifconfig eth0 192.168.1.10 netmask 255.255.255.0
```

