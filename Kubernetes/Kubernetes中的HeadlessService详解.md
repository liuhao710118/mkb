# Kubernetes Headless Service 详解

在 Kubernetes 中，普通 Service 会提供：

```text
ClusterIP（虚拟IP）
```

而：

```text
Headless Service
```

则：

```text
没有 ClusterIP
```

它不会做：

- kube-proxy 负载均衡
- VIP 转发

而是：

```text
直接返回 Pod IP
```

------

# 一、什么是 Headless Service

------

## 普通 Service

普通 Service：

```text
Client
   ↓
ClusterIP(VIP)
   ↓
iptables/ipvs
   ↓
Pod
```

例如：

```yaml
spec:
  type: ClusterIP
```

K8s 会自动分配：

```text
10.96.x.x
```

虚拟 IP。

------

## Headless Service

Headless：

```yaml
clusterIP: None
```

表示：

```text
不要 VIP
```

架构：

```text
Client
   ↓
DNS
   ↓
直接返回 Pod IP
```

------

# 二、为什么需要 Headless Service

普通 Service：

```text
适合无状态服务
```

因为：

```text
不关心具体 Pod
```

例如：

- Nginx
- SpringBoot
- API Gateway

------

但某些系统：

```text
必须知道具体节点
```

例如：

- MySQL 主从
- Redis Cluster
- Kafka
- Elasticsearch
- ZooKeeper
- MongoDB
- RabbitMQ

这些属于：

```text
有状态服务（Stateful）
```

它们要求：

```text
固定身份
固定 DNS
固定网络标识
```

------

# 三、Headless Service 核心作用

------

## 1. 直接暴露 Pod IP

DNS 查询：

```text
不返回 VIP
```

而返回：

```text
所有 Pod IP
```

------

## 2. StatefulSet 固定 DNS

这是：

```text
最重要用途
```

------

## 3. 服务发现

客户端：

```text
自行做负载均衡
```

------

# 四、Headless Service 工作原理

------

## 普通 Service DNS

普通 Service：

```text
myservice.default.svc.cluster.local
```

解析：

```text
10.96.0.10
```

（ClusterIP）

------

## Headless Service DNS

Headless：

```text
myservice.default.svc.cluster.local
```

解析：

```text
10.244.1.2
10.244.1.3
10.244.2.5
```

即：

```text
多个 Pod IP
```

------

# 五、Headless Service YAML

------

## 最简单示例

```yaml
apiVersion: v1
kind: Service

metadata:
  name: nginx-headless

spec:
  clusterIP: None

  selector:
    app: nginx

  ports:
  - port: 80
```

------

# 六、关键字段详解

------

## 1. clusterIP: None

核心：

```yaml
clusterIP: None
```

表示：

```text
不要 VIP
```

这是：

```text
Headless 唯一核心配置
```

------

## 2. selector

```yaml
selector:
  app: nginx
```

用于：

```text
匹配 Pod
```

------

## 3. ports

即使：

```text
没有 ClusterIP
```

也需要：

```yaml
ports:
```

否则：

```text
DNS SRV 记录无法生成
```

------

# 七、DNS 解析（重点）

------

## 1. 查看 DNS

进入 Pod：

```bash
nslookup nginx-headless
```

输出：

```text
Name: nginx-headless
Address: 10.244.1.2
Address: 10.244.1.3
Address: 10.244.2.5
```

------

## 2. dig 查看

```bash
dig nginx-headless.default.svc.cluster.local
```

------

## 3. 普通 Service 对比

普通 Service：

```text
返回一个 VIP
```

Headless：

```text
返回多个 Pod IP
```

------

# 八、Headless + StatefulSet（核心）

这是：

```text
生产环境最重要组合
```

------

# 九、StatefulSet 为什么必须 Headless

StatefulSet 特点：

```text
Pod 必须有固定身份
```

例如：

```text
mysql-0
mysql-1
mysql-2
```

它们必须有：

```text
固定 DNS
```

------

# 十、StatefulSet DNS 规则

------

假设：

Service：

```text
mysql-headless
```

StatefulSet：

```text
mysql
```

Namespace：

```text
default
```

------

则：

```text
mysql-0.mysql-headless.default.svc.cluster.local
mysql-1.mysql-headless.default.svc.cluster.local
mysql-2.mysql-headless.default.svc.cluster.local
```

都会固定存在。

------

# 十一、完整 StatefulSet 示例

------

## Headless Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mysql-headless
spec:
  clusterIP: None
  selector:
    app: mysql
  ports:
  - port: 3306
```

------

## StatefulSet

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql
spec:
  serviceName: mysql-headless
  replicas: 3
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:8
```

------

# 十二、最终 DNS

