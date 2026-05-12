# Kubernetes Ingress 详解

在 Kubernetes 中：

```text
Ingress
```

用于：

```text
HTTP/HTTPS 七层流量入口管理
```

它是 Kubernetes 对外暴露 Web 服务最核心组件之一。

------

# 一、为什么需要 Ingress

如果没有 Ingress：

每个 Service 都需要：

```text
NodePort
或者
LoadBalancer
```

问题：

- 端口太多
- 无法统一域名
- HTTPS 难管理
- 无法做路径转发
- 成本高

因此 Kubernetes 引入：

```text
Ingress
```

------

# 二、Ingress 是什么

Ingress 本质：

```text
HTTP/HTTPS 路由规则
```

它定义：

- 域名
- 路径
- 转发规则
- HTTPS
- 负载均衡

------

# 三、Ingress 架构

```text
Browser
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

# 四、Ingress 不是真正转发流量

很多人误解：

```text
Ingress ≠ 负载均衡器
```

Ingress：

```text
只是规则
```

真正处理流量的是：

```text
Ingress Controller
```

------

# 五、Ingress Controller

常见 Controller：

| 类型          | 说明         |
| ------------- | ------------ |
| NGINX Ingress | 最常用       |
| Traefik       | 云原生       |
| HAProxy       | 高性能       |
| Kong          | API 网关     |
| Istio Gateway | Service Mesh |

------

# 六、最常用：NGINX Ingress

常见部署：

```text
Ingress-NGINX
```

本质：

```text
一个 nginx Pod
```

监听：

```text
80/443
```

------

# 七、完整流量流程

```text
用户请求:
http://test.com/api

↓

Ingress Controller

↓

匹配 Ingress Rule

↓

转发到 Service

↓

Service 转发 Pod
```

------

# 八、最简单 Ingress YAML

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress

metadata:
  name: nginx-ingress

spec:
  rules:
    - host: test.com

      http:
        paths:
          - path: /

            pathType: Prefix

            backend:
              service:
                name: nginx-svc

                port:
                  number: 80
```

------

# 九、Ingress YAML 详解

------

# 1. apiVersion

```yaml
apiVersion: networking.k8s.io/v1
```

Ingress 属于：

```text
networking API
```

------

# 2. kind

```yaml
kind: Ingress
```

------

# 3. metadata

```yaml
metadata:
  name: nginx-ingress
```

------

# 4. rules

定义路由规则。

------

# 十、host

域名匹配。

```yaml
host: test.com
```

表示：

```text
只有 test.com 才匹配
```

------

# 十一、path

路径匹配。

```yaml
path: /
```

------

# 十二、pathType

非常重要。

------

# Prefix

前缀匹配。

```yaml
pathType: Prefix
```

------

# Exact

精确匹配。

```yaml
pathType: Exact
```

------

# ImplementationSpecific

由 Controller 自己决定。

------

# 十三、backend

真正转发目标。

------

# Service 名称

```yaml
service:
  name: nginx-svc
```

------

# Service 端口

```yaml
port:
  number: 80
```

------

# 十四、Ingress 必须配合 Service

Ingress：

```text
不能直接转发 Pod
```

只能：

```text
Ingress
    ↓
Service
    ↓
Pod
```

------

# 十五、完整示例

------

# Deployment

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

# Service

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

# Ingress

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress

metadata:
  name: nginx-ingress

spec:
  rules:
    - host: test.com

      http:
        paths:
          - path: /

            pathType: Prefix

            backend:
              service:
                name: nginx-svc

                port:
                  number: 80
```

------

# 十六、访问流程

------

# 1. 浏览器访问

```text
http://test.com
```

------

# 2. DNS 解析

解析到：

```text
Ingress Controller IP
```

------

# 3. Controller 匹配规则

匹配：

```text
host + path
```

------

# 4. 转发到 Service

------

# 5. Service 转发 Pod

------

# 十七、路径路由（非常常用）

------

# 示例

```yaml
rules:
  - host: test.com

    http:
      paths:
        - path: /api

          pathType: Prefix

          backend:
            service:
              name: api-svc

              port:
                number: 80

        - path: /web

          pathType: Prefix

          backend:
            service:
              name: web-svc

              port:
                number: 80
