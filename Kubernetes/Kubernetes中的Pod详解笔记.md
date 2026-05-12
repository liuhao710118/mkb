# Kubernetes YAML 详解笔记

## 一、什么是 Kubernetes YAML

在 Kubernetes 中，几乎所有资源都通过 **YAML 文件**定义。

YAML 的作用：

- 描述资源对象
- 声明式配置
- 便于版本管理
- 可重复部署

常见命令：

```bash
kubectl apply -f nginx.yaml
kubectl delete -f nginx.yaml
kubectl get pods
```

------

# 二、YAML 基础语法

## 1. 键值格式

```yaml
name: nginx
image: nginx:1.25
```

------

## 2. 缩进

YAML 使用空格表示层级。

⚠️ 不能使用 TAB。

```yaml
spec:
  containers:
    - name: nginx
      image: nginx
```

------

## 3. 数组

使用 `-`

```yaml
containers:
  - name: nginx
    image: nginx

  - name: redis
    image: redis
```

------

## 4. 注释

```yaml
# 这是注释
```

------

# 三、Kubernetes YAML 四大核心字段

几乎所有资源都有：

```yaml
apiVersion:
kind:
metadata:
spec:
```

------

# 四、核心字段详解

------

## 1. apiVersion

表示资源使用哪个 API 版本。

示例：

```yaml
apiVersion: v1
```

或者：

```yaml
apiVersion: apps/v1
```

常见版本：

| 资源        | apiVersion           |
| ----------- | -------------------- |
| Pod         | v1                   |
| Service     | v1                   |
| Deployment  | apps/v1              |
| DaemonSet   | apps/v1              |
| StatefulSet | apps/v1              |
| Ingress     | networking.k8s.io/v1 |

------

## 2. kind

表示资源类型。

```yaml
kind: Pod
```

常见类型：

| 类型        | 作用         |
| ----------- | ------------ |
| Pod         | 最小运行单元 |
| Deployment  | 无状态部署   |
| StatefulSet | 有状态应用   |
| DaemonSet   | 每节点运行   |
| Service     | 服务暴露     |
| ConfigMap   | 配置         |
| Secret      | 密钥         |
| Ingress     | 七层代理     |

------

## 3. metadata

资源元数据。

```yaml
metadata:
  name: nginx
  namespace: default
  labels:
    app: nginx
```

------

### metadata 常见字段

| 字段        | 作用     |
| ----------- | -------- |
| name        | 资源名称 |
| namespace   | 命名空间 |
| labels      | 标签     |
| annotations | 注解     |

------

## labels

标签用于：

- 资源分类
- Service 选择 Pod
- Deployment 管理 Pod

```yaml
labels:
  app: nginx
  env: prod
```

------

## annotations

用于存储说明信息。

```yaml
annotations:
  description: "生产环境 nginx"
```

------

# 五、spec 详解

`spec` 是最重要部分。

真正描述资源行为。

------

# 六、Pod YAML 详解

------

## 最简单 Pod

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: nginx-pod

spec:
  containers:
    - name: nginx
      image: nginx:1.25
```

------

# 七、containers 详解

------

## name

容器名称。

```yaml
name: nginx
```

------

## image

镜像。

```yaml
image: nginx:1.25
```

------

## imagePullPolicy

镜像拉取策略。

```yaml
imagePullPolicy: IfNotPresent
```

------

### 三种策略

| 值           | 含义           |
| ------------ | -------------- |
| Always       | 总是拉取       |
| IfNotPresent | 本地没有才拉取 |
| Never        | 永不拉取       |

------

## ports

开放端口。

```yaml
ports:
  - containerPort: 80
```

------

## env

环境变量。

```yaml
env:
  - name: MYSQL_HOST
    value: mysql
```

------

## command

覆盖 ENTRYPOINT。

```yaml
command: ["sleep"]
args: ["3600"]
```

------

## resources

资源限制。

```yaml
resources:
  requests:
    cpu: "100m"
    memory: "128Mi"

  limits:
    cpu: "500m"
    memory: "512Mi"
```

------

### CPU 单位

| 值    | 含义  |
| ----- | ----- |
| 1000m | 1核   |
| 500m  | 0.5核 |

------

### Memory 单位

| 值   | 含义     |
| ---- | -------- |
| Mi   | Mebibyte |
| Gi   | Gibibyte |

------

# 八、Pod 生命周期探针

------

## livenessProbe

存活探针。

失败会重启容器。

```yaml
livenessProbe:
  httpGet:
    path: /
    port: 80

  initialDelaySeconds: 10
  periodSeconds: 5
```

------

## readinessProbe

就绪探针。

失败不会重启。

但不会加入 Service。

```yaml
readinessProbe:
  tcpSocket:
    port: 80
```

------

## startupProbe

启动探针。

适合慢启动应用。

```yaml
startupProbe:
  httpGet:
    path: /
    port: 80
```

------

# 九、Volume 存储

------

## emptyDir

临时目录。

```yaml
volumes:
  - name: cache
    emptyDir: {}
