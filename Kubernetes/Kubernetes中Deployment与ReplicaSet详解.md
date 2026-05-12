# Kubernetes Deployment 与 ReplicaSet 详解

在 Kubernetes 中：

```text
Deployment
    ↓ 管理
ReplicaSet
    ↓ 管理
Pod
```

它们是 Kubernetes 最核心的控制器。

------

# 一、为什么需要 Deployment

如果直接创建 Pod：

```text
Pod 删除后不会自动恢复
```

例如：

```bash
kubectl delete pod nginx
```

Pod 就没了。

因此 Kubernetes 引入：

```text
控制器（Controller）
```

用于：

- 自动恢复
- 副本管理
- 滚动更新
- 扩缩容

------

# 二、Deployment 是什么

Deployment 是：

```text
无状态应用控制器
```

用于：

- 管理 Pod
- 管理 ReplicaSet
- 滚动升级
- 回滚
- 扩缩容

------

# 三、ReplicaSet 是什么

ReplicaSet（RS）作用：

```text
保证指定数量 Pod 始终运行
```

例如：

```yaml
replicas: 3
```

如果少一个：

```text
自动补一个
```

如果多一个：

```text
自动删一个
```

------

# 四、三者关系

```text
Deployment
    ↓
ReplicaSet
    ↓
Pod
```

------

# 五、工作原理图

```text
Deployment
  │
  ├── ReplicaSet(v1)
  │       ├── Pod
  │       ├── Pod
  │       └── Pod
  │
  └── ReplicaSet(v2)
          ├── Pod
          ├── Pod
          └── Pod
```

------

# 六、为什么需要 ReplicaSet

因为 Deployment：

```text
不直接管理 Pod
```

而是：

```text
通过 ReplicaSet 管理
```

原因：

```text
方便版本切换
```

这是：

```text
滚动更新的核心
```

------

# 七、最简单 Deployment YAML

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

# 八、Deployment YAML 详解

------

# 1. apiVersion

```yaml
apiVersion: apps/v1
```

Deployment 属于：

```text
apps API 组
```

------

# 2. kind

```yaml
kind: Deployment
```

资源类型。

------

# 3. metadata

```yaml
metadata:
  name: nginx-deploy
```

Deployment 名称。

------

# 九、spec 核心字段

------

# 1. replicas

副本数。

```yaml
replicas: 3
```

表示：

```text
始终保持 3 个 Pod
```

------

# 2. selector

标签选择器。

```yaml
selector:
  matchLabels:
    app: nginx
```

作用：

```text
Deployment 通过标签找到 Pod
```

------

# 3. template

Pod 模板。

```yaml
template:
```

真正创建 Pod 的地方。

------

# 十、template 详解

------

# template.metadata

```yaml
template:
  metadata:
    labels:
      app: nginx
```

注意：

```text
必须与 selector 匹配
```

否则：

```text
Deployment 无法管理 Pod
```

------

# template.spec

真正的 Pod 配置。

```yaml
spec:
  containers:
```

和 Pod YAML 一样。

------

# 十一、Deployment 创建流程

执行：

```bash
kubectl apply -f deploy.yaml
```

流程：

```text
Deployment
    ↓ 创建
ReplicaSet
    ↓ 创建
Pod
```

------

# 十二、查看 Deployment

------

## 查看 Deployment

```bash
kubectl get deploy
```

------

## 查看 ReplicaSet

```bash
kubectl get rs
```

------

## 查看 Pod

```bash
kubectl get pods
```

------

# 十三、ReplicaSet 工作机制

ReplicaSet 会不断检查：

```text
当前 Pod 数量
```

------

# 示例

期望：

```yaml
replicas: 3
```

实际：

```text
2
```

ReplicaSet：

```text
自动创建 1 个
```

------

# 删除 Pod 测试

```bash
kubectl delete pod xxx
```

会发现：

```text
立刻创建新 Pod
```

因为：

```text
ReplicaSet 自愈
```

------

# 十四、Deployment 滚动更新

最核心功能。

------

# 修改镜像

```yaml
image: nginx:1.26
```

然后：

```bash
kubectl apply -f deploy.yaml
```

------

# Kubernetes 会：

```text
创建新的 ReplicaSet
↓
逐步创建新 Pod
↓
逐步删除旧 Pod
```

这就是：

```text
Rolling Update
```

------

# 十五、滚动更新过程

------

## 旧版本

```text
RS-v1
```

运行：

```text
nginx:1.25
```

------

## 更新后

创建：

```text
RS-v2
```

运行：

```text
nginx:1.26
```

------

# 整个过程

```text
RS-v1: 3 Pod
RS-v2: 0 Pod

↓

RS-v1: 2 Pod
RS-v2: 1 Pod

↓

RS-v1: 1 Pod
RS-v2: 2 Pod

↓

RS-v1: 0 Pod
RS-v2: 3 Pod
```

------

# 十六、为什么 Deployment 能回滚

