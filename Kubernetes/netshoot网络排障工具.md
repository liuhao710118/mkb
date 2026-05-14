# [nicolaka/netshoot 官方 GitHub](https://github.com/nicolaka/netshoot?utm_source=chatgpt.com)

netshoot 是 Kubernetes / Docker 场景下非常经典的网络排障工具镜像，被很多运维、SRE、DevOps 工程师称为：

> “网络故障排查瑞士军刀”

它本质上是一个预装了大量网络诊断工具的容器镜像。
你无需在业务 Pod 中安装 curl、tcpdump、dig、mtr 等工具，只需要临时拉起 netshoot 即可排查问题。 ([GitHub](https://github.com/nicolaka/netshoot?utm_source=chatgpt.com))

------

# 一、为什么 Kubernetes 必须要 netshoot

K8s 官方镜像越来越“瘦”：

- alpine
- distroless
- scratch
- busybox

这些镜像里：

- 没有 ping
- 没有 curl
- 没有 tcpdump
- 没有 dig
- 没有 netstat
- 没有 ss
- 没有 traceroute

导致：

```bash
kubectl exec -it pod bash
```

进去以后：

```bash
curl: command not found
ping: command not found
```

因此生产环境排障非常困难。

于是：

```bash
nicolaka/netshoot
```

成为 Kubernetes 网络排障事实标准镜像之一。 ([GitHub](https://github.com/nicolaka/netshoot?utm_source=chatgpt.com))

------

# 二、netshoot 的核心能力

netshoot 集成了：

| 类别           | 工具                  |
| -------------- | --------------------- |
| 网络连通性     | ping、curl、wget、nc  |
| DNS 排查       | dig、nslookup、drill  |
| 路由分析       | ip、route、traceroute |
| 抓包分析       | tcpdump、tshark       |
| 端口检测       | ss、netstat、lsof     |
| 网络性能       | iperf、iperf3         |
| 防火墙         | iptables、nft         |
| ARP/邻居表     | arp、ip neigh         |
| Kubernetes/CNI | calicoctl             |
| 流量监控       | iftop、iptraf-ng      |
| WebSocket      | websocat              |
| HTTP测试       | httpie                |
| TLS分析        | openssl               |
| gRPC测试       | grpcurl               |

官方列出的内置工具超过几十种。 ([GitHub](https://github.com/nicolaka/netshoot?utm_source=chatgpt.com))

------

# 三、Kubernetes 中最常见使用方式

------

## 1. 临时启动一个排障 Pod

最经典方式：

```bash
kubectl run tmp-shell \
  --rm -it \
  --image=nicolaka/netshoot
```

进入后：

```bash
bash
```

然后就能：

```bash
curl
ping
dig
tcpdump
```

等各种操作。

------

## 2. 在指定 Namespace 启动

```bash
kubectl run tmp-shell \
  -n prod \
  --rm -it \
  --image=nicolaka/netshoot
```

因为 Pod 在同一 namespace：

因此：

- DNS
- Service
- NetworkPolicy

都会更真实。

------

## 3. 进入现有 Pod 网络命名空间（最重要）

K8s 1.18+：

```bash
kubectl debug mypod \
  -it \
  --image=nicolaka/netshoot
```

这是目前最推荐方式。 ([GitHub](https://github.com/nicolaka/netshoot?utm_source=chatgpt.com))

原理：

- 创建 ephemeral container（临时容器）
- 与目标 Pod 共享 network namespace

因此：

你看到的：

- IP
- route
- iptables
- DNS
- socket

都和业务容器一致。

这比：

```bash
kubectl exec
```

强太多。

------

# 四、为什么 netshoot 特别适合 Kubernetes

------

## Kubernetes 网络本质

每个 Pod 都有：

- 独立网络命名空间（network namespace）

同一个 Pod 内多个容器：

- 共享 network namespace

因此：

sidecar/netshoot 可以直接看到：

```bash
eth0
lo
route
socket
```

等全部网络信息。 ([GitHub](https://github.com/nicolaka/netshoot?utm_source=chatgpt.com))

------

# 五、生产环境最常见排障场景

------

## 场景1：Pod 无法访问 Service

------

### Step1 查看 DNS

```bash
dig kubernetes.default.svc.cluster.local
```

或者：

```bash
nslookup nginx.default.svc.cluster.local
```

排查：

- CoreDNS
- DNS policy
- search domain

------

### Step2 查看 Service IP

```bash
kubectl get svc
```

------

### Step3 curl 测试

```bash
curl http://nginx.default.svc.cluster.local
```

------

### Step4 检查 Endpoint

```bash
kubectl get ep
```

如果 endpoints 为空：

说明：

- selector 不匹配
- Pod 未 ready

------

## 场景2：Pod 间无法通信

------

### ping Pod IP

```bash
ping 10.244.1.12
```

------

### nc 测试端口

```bash
nc -zv 10.244.1.12 8080
```

------

### route 检查

```bash
ip route
```

------

### 查看 CNI

```bash
kubectl get pods -n kube-system
```

检查：

- calico
- flannel
- cilium

是否异常。 ([Kubernetes Recipe Book](https://kubernetes.recipes/recipes/troubleshooting/kubernetes-network-troubleshooting/?utm_source=chatgpt.com))

------

## 场景3：怀疑 NetworkPolicy 拦截

这是 netshoot 最经典场景。

直接：

```bash
curl
nc
telnet
```

测试即可。

例如：

```bash
nc -zv redis 6379
```

如果失败：

大概率：

- ingress policy
- egress policy

有问题。

------

## 场景4：抓包（生产必会）

------

### tcpdump

```bash
tcpdump -i eth0
```

------

### 抓指定端口

```bash
tcpdump -i eth0 port 80
```

------

### 保存 pcap

```bash
tcpdump -i eth0 -w test.pcap
```

然后：

```bash
kubectl cp
```

下载到本地 Wireshark 分析。

Reddit 上很多 K8s 工程师都把：

```bash
kubectl debug + netshoot + tcpdump
```

作为生产排障神器。 ([reddit.com](https://www.reddit.com/r/kubernetes/comments/1nt88vu?utm_source=chatgpt.com))

------

# 六、netshoot 最重要工具详解

------

## 1. dig

DNS 排障核心。

```bash
dig baidu.com
```

查看：

- A记录
- TTL
- DNS服务器

------

## 2. curl

HTTP 排障。

```bash
curl -v http://service:8080
```

重点看：

- DNS解析
- TCP连接
- TLS握手
- HTTP状态码

------

## 3. ss

比 netstat 更现代。

查看监听端口：

```bash
ss -lntp
```

查看连接：

```bash
ss -ant
```

------

## 4. ip

Linux 网络核心命令。

查看 IP：

```bash
ip addr
```

查看路由：

```bash
ip route
```

查看邻居表：

```bash
ip neigh
```

------

## 5. tcpdump

网络抓包之王。

例如：

```bash
tcpdump -nn host 10.0.0.5
```

------

## 6. mtr

路由追踪增强版。

```bash
mtr 8.8.8.8
```

结合：

- ping
- traceroute

功能。

------

## 7. iperf3

网络性能测试。

服务端：

```bash
iperf3 -s
```

客户端：

```bash
iperf3 -c server-ip
```

测试：

- 带宽
- 丢包
- 延迟

------

# 七、HostNetwork 模式排障

------

## 直接进入宿主机网络

```bash
kubectl run tmp-shell \
  --rm -it \
  --overrides='{"spec":{"hostNetwork":true}}' \
  --image=nicolaka/netshoot
```

此时：

Pod 直接使用 Node 网络。

适合：

- kube-proxy
- nodePort
- 宿主机路由
- 宿主机iptables

排障。 ([GitHub](https://github.com/nicolaka/netshoot?utm_source=chatgpt.com))

------

# 八、netshoot + tcpdump 生产经典组合

------

## 抓某个 Pod 的流量

```bash
kubectl debug pod/nginx \
  -it \
  --image=nicolaka/netshoot \
  --target=nginx
```

然后：

```bash
tcpdump -i eth0
```

这是当前 Kubernetes 官方推荐调试方式之一。 ([reddit.com](https://www.reddit.com/r/kubernetes/comments/1t9wras/what_k8s_debugging_trick_would_you_have_wished/?utm_source=chatgpt.com))

------

# 九、排障思路（非常重要）

很多人不会排障，不是不会命令。

而是：

没有“网络分层思维”。

------

## Kubernetes 网络排障标准流程

------

### 第一层：DNS

检查：

```bash
dig
nslookup
```

------

### 第二层：TCP

检查：

```bash
nc
telnet
curl
```

------

### 第三层：路由

检查：

```bash
ip route
```

------

### 第四层：iptables

检查：

```bash
iptables -L
```

------

### 第五层：CNI

检查：

```bash
calico
flannel
cilium
```

------

### 第六层：抓包

最终：

```bash
tcpdump
```

看真实流量。

------

# 十、生产最佳实践

------

## 1. 不要把 netshoot 放正式业务镜像

原因：

- 工具太多
- 安全面扩大
- 镜像太大

正确方式：

- 临时 debug
- ephemeral container

------

## 2. 推荐使用 kubectl debug

不要：

```bash
kubectl exec
```

安装工具。

而是：

```bash
kubectl debug
```

------

## 3. 建议保留一个 debug namespace

例如：

```bash
kubectl create ns debug
```

专门跑：

- netshoot
- busybox
- debug pod

------

# 十一、和 busybox 的区别

| 工具     | 特点         |
| -------- | ------------ |
| busybox  | 极小         |
| alpine   | 轻量         |
| netshoot | 专业网络排障 |

------

# 十二、netshoot 缺点

------

## 1. 镜像大

因为工具很多。

------

## 2. 安全风险较高

包含：

- tcpdump
- nsenter
- iptables

不适合长期运行。

------

## 3. 不是最小权限

很多场景：

需要：

```yaml
NET_ADMIN
NET_RAW
privileged
```

------

# 十三、面试高频问题

------

## Q1：为什么 netshoot 能看到 Pod 网络？

因为：

同 Pod 内容器共享：

```text
network namespace
```

------

## Q2：为什么 kubectl debug 比 exec 更好？

因为：

生产镜像通常没有调试工具。

ephemeral container：

- 不污染业务镜像
- 更安全
- 更规范

------

## Q3：tcpdump 为什么抓不到包？

可能：

- 网卡不对
- 容器无 NET_RAW
- CNI 做了封装
- 流量走 vxlan

------

# 十四、推荐学习路线

建议你按下面顺序学：

1. Linux 网络基础
2. namespace
3. veth pair
4. bridge
5. iptables
6. Kubernetes CNI
7. kube-proxy
8. calico/cilium
9. tcpdump
10. netshoot

------

# 十五、最实用命令速查（建议收藏）

```bash
# 启动调试Pod
kubectl run tmp-shell --rm -it --image=nicolaka/netshoot

# 调试现有Pod
kubectl debug mypod -it --image=nicolaka/netshoot

# DNS测试
dig kubernetes.default.svc.cluster.local

# HTTP测试
curl -v http://nginx

# TCP测试
nc -zv nginx 80

# 查看路由
ip route

# 查看socket
ss -ant

# 抓包
tcpdump -i eth0

# 网络性能
iperf3 -s
iperf3 -c IP

# 查看iptables
iptables -L -n

# 查看邻居表
ip neigh
```

------

# 十六、社区评价

Reddit/K8s 社区里，很多工程师都把：

- netshoot
- k9s
- stern
- kubectl debug

列为 Kubernetes 排障必备工具。 ([reddit.com](https://www.reddit.com/r/kubernetes/comments/1nt88vu?utm_source=chatgpt.com))

有工程师评价：

> “一旦用了 netshoot，就回不去了。” ([reddit.com](https://www.reddit.com/r/kubernetes/comments/1t9wras/what_k8s_debugging_trick_would_you_have_wished/?utm_source=chatgpt.com))