# Kubernetes Service 详解

在 Kubernetes 中：

```text
Service
```

是 Kubernetes 网络模型最核心组件之一。

它的作用：

```text
为 Pod 提供稳定访问入口
```

------

# 一、为什么需要 Service

Pod 存在一个致命问题：

```text
Pod IP 会变化
```

例如：

```text
Pod 删除重建
↓
IP 改变
```

这样：

```text
应用之间无法稳定通信
```

因此 Kubernetes 引入：

```text
Service
```

------

# 二、Service 的本质

Service 本质上：

```text
一组 Pod 的统一访问入口
```

------

# 三、Service 解决的问题

------

## 1. Pod IP 不固定

Service 提供：

```text
固定虚拟 IP（ClusterIP）
```

------

## 2. Pod 动态变化

Service 自动发现 Pod。

------

## 3. 负载均衡

自动转发到多个 Pod。

------

# 四、Service 工作原理

```text
Client
   ↓
Service (VIP)
   ↓
Pod1
Pod2
Pod3
```

------

# 五、Service 如何找到 Pod

通过：

```text
Label Selector
```

------

## 示例

Pod：

```yaml
labels:
  app: nginx
```

------

Service：

```yaml
selector:
  app: nginx
```

------

# 六、最简单 Service YAML

```yaml
apiVersion: v1
kind: Service

metadata:
  name: nginx-svc

spec:
  selector:
    app: nginx

  ports:
    - port: 80
      targetPort: 80
```

------

# 七、Service YAML 详解

------

## 1. apiVersion

```yaml
apiVersion: v1
```

------

## 2. kind

```yaml
kind: Service
```

------

## 3. metadata

```yaml
metadata:
  name: nginx-svc
```

------

## 4. selector

最重要。

```yaml
selector:
  app: nginx
```

作用：

```text
找到对应 Pod
```

------

## 5. ports

端口映射。

```yaml
ports:
  - port: 80
    targetPort: 80
```

------

# 八、port 与 targetPort 区别

------

## port

Service 暴露端口。

------

## targetPort

Pod 容器端口。

------

## 示例

```yaml
ports:
  - port: 80
    targetPort: 8080
```

含义：

```text
Service:80
    ↓
Pod:8080
```

------

# 九、Service 类型（重点）

Service 有四种主要类型。

------

## 1. ClusterIP（默认）

------

### 作用

```text
集群内部访问
```

------

### YAML

```yaml
spec:
  type: ClusterIP
```

------

### 特点

- 默认类型
- 只能集群内访问
- 提供 ClusterIP

------

### 示例

```text
10.96.0.1
```

------

### 使用场景

- 微服务调用
- 数据库
- Redis
- 内部 API

------

## 2. NodePort

------

### 作用

```text
对外暴露服务
```

------

### 原理

```text
NodeIP:Port
```

访问。

------

### YAML

```yaml
spec:
  type: NodePort

  ports:
    - port: 80
      targetPort: 80
      nodePort: 30080
```

------

### 访问方式

```text
http://NodeIP:30080
```

------

### NodePort 范围

默认：

```text
30000-32767
```

------

## 3. LoadBalancer

云环境最常用。

------

### 原理

```text
云厂商负载均衡
    ↓
NodePort
    ↓
Pod
```

------

### YAML

```yaml
spec:
  type: LoadBalancer
```

------

### 云厂商自动创建

例如：

- AWS ELB
- 阿里云 SLB
- GCP LB

------

## 4. ExternalName

------

### 作用

DNS 别名。

------

### YAML

```yaml
apiVersion: v1
kind: Service

metadata:
  name: mysql

spec:
  type: ExternalName

  externalName: db.example.com
```

------

# 十三、Service 网络流程

------

## ClusterIP

```text
Client Pod
    ↓
ClusterIP
    ↓
iptables/ipvs
    ↓
Backend Pod
```

------

# 十四、kube-proxy

Service 真正实现核心。

------

## kube-proxy 作用

维护：

```text
iptables/ipvs 转发规则
```

------

## 工作流程

```text
Service
    ↓
Endpoint
    ↓
kube-proxy
    ↓
iptables/ipvs
```

------

# 十五、Endpoint

Service 对应的 Pod 列表。

------

## 查看

```bash
kubectl get endpoints
```

------

## 示例

```text
nginx-svc -> 10.244.1.5:80
```

------

# 十六、Service 与 Pod 通信

------

## Pod DNS

