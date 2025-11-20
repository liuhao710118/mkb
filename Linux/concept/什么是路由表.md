## 1. 简介：什么是路由表

路由表（Routing Table）是操作系统用来决定网络数据包如何转发到目标地址的一张表。它包含了多条路由规则，当内核需要发送一个 IP 包时，会在路由表中查找最匹配的路由条目，从而决定选择哪张网卡、是否使用网关（下一跳）、或使用哪张路由表。

路由表存在于主机和路由器上。主机的路由表通常较简单（本地、默认网关、点到点），而路由器的路由表更复杂（静态路由、动态路由协议学到的路由等）。

## 2. 路由表的基本概念与术语

- **目的网络/前缀（Destination/Prefix）**：目标 IP 或子网（例如 `192.168.1.0/24`、`0.0.0.0/0`）。
- **下一跳（Next hop / via）**：数据包转发给哪个路由器/网关（如 `192.168.1.1`）。
- **出接口（Dev / Interface）**：使用哪个网卡发送（如 `eth0`、`ens160`）。
- **源地址（src）**：路由表有时会指定发送包使用的源 IP。
- **scope（作用域）**：路由可达性范围（`link`、`global`、`host` 等）。
- **proto（路由来源）**：该条目如何产生（`kernel`、`static`、`dhcp`、`boot`、`ospf` 等）。
- **metric（度量 / 优先级）**：路由的优先级（值越小优先级越高）。
- **route type（路由类型）**：比如 `unicast`、`blackhole`（丢弃）、`unreachable`（不可达）等。

## 3. 路由表条目详解（字段含义）

以常见 `ip route show` 输出的一行为例：

```
default via 192.168.246.2 dev ens160 proto static metric 100
```

字段解释：

- `default`：目的为 0.0.0.0/0，即所有非本地匹配的流量。
- `via 192.168.246.2`：下一跳网关地址。
- `dev ens160`：通过网络设备 `ens160` 发送。
- `proto static`：路由来自静态配置。
- `metric 100`：度量值，决定优先级。

另一个例子（本地直连）：

```
192.168.246.0/24 dev ens160 proto kernel scope link src 192.168.246.3 metric 100
```

解释：

- `192.168.246.0/24`：目标子网。
- `dev ens160`：直接从该网卡发送，使用 ARP 解析目标 MAC。
- `proto kernel`：内核自动生成（由网卡地址配置触发）。
- `scope link`：直接链路可达，不走网关。
- `src 192.168.246.3`：默认源地址。

## 4. 常见命令与输出解读（Linux）

### 查看路由

- `ip route` / `ip route show`（推荐）
- `route -n`（传统，显示数字格式）
- `netstat -rn`（已被 `ip` 取代，但仍可用）

示例：

```
$ ip route show
default via 10.0.0.1 dev eth0
10.0.0.0/24 dev eth0 proto kernel scope link src 10.0.0.10
```

### 更详细的路由表（包括策略路由表）

- `ip rule show`：显示策略路由规则（policy routing rules）。
- `ip route show table all`：列出所有路由表（含自定义表）。

### 查看邻居表（ARP）

- `ip neigh` 或 `arp -n`

### 查看路由缓存（在旧内核中）

- `ip route get <ip>`：显示内核如何选择路由以及会使用哪个源地址。例如：`ip route get 8.8.8.8`。

## 5. 添加、删除、修改路由（实战命令）

> **注意**：对生产服务器做路由修改前请谨慎操作，建议在控制台或物理访问环境下执行，避免断开 SSH 连接。

### 添加静态路由

- 添加到子网：

```
sudo ip route add 192.168.10.0/24 via 10.0.0.1 dev eth0
```

- 添加默认路由：

```
sudo ip route add default via 10.0.0.1 dev eth0 metric 100
```

### 删除路由

```
sudo ip route del 192.168.10.0/24 via 10.0.0.1
```

或删除默认路由：

```
sudo ip route del default via 10.0.0.1
```

### 修改路由

实际上 `ip` 没有专门的 `modify`，通常是先 `del` 再 `add`，或使用 `replace`：

```
sudo ip route replace default via 10.0.0.254 dev eth0
```

### 临时与持久

- 使用 `ip route` 修改的是运行时路由（重启或网卡重置后丢失）。要永久生效，需写到网络配置文件或使用系统网络管理器（见下一节）。

## 6. 持久化路由配置（主流发行版/工具）

不同系统持久化方式不同：

- **CentOS / RHEL（基于 ifcfg）**：在 `/etc/sysconfig/network-scripts/route-<IFACE>` 或在 ifcfg 文件中加入 `GATEWAY`、`IPADDR` 等。
- **Debian / Ubuntu（传统 /etc/network/interfaces）**：在 `/etc/network/interfaces` 中配置 `up ip route add ...`，或使用 `netplan`。
- **Ubuntu（netplan）**：在 `/etc/netplan/*.yaml` 中配置 `routes:`。
- **NetworkManager**：可在 `nm-connection-editor` 中添加静态路由，或编辑连接的 `ipv4.routes` 属性。
- **systemd-networkd**：编辑 `.network` 文件，`Routes=` 字段。

示例（netplan）：

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    eth0:
      dhcp4: false
      addresses: [10.0.0.10/24]
      gateway4: 10.0.0.1
      routes:
        - to: 192.168.10.0/24
          via: 10.0.0.1