```

------

## hostPath

挂载宿主机目录。

```yaml
volumes:
  - name: logs
    hostPath:
      path: /data/logs
```

------

## PVC

持久卷。

```yaml
volumes:
  - name: data
    persistentVolumeClaim:
      claimName: mysql-pvc
```

------

# 十、volumeMounts

```yaml
volumeMounts:
  - name: data
    mountPath: /var/lib/mysql
```

------

# 十一、完整 Pod 示例

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: nginx-demo
  labels:
    app: nginx

spec:
  containers:
    - name: nginx
      image: nginx:1.25

      ports:
        - containerPort: 80

      env:
        - name: ENV
          value: prod

      resources:
        requests:
          cpu: "100m"
          memory: "128Mi"

        limits:
          cpu: "500m"
          memory: "512Mi"

      volumeMounts:
        - name: html
          mountPath: /usr/share/nginx/html

  volumes:
    - name: html
      emptyDir: {}
```

------

# 十二、Deployment 详解

Deployment 用于：

- 副本管理
- 滚动更新
- 自动恢复

------

## Deployment YAML

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: nginx-deploy

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
          image: nginx:1.25
```

------

# 十三、Deployment 核心字段

------

## replicas

副本数量。

```yaml
replicas: 3
```

------

## selector

选择 Pod。

```yaml
selector:
  matchLabels:
    app: nginx
```

------

## template

Pod 模板。

```yaml
template:
```

Deployment 最终会创建 Pod。

------

# 十四、Service 详解

Service 用于暴露服务。

------

## ClusterIP

集群内部访问。

```yaml
apiVersion: v1
kind: Service

metadata:
  name: nginx-svc

spec:
  type: ClusterIP

  selector:
    app: nginx

  ports:
    - port: 80
      targetPort: 80
```

------

## NodePort

对外暴露。

```yaml
type: NodePort
```

访问：

```bash
<NodeIP>:30080
```

------

## LoadBalancer

云厂商负载均衡。

```yaml
type: LoadBalancer
```

------

# 十五、ConfigMap

配置文件。

------

## 创建 ConfigMap

```yaml
apiVersion: v1
kind: ConfigMap

metadata:
  name: app-config

data:
  APP_ENV: prod
  LOG_LEVEL: info
```

------

# 十六、Secret

存储敏感信息。

```yaml
apiVersion: v1
kind: Secret

metadata:
  name: db-secret

type: Opaque

data:
  password: MTIzNDU2
```

base64 编码：

```bash
echo -n "123456" | base64
```

------

# 十七、Ingress

Ingress 提供：

- 域名访问
- HTTPS
- 七层路由

------

## Ingress 示例

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

# 十八、多资源 YAML

使用 `---` 分隔。

```yaml
apiVersion: v1
kind: Service
...

---

apiVersion: apps/v1
kind: Deployment
...
```

------

# 十九、常用 kubectl 命令

------

## 查看资源

```bash
kubectl get pods
kubectl get svc
kubectl get deploy
```

------

## 查看详细信息

```bash
kubectl describe pod nginx
```

------

## 查看日志

```bash
kubectl logs nginx
```

------

## 进入容器

```bash
kubectl exec -it nginx -- /bin/bash
```

------

# 二十、生产最佳实践

------

## 必加 resources

避免资源抢占。

------

## 必加 readinessProbe

防止未启动完成流量进入。

------

## 使用 Deployment 而不是 Pod

Pod 删除后不会自动恢复。

------

## labels 规范化

推荐：

```yaml
labels:
  app: nginx
  version: v1
  env: prod
```

------

## 镜像不要使用 latest

推荐：

```yaml
image: nginx:1.25.5
```

------

# 二十一、学习路线

推荐顺序：

1. Pod
2. Deployment
3. Service
4. ConfigMap
5. Secret
6. Volume
7. Ingress
8. StatefulSet
9. Helm
10. Operator

------

# 二十二、排错思路

------

## Pod 起不来

```bash
kubectl describe pod
kubectl logs
```

------

## Service 不通

检查：

```bash
selector
labels
targetPort
```

------

## 镜像拉取失败

```bash
ImagePullBackOff
```

检查：

- 镜像名
- 网络
- 镜像仓库认证

------

# 二十三、建议重点掌握

面试高频：

- Deployment 滚动更新
- Service 四层转发
- Ingress 七层代理
- ConfigMap/Secret
- 探针
- requests/limits
- StatefulSet
- PVC/PV

------

# 二十四、一个完整 Web 应用 YAML

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: web

spec:
  replicas: 2

  selector:
    matchLabels:
      app: web

  template:
    metadata:
      labels:
        app: web

    spec:
      containers:
        - name: nginx
          image: nginx:1.25

          ports:
            - containerPort: 80

---
apiVersion: v1
kind: Service

metadata:
  name: web-svc

spec:
  selector:
    app: web

  ports:
    - port: 80
      targetPort: 80

  type: NodePort
```