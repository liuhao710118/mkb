# Kubernetes Ingress 详解

Ingress 是 Kubernetes 中：

> “七层（HTTP/HTTPS）流量入口管理”

它的作用：

```text
把外部用户请求
转发到集群内部 Service
```

本质：

```text
K8s 的 HTTP/HTTPS 网关
```

------

# 一、为什么需要 Ingress

先理解：

K8s Pod 默认：

```text
集群外无法访问
```

因为：

Pod IP：

- 是集群内部 IP
- 外部网络不可达

因此：

需要暴露服务。

------

# 二、K8s 暴露服务的几种方式

| 类型         | 层级      | 场景     |
| ------------ | --------- | -------- |
| ClusterIP    | 集群内部  | 默认     |
| NodePort     | 四层      | 测试     |
| LoadBalancer | 云厂商 LB | 公有云   |
| Ingress      | 七层 HTTP | 生产主流 |

------

# 三、NodePort 的问题

NodePort：

例如：

```text
192.168.1.10:30080
```

问题很多：

------

## 1. 端口难记

每个服务：

一个随机高端口。

------

## 2. 无法统一域名

例如：

```text
api.company.com
web.company.com
```

NodePort 做不到。

------

## 3. 无法做七层路由

不能：

- 根据 URL 转发
- 根据 Host 转发

------

## 4. HTTPS 麻烦

TLS 证书管理困难。

------

# 四、Ingress 的本质

Ingress：

本身不是流量转发程序。

它只是：

```text
一套路由规则
```

真正工作的：

是：

```text
Ingress Controller
```

------

# 五、Ingress 架构（非常重要）

```text
用户
  ↓
Ingress Controller
  ↓
Ingress Rules
  ↓
Service
  ↓
Pod
```

------

# 六、核心组成

Ingress 有两部分：

| 组件               | 作用         |
| ------------------ | ------------ |
| Ingress            | 路由规则     |
| Ingress Controller | 真正转发流量 |

------

# 七、Ingress Controller 是什么

Controller：

是真正：

- 接收 HTTP 请求
- 转发流量
- 负载均衡

的软件。

------

# 八、常见 Ingress Controller

生产最常用：

| Controller      | 特点         |
| --------------- | ------------ |
| NGINX Ingress   | 最主流       |
| Traefik         | 云原生       |
| HAProxy Ingress | 高性能       |
| Kong            | API 网关     |
| Istio Gateway   | Service Mesh |

------

# 九、为什么 Ingress 不能单独工作

很多新手：

创建了：

```yaml
kind: Ingress
```

然后：

发现访问不了。

原因：

```text
没有安装 Ingress Controller
```

Ingress：

只是规则。

Controller：

才是真正干活的。

------

# 十、最主流：NGINX Ingress

最常用的是：

ingress-nginx

本质：

```text
K8s + NGINX
```

Controller：

会动态生成：

```text
nginx.conf
```

实现流量转发。

------

# 十一、安装 Ingress NGINX

官方：