```

------

# 访问效果

| URL  | 转发    |
| ---- | ------- |
| /api | api-svc |
| /web | web-svc |

------

# 十八、多域名路由

------

# 示例

```yaml
rules:
  - host: api.test.com
    ...

  - host: admin.test.com
    ...
```

------

# 十九、HTTPS（重点）

Ingress 最大价值之一：

```text
统一 HTTPS
```

------

# TLS 配置

```yaml
spec:
  tls:
    - hosts:
        - test.com

      secretName: tls-secret
```

------

# 二十、TLS Secret

证书存储在：

```text
Secret
```

------

# 创建 TLS Secret

```bash
kubectl create secret tls tls-secret \
  --cert=tls.crt \
  --key=tls.key
```

------

# 二十一、Ingress Class

Kubernetes 可能有多个 Controller。

因此需要：

```text
IngressClass
```

指定。

------

# 示例

```yaml
spec:
  ingressClassName: nginx
```

------

# 二十二、查看 Ingress

------

# 查看

```bash
kubectl get ingress
```

------

# 查看详细信息

```bash
kubectl describe ingress nginx-ingress
```

------

# 查看 YAML

```bash
kubectl get ingress nginx-ingress -o yaml
```

------

# 二十三、安装 NGINX Ingress

常见安装：

------

# Helm

```bash
helm install ingress-nginx ingress-nginx/ingress-nginx
```

------

# 官方 YAML

```bash
kubectl apply -f deploy.yaml
```

------

# 二十四、Ingress 常见问题

------

# 1. Ingress 不生效

最常见：

```text
没安装 Ingress Controller
```

------

# 2. 404

说明：

```text
规则未匹配
```

检查：

- host
- path
- ingressClass

------

# 3. Service 无 endpoints

检查：

```bash
kubectl get ep
```

------

# 4. HTTPS 失败

检查：

- Secret
- 域名
- 证书

------

# 二十五、Ingress 与 Service 区别

| Ingress    | Service  |
| ---------- | -------- |
| 七层       | 四层     |
| HTTP/HTTPS | TCP/UDP  |
| 域名路由   | Pod 转发 |
| SSL        | 无       |

------

# 二十六、Ingress 与 Gateway API

现代 Kubernetes：

```text
Ingress 正逐渐被 Gateway API 替代
```

------

# Gateway API 更强：

- 更标准
- 更灵活
- 更适合云原生

------

# 二十七、Ingress 常用注解

NGINX Ingress 常用：

------

# URL Rewrite

```yaml
annotations:
  nginx.ingress.kubernetes.io/rewrite-target: /
```

------

# 限流

```yaml
annotations:
  nginx.ingress.kubernetes.io/limit-rps: "10"
```

------

# Body 大小

```yaml
annotations:
  nginx.ingress.kubernetes.io/proxy-body-size: 50m
```

------

# 二十八、生产最佳实践

------

# 1. 使用 HTTPS

必须。

------

# 2. 使用 ClusterIP Service

不要：

```text
Ingress + NodePort
```

直接暴露业务。

------

# 3. 配置 readinessProbe

避免：

```text
未启动完成接流量
```

------

# 4. 使用 cert-manager

自动签发证书。

------

# 5. 配置限流

防止：

- CC
- 爬虫
- 恶意请求

------

# 二十九、生产架构

典型：

```text
Internet
    ↓
SLB/ELB
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

# 三十、Ingress Controller 本质

本质：

```text
监听 Kubernetes API
```

发现：

```text
Ingress 资源变化
```

动态生成：

```text
Nginx 配置
```

然后：

```text
reload nginx
```

------

# 三十一、面试高频问题

------

# 1. Ingress 为什么存在

统一：

```text
域名
HTTPS
七层路由
```

------

# 2. Ingress 与 Service 区别

| Ingress | Service |
| ------- | ------- |
| 七层    | 四层    |

------

# 3. Ingress 为什么必须 Controller

因为：

```text
Ingress 只是规则
```

------

# 4. Ingress 如何实现 HTTPS

通过：

```text
TLS Secret
```

------

# 5. Ingress 如何实现路径转发

通过：

```text
host + path
```

匹配。

------

# 三十二、建议继续学习

推荐继续：

1. CoreDNS
2. kube-proxy
3. CNI 网络
4. StatefulSet
5. ConfigMap
6. Secret
7. PV/PVC
8. Helm
9. HPA
10. Gateway API