```

## 7. 高级主题：策略路由（policy routing）与多路由表

当单一路由表无法满足复杂需求（如基于源地址、端口或防火墙 mark 路由）时，就需要策略路由（Policy-Based Routing，PBR）。Linux 使用 `ip rule` + 多个路由表来实现。

常见步骤：

1. 在 `/etc/iproute2/rt_tables` 添加自定义表名（例如 `200  homevpn`）。
2. 使用 `ip route` 将特定路由写入该表：

```
ip route add default via 192.0.2.1 table homevpn
```

1. 使用 `ip rule` 添加规则基于源地址、uid、fwmark 等选择路由表：

```
ip rule add from 192.168.10.0/24 table homevpn
ip rule add fwmark 0x1 table homevpn
```

使用场景：

- 双上行（两网关），不同源流量走不同网关。
- VPN 路由策略（部分流量走 VPN，部分直连）。

## 8. 路由选择过程（包如何被路由）

简化的查找逻辑：

1. 内核匹配 `ip rule`（policy routing）列表，找到使用哪个路由表（默认 `main` 表）。
2. 在指定表中按最长前缀匹配（Longest Prefix Match）查找目的地址对应路由条目。
3. 选中后根据条目判断是否需要 ARP（直接连接）或将包发给下一跳（via）。
4. 查询 ARP 表（neighbor）以获得目标 MAC，然后封装并发出。

“最长前缀匹配”意思：有多个匹配路由时，选择掩码最长（最具体）的那一条。例如 `192.168.0.0/16` 与 `192.168.1.0/24` 都匹配 `192.168.1.5`，但后者更具体，被选中。

## 9. 路由调试与排障技巧

### 常用命令

- `ip route get <addr>`：显示内核选择的路由及使用的源地址。
- `traceroute <host>` / `tracepath`：跟踪经过的路由器。
- `tcpdump -i <iface> host <ip>`：抓包，观察是否发出/收到数据包。
- `ip neigh`：查看 ARP 是否已解析。
- `ss -ntlp` 或 `netstat -rn`：查看监听端口与路由表（旧）。

### 排查思路

1. 先 `ip route` 看是否存在匹配条目。
2. `ip route get <dest>` 看内核如何选择。
3. 检查本地 IP/掩码/网卡是否正确（`ip addr`）。
4. 检查网关是否可达（`ping <gateway>`）。
5. 抓包验证是否发包以及下一跳是否回应（`tcpdump`）。
6. 检查防火墙（`iptables`/`nftables`）是否拦截流量。
7. 如果有策略路由，`ip rule show` 与 `ip route show table <x>` 一定要看。

## 10. 常见场景与案例分析

### 场景 A：无法访问公网

步骤：

- `ip route` 是否存在 `default` 路由？
- `ip route get 8.8.8.8` 出什么？
- `ping <gateway>` 成功吗？
- `tcpdump -i <iface> icmp or host 8.8.8.8` 看是否发出请求。

可能原因：默认路由缺失、网关不可达、防火墙策略、网卡配置错误。

### 场景 B：服务器有两块网卡，流量走错网卡

解决：使用策略路由根据 `from` 地址或 `iif` 指定不同路由表，或设置合适的 `metric`。

### 场景 C：VPN 后流量全部走 VPN（想做分流）

在 VPN 服务器或客户端上使用策略路由：把希望走直连的网段添加回主表，或在 VPN 客户端配置仅路由特定子网到 VPN（而非默认路由）。

## 11. 安全、最佳实践与常见误区

- **谨慎操作默认路由**：误删默认路由会导致远程 SSH 断开。
- **使用 `metric` 管理优先级**：在多条默认路由时用 metric 决定主备。
- **记录变更**：对生产系统做路由变更前做好备份。
- **最小特权**：只有必要时才添加黑洞（blackhole）或 unreachable 路由。
- **日志与监控**：监控路由可达性（例如 BGP/OSPF 环境）和关键网关。

常见误区：

- 以为 `ip route add` 就是持久生效（不是）。
- 混淆 `ip rule` 与 `ip route` 的关系（rule 选择表，route 在表里定义路由）。

## 12. 练习题与参考答案

**练习 1**：解释下列命令的效果：

```
ip route add 10.10.10.0/24 via 192.168.1.1 dev eth0 metric 50
```

**参考答案**：为到达 `10.10.10.0/24` 的流量添加一条路由，下一跳是 `192.168.1.1`，使用 `eth0` 发包，优先级（metric）为 50。

**练习 2**：如何让 `192.168.1.5` 这个源 IP 的流量走另一张网卡 `eth1` 的默认网关？

**参考答案摘要**：创建自定义路由表（例如 `200 mytable`），在该表中添加默认路由 `ip route add default via <eth1-gateway> table mytable`，然后用 `ip rule add from 192.168.1.5 table mytable`。

------

## 附录：快速命令速查表

- 查看：`ip route show`
- 查看策略规则：`ip rule show`
- 查看所有表：`ip route show table all`
- 临时添加路由：`ip route add ...`
- 删除路由：`ip route del ...`
- 替换路由：`ip route replace ...`
- 查看下一跳选择：`ip route get <ip>`
- 查看 ARP：`ip neigh`