Kubernetes 自动提供 DNS。

------

## Service 访问方式

```text
service-name
```

例如：

```bash
curl http://nginx-svc
```

------

## 完整域名

```text
service.namespace.svc.cluster.local
```

------

## 示例

```text
nginx.default.svc.cluster.local
```

------

# 十七、Headless Service（重点）

------

## 什么是 Headless

没有 ClusterIP。

------

## YAML

```yaml
spec:
  clusterIP: None
```

------

## 为什么需要

直接返回：

```text
Pod IP
```

------

## 用途

- StatefulSet
- MySQL 集群
- Kafka
- Redis Cluster

------

# 十八、Headless 返回结果

普通 Service：

```text
一个 VIP
```

------

Headless：

```text
多个 Pod IP
```

------

# 十九、Session Affinity

------

## 默认

随机负载均衡。

------

## 开启会话保持

```yaml
sessionAffinity: ClientIP
```

------

# 二十、Service 与 Deployment 关系

------

## Deployment 创建 Pod

------

## Service 选择 Pod

通过：

```text
selector + labels
```

关联。

------

# 二十一、完整示例

------

## Deployment

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: nginx

spec:
  replicas: 3

  selector:
    matchLabels:
      app: nginx

  template:
    metadata:
      labels:
        app: nginx

    spec:
      containers:
        - name: nginx
          image: nginx

          ports:
            - containerPort: 80
```

------

## Service

```yaml
apiVersion: v1
kind: Service

metadata:
  name: nginx-svc

spec:
  selector:
    app: nginx

  ports:
    - port: 80
      targetPort: 80

  type: ClusterIP
```

------

# 二十二、查看 Service

------

## 查看 svc

```bash
kubectl get svc
```

------

## 查看详细信息

```bash
kubectl describe svc nginx-svc
```

------

## 查看 endpoints

```bash
kubectl get ep
```

------

## 查看 YAML

```bash
kubectl get svc nginx-svc -o yaml
```

------

# 二十三、Service 常见问题

------

## 1. Service 无法访问

检查：

```bash
kubectl get ep
```

如果：

```text
ENDPOINTS <none>
```

说明：

```text
selector 不匹配
```

------

## 2. Pod 没端口

检查：

```yaml
targetPort
```

------

## 3. DNS 失败

检查：

```bash
kubectl get pods -n kube-system
```

确认：

```text
CoreDNS
```

正常。

------

## 4. NodePort 无法访问

检查：

- 防火墙
- 安全组
- 节点网络

------

# 二十四、Service 与 Ingress 区别

| Service  | Ingress    |
| -------- | ---------- |
| 四层     | 七层       |
| TCP/UDP  | HTTP/HTTPS |
| 内部转发 | 域名路由   |

------

## Service

负责：

```text
Pod 访问
```

------

## Ingress

负责：

```text
域名访问
HTTPS
路径路由
```

------

# 二十五、Service 类型总结

| 类型         | 用途         |
| ------------ | ------------ |
| ClusterIP    | 集群内部     |
| NodePort     | 节点端口暴露 |
| LoadBalancer | 云负载均衡   |
| ExternalName | DNS 别名     |

------

# 二十六、生产最佳实践

------

## 1. Web 服务使用 ClusterIP + Ingress

不要直接 NodePort。

------

## 2. 数据库使用 Headless

例如：

- MySQL
- Kafka
- Redis

------

## 3. labels 规范化

```yaml
labels:
  app: nginx
  version: v1
```

------

## 4. readinessProbe 必须配置

避免：

```text
未启动完成接流量
```

------

# 二十七、生产架构

典型：

```text
Ingress
    ↓
Service
    ↓
Pod
```

------

# 二十八、面试高频问题

------

## 1. Service 为什么存在

因为：

```text
Pod IP 不固定
```

------

## 2. Service 如何找到 Pod

通过：

```text
selector + labels
```

------

## 3. Service 如何实现负载均衡

通过：

```text
kube-proxy + iptables/ipvs
```

------

## 4. Headless Service 用途

用于：

```text
StatefulSet
Kafka
MySQL
```

------

## 5. ClusterIP 为什么能访问

因为：

```text
iptables/ipvs
```

转发。

------

# 二十九、建议继续学习

下一步推荐：

1. Ingress
2. CoreDNS
3. kube-proxy
4. CNI 网络
5. StatefulSet
6. ConfigMap
7. Secret
8. PV/PVC
9. Helm
10. HPA