------

```text
mysql-0.mysql-headless
mysql-1.mysql-headless
mysql-2.mysql-headless
```

每个：

```text
都对应固定 Pod
```

------

# 十三、Kafka 为什么需要 Headless

Kafka Broker：

```text
Broker 之间需要互相通信
```

并且：

```text
必须知道对方真实 IP
```

不能：

```text
经过 VIP
```

否则：

- Broker ID 混乱
- Leader 选举异常
- 元数据错误

因此：

```text
Kafka StatefulSet
+
Headless Service
```

是标准方案。

------

# 十四、Redis Cluster 为什么需要

Redis Cluster：

```text
节点间 gossip 通信
```

需要：

```text
真实节点地址
```

------

# 十五、Headless 不做负载均衡

普通 Service：

```text
kube-proxy
负责负载均衡
```

Headless：

```text
不做 LB
```

客户端：

```text
自行决定访问哪个 Pod
```

------

# 十六、Headless Service Endpoint

查看：

```bash
kubectl get ep
```

输出：

```text
NAME               ENDPOINTS
nginx-headless     10.244.1.2,10.244.1.3
```

------

# 十七、publishNotReadyAddresses

默认：

```text
只有 Ready Pod 才加入 DNS
```

------

## 有状态场景问题

例如：

```text
MySQL 集群初始化
```

节点：

```text
需要互相发现
```

即使：

```text
还没 Ready
```

------

## 解决方案

```yaml
publishNotReadyAddresses: true
```

------

## 示例

```yaml
spec:
  clusterIP: None

  publishNotReadyAddresses: true
```

------

# 十八、SRV 记录（高级）

Headless 支持：

```text
SRV DNS
```

用于：

```text
服务发现
```

------

## 查询

```bash
dig SRV _mysql._tcp.mysql-headless.default.svc.cluster.local
```

------

## 返回

```text
mysql-0.mysql-headless
mysql-1.mysql-headless
```

------

# 十九、Headless 与普通 Service 区别

| 对比       | 普通 Service | Headless |
| ---------- | ------------ | -------- |
| ClusterIP  | 有           | 无       |
| kube-proxy | 使用         | 不使用   |
| DNS 返回   | VIP          | Pod IP   |
| 负载均衡   | kube-proxy   | 客户端   |
| 适合       | 无状态       | 有状态   |

------

# 二十、生产最佳实践

------

## 1. StatefulSet 必须搭配 Headless

标准组合：

```text
StatefulSet
+
Headless Service
```

------

## 2. 不要给 Web 服务用 Headless

例如：

- Nginx
- SpringBoot

因为：

```text
客户端无法自动负载均衡
```

------

## 3. Kafka/Redis/MySQL 强烈推荐

因为：

```text
需要固定身份
```

------

## 4. 使用 publishNotReadyAddresses

集群初始化场景必须开启。

------

# 二十一、常见问题

------

## 1. nslookup 没返回 Pod IP

检查：

```text
Pod 是否 Ready
```

------

## 2. StatefulSet DNS 不生效

检查：

```yaml
serviceName:
```

是否正确。

------

## 3. Pod 名称变化

Deployment：

```text
Pod 名随机
```

StatefulSet：

```text
Pod 名固定
```

------

# 二十二、面试高频问题

------

## 1. 什么是 Headless Service？

```text
没有 ClusterIP 的 Service
```

------

## 2. Headless 与普通 Service 区别？

```text
普通 Service 返回 VIP
Headless 返回 Pod IP
```

------

## 3. 为什么 StatefulSet 需要 Headless？

因为：

```text
StatefulSet 需要固定 DNS
```

------

## 4. Headless 是否负载均衡？

```text
不负责
```

由：

```text
客户端自己做
```

------

## 5. Kafka 为什么必须 Headless？

因为：

```text
Broker 需要真实网络身份
```

------

# 二十三、总结

------

## Headless 本质

```text
无 VIP Service
```

------

## 核心能力

| 能力        | 说明         |
| ----------- | ------------ |
| 返回 Pod IP | 服务发现     |
| 固定 DNS    | StatefulSet  |
| 不做 LB     | 客户端控制   |
| 支持 SRV    | 高级服务发现 |

------

## 最重要生产组合

```text
StatefulSet
+
Headless Service
```

------

## 最典型场景

| 系统          | 原因        |
| ------------- | ----------- |
| Kafka         | Broker 通信 |
| Redis Cluster | Gossip      |
| MySQL 主从    | 固定节点    |
| ZooKeeper     | 固定身份    |
| Elasticsearch | 节点发现    |

------

## 一句话理解

```text
普通 Service：
访问服务

Headless Service：
发现具体 Pod
```