[ingress-nginx 官方网站](https://kubernetes.github.io/ingress-nginx/?utm_source=chatgpt.com)

------

## 安装（裸机常见）

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/cloud/deploy.yaml
```

------

# 十二、安装后会发生什么

会创建：

| 资源             | 作用             |
| ---------------- | ---------------- |
| Deployment       | nginx-controller |
| Service          | 暴露入口         |
| ConfigMap        | nginx 配置       |
| AdmissionWebhook | 校验 ingress     |

------

# 十三、查看 Controller

```bash
kubectl get pods -n ingress-nginx
```

------

# 十四、查看 Service

```bash
kubectl get svc -n ingress-nginx
```

通常：

```text
NodePort
```

或者：

```text
LoadBalancer
```

------

# 十五、Ingress 工作流程（核心）

用户：

访问：

```text
www.test.com/api
```

流程：

```text
DNS
 ↓
Ingress Controller
 ↓
匹配 Ingress 规则
 ↓
转发到 Service
 ↓
转发到 Pod
```

------

# 十六、Ingress 最核心能力

------

## 1. Host 路由

不同域名：

转发不同服务。

例如：

| 域名         | 服务        |
| ------------ | ----------- |
| api.test.com | api-service |
| web.test.com | web-service |

------

## 2. Path 路由

不同路径：

转发不同服务。

例如：

| 路径  | 服务         |
| ----- | ------------ |
| /api  | api-service  |
| /user | user-service |

------

## 3. HTTPS/TLS

支持：

- SSL
- HTTPS
- TLS termination

------

## 4. 负载均衡

自动：

转发多个 Pod。

------

# 十七、最简单 Ingress

------

## Service

先有：

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
  - port: 80
```

------

## Ingress

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx-ingress
spec:
  ingressClassName: nginx
  rules:
  - host: test.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nginx-service
            port:
              number: 80
```

------

# 十八、字段详解

------

## ingressClassName

```yaml
ingressClassName: nginx
```

指定：

```text
哪个 ingress controller 处理
```

因为：

集群可能有多个 controller。

------

## rules

定义：

```text
路由规则
```

------

## host

```yaml
host: test.com
```

表示：

匹配域名。

------

## paths

定义：

URL 路径规则。

------

## backend

最终转发：

哪个 Service。

------

# 十九、pathType（重点）

K8s 1.18 后必须写。

------

## 1. Prefix

```yaml
pathType: Prefix
```

前缀匹配。

例如：

```text
/api
```

匹配：

```text
/api/user
/api/test
```

------

## 2. Exact

精确匹配。

例如：

```text
/api
```

只匹配：

```text
/api
```

不匹配：

```text
/api/user
```

------

## 3. ImplementationSpecific

由 controller 自己决定。

不推荐。

------

# 二十、Host 路由实战

------

## YAML

```yaml
rules:
- host: api.test.com
  http:
    paths:
    - path: /
      pathType: Prefix
      backend:
        service:
          name: api-service
          port:
            number: 80

- host: web.test.com
  http:
    paths:
    - path: /
      pathType: Prefix
      backend:
        service:
          name: web-service
          port:
            number: 80
```

------

# 二十一、效果

```text
api.test.com → api-service
web.test.com → web-service
```

------

# 二十二、Path 路由实战

```yaml
rules:
- host: test.com
  http:
    paths:
    - path: /api
      pathType: Prefix
      backend:
        service:
          name: api-service
          port:
            number: 80

    - path: /user
      pathType: Prefix
      backend:
        service:
          name: user-service
          port:
            number: 80
```

------

# 二十三、效果

```text
test.com/api  → api-service
test.com/user → user-service
```

------

# 二十四、TLS/HTTPS（重点）

Ingress：

最重要能力之一：

```text
HTTPS 统一入口
```

------

# 二十五、TLS Secret

HTTPS 证书：

必须存放到：

```text
Secret
```

------

## 创建证书 Secret

```bash
kubectl create secret tls test-tls \
  --cert=server.crt \
  --key=server.key
```

------

# 二十六、Ingress TLS 配置

```yaml
spec:
  tls:
  - hosts:
    - test.com

    secretName: test-tls
```

------

# 二十七、HTTPS 流程

```text
用户 HTTPS
 ↓
Ingress Controller 解密 TLS
 ↓
内部 HTTP 转发
 ↓
Service
```

这叫：

```text
TLS Termination
```

------

# 二十八、默认后端（Default Backend）

如果：

没有匹配规则。

会进入：

```text
default backend
```

通常：

返回：

```text
404
```

------

# 二十九、Ingress 注解（非常重要）

NGINX Ingress：

大量功能：

通过：

```yaml
annotations:
```

实现。

------

# 三十、常见注解

------

## 1. URL 重写

```yaml
nginx.ingress.kubernetes.io/rewrite-target: /
```

------

## 2. 强制 HTTPS

```yaml
nginx.ingress.kubernetes.io/ssl-redirect: "true"
```

------

## 3. 限流

```yaml
nginx.ingress.kubernetes.io/limit-rps: "10"
```

------

## 4. 上传大小

```yaml
nginx.ingress.kubernetes.io/proxy-body-size: 100m
```

------

## 5. 超时

```yaml
nginx.ingress.kubernetes.io/proxy-read-timeout: "300"
```

------

# 三十一、Rewrite 示例（高频）

------

## 需求

```text
/api/user
↓
转发时变成：
/user
```

------

## YAML

```yaml
annotations:
  nginx.ingress.kubernetes.io/rewrite-target: /$2
```

配合：

```yaml
path: /api(/|$)(.*)
```

------

# 三十二、Ingress 与 Service 区别（面试高频）

------

## Service

四层：

```text
TCP/UDP
```

------

## Ingress

七层：

```text
HTTP/HTTPS
```

------

# 三十三、Ingress 只能处理 HTTP 吗

基本是。

Ingress：

主要：

```text
L7 HTTP/HTTPS
```

TCP/UDP：

需要：

- Service
- 或特殊配置

------

# 三十四、生产中的典型架构

------

## 裸机环境

```text
用户
 ↓
NodePort
 ↓
NGINX Ingress
 ↓
Service
 ↓
Pod
```

------

云环境

```text
用户
 ↓
云LB
 ↓
Ingress Controller
 ↓
Service
 ↓
Pod
```

------

# 三十五、IngressClass（新版本重点）

以前：

通过 annotation：

```yaml
kubernetes.io/ingress.class: nginx
```

现在：

推荐：

```yaml
ingressClassName: nginx
```

------

# 三十六、查看 Ingress

```bash
kubectl get ingress
```

------

# 三十七、查看详情

```bash
kubectl describe ingress nginx-ingress
```

------

# 三十八、排查 Ingress 问题（生产重要）

------

## 1. 查看 ingress

```bash
kubectl get ingress
```

------

## 2. 查看 controller

```bash
kubectl get pods -n ingress-nginx
```

------

## 3. 查看 controller 日志

```bash
kubectl logs -n ingress-nginx pod-name
```

------

## 4. 查看 service

```bash
kubectl get svc
```

------

## 5. 查看 endpoints

```bash
kubectl get ep
```

------

# 三十九、常见错误

------

## 1. 没装 ingress controller

最常见。

------

## 2. Service selector 错误

导致：

```text
Endpoints为空
```

------

## 3. DNS 未解析

域名：

没指向 ingress IP。

------

## 4. pathType 缺失

新版本：

必须写。

------

## 5. ingressClassName 不匹配

Controller：

不会接管。

------

# 四十、生产最佳实践

------

## 1. HTTPS 必开

生产：

必须 TLS。

------

## 2. 多副本 ingress-controller

避免单点。

------

## 3. 配合 HPA

自动扩缩容。

------

## 4. 使用 ExternalDNS

自动同步 DNS。

------

## 5. 配合 Cert-Manager

自动签发 HTTPS 证书。

常配合：

cert-manager

------

# 四十一、Ingress 的本质（最重要）

本质：

```text
HTTP 路由规则
```

真正干活：

是：

```text
Ingress Controller
```

------

# 四十二、面试高频问题

------

## Q1：Ingress 和 Service 区别

| Service  | Ingress    |
| -------- | ---------- |
| 四层     | 七层       |
| TCP/UDP  | HTTP/HTTPS |
| 负载均衡 | 路由转发   |

------

## Q2：Ingress 为什么不能直接工作

因为：

```text
需要 Ingress Controller
```

------

## Q3：Ingress 能做什么

- 域名路由
- 路径路由
- HTTPS
- 限流
- Rewrite

------

## Q4：Ingress Controller 本质是什么

本质：

```text
反向代理
```

最常见：

```text
NGINX
```

------

# 四十三、总结（核心记忆）

------

## Ingress

本质：

```text
HTTP/HTTPS 路由规则
```

------

## Ingress Controller

本质：

```text
真正转发流量的程序
```

------

## 最常见功能

| 功能      | 示例         |
| --------- | ------------ |
| Host 路由 | api.test.com |
| Path 路由 | /api         |
| HTTPS     | TLS          |
| Rewrite   | URL 重写     |
| 限流      | rate limit   |

------

## 最重要认知

```text
Ingress 不是代理
Ingress Controller 才是代理
```