因为：

```text
旧 ReplicaSet 不会立刻删除
```

Deployment 保留历史版本。

------

# 查看历史版本

```bash
kubectl rollout history deployment nginx-deploy
```

------

# 十七、回滚 Deployment

------

## 回滚上一版本

```bash
kubectl rollout undo deployment nginx-deploy
```

------

## 回滚指定版本

```bash
kubectl rollout undo deployment nginx-deploy --to-revision=2
```

------

# 十八、Deployment 更新策略

------

# strategy

```yaml
strategy:
  type: RollingUpdate
```

------

# 两种策略

| 类型          | 说明     |
| ------------- | -------- |
| RollingUpdate | 滚动更新 |
| Recreate      | 先删后建 |

------

# RollingUpdate 参数

------

## maxSurge

最大超出副本数。

```yaml
maxSurge: 1
```

------

## maxUnavailable

最大不可用数量。

```yaml
maxUnavailable: 1
```

------

# 示例

```yaml
strategy:
  type: RollingUpdate

  rollingUpdate:
    maxSurge: 1
    maxUnavailable: 1
```

------

# 十九、扩缩容

------

# 修改 replicas

```yaml
replicas: 5
```

------

# 或命令行

```bash
kubectl scale deploy nginx-deploy --replicas=5
```

------

# 缩容

```bash
kubectl scale deploy nginx-deploy --replicas=1
```

------

# 二十、Deployment 状态

------

# 查看状态

```bash
kubectl rollout status deploy nginx-deploy
```

------

# 常见状态

| 状态        | 含义   |
| ----------- | ------ |
| Progressing | 更新中 |
| Complete    | 完成   |
| Failed      | 失败   |

------

# 二十一、ReplicaSet 详解

通常：

```text
不直接使用 ReplicaSet
```

因为：

```text
Deployment 已经封装
```

------

# ReplicaSet YAML

```yaml
apiVersion: apps/v1
kind: ReplicaSet

metadata:
  name: nginx-rs

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
```

------

# ReplicaSet 的缺点

只有：

```text
副本控制
```

没有：

- 回滚
- 滚动升级
- 历史版本

所以生产：

```text
使用 Deployment
```

------

# 二十二、Deployment 常用命令

------

# 创建

```bash
kubectl apply -f deploy.yaml
```

------

# 查看

```bash
kubectl get deploy
```

------

# 查看详细信息

```bash
kubectl describe deploy nginx-deploy
```

------

# 查看 YAML

```bash
kubectl get deploy nginx-deploy -o yaml
```

------

# 删除

```bash
kubectl delete deploy nginx-deploy
```

------

# 二十三、生产最佳实践

------

# 1. 必须加 readinessProbe

避免未启动完成接收流量。

------

# 2. 必须加 resources

防止资源抢占。

------

# 3. 不要使用 latest

推荐：

```yaml
image: nginx:1.25.5
```

------

# 4. replicas 至少 2

避免单点。

------

# 5. labels 规范化

```yaml
labels:
  app: nginx
  version: v1
  env: prod
```

------

# 二十四、完整生产级 Deployment

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: nginx-prod

spec:
  replicas: 3

  strategy:
    type: RollingUpdate

    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1

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

          ports:
            - containerPort: 80

          resources:
            requests:
              cpu: "100m"
              memory: "128Mi"

            limits:
              cpu: "500m"
              memory: "512Mi"

          readinessProbe:
            httpGet:
              path: /
              port: 80

            initialDelaySeconds: 5

          livenessProbe:
            httpGet:
              path: /
              port: 80

            initialDelaySeconds: 10
```

------

# 二十五、面试高频问题

------

# 1. Deployment 和 ReplicaSet 区别

| Deployment   | ReplicaSet |
| ------------ | ---------- |
| 管理 RS      | 管理 Pod   |
| 支持回滚     | 不支持     |
| 支持滚动更新 | 不支持     |
| 生产常用     | 很少直接用 |

------

# 2. 为什么 Deployment 不直接管理 Pod

为了：

```text
版本切换
滚动更新
回滚
```

------

# 3. Deployment 更新原理

```text
新建 ReplicaSet
逐步替换 Pod
```

------

# 4. 删除 Pod 为什么会恢复

因为：

```text
ReplicaSet 自愈
```

------

# 5. selector 为什么重要

Deployment 通过 selector 找到 Pod。

------

# 二十六、Deployment 与 StatefulSet 区别

| Deployment | StatefulSet |
| ---------- | ----------- |
| 无状态     | 有状态      |
| Pod 名随机 | Pod 名固定  |
| 适合 Web   | 适合 DB     |
| 可随意扩缩 | 有顺序      |

------

# 二十七、建议下一步学习

推荐继续学习：

1. Service
2. Ingress
3. ConfigMap
4. Secret
5. Volume/PVC
6. StatefulSet
7. HPA
8. DaemonSet
9. CNI 网络
10. Helm