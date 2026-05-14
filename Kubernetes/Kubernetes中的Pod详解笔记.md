# Kubernetes Pod 详解笔记

在 Kubernetes 中，Pod 是最核心、最基础的对象。

你可以理解为：

> Pod = Kubernetes 中最小的运行单元

------

# 一、Pod 是什么

Pod（豆荚）本质上是：

```text
一个或多个容器
+ 共享网络
+ 共享存储
+ 共享生命周期
```

------

# 二、为什么 Kubernetes 不直接管理容器

因为真实业务中：

- 一个应用可能有多个容器
- 容器之间需要通信
- 需要共享文件
- 需要统一调度

所以 Kubernetes 引入：

```text
Pod
```

作为最小调度单位。

------

# 三、Pod 的核心特点

------

## 1. 一个 Pod 可以有多个容器

例如：

```text
nginx 容器
+
日志采集容器
```

------

## 2. Pod 内共享网络

Pod 内所有容器：

- 共享 IP
- 共享端口空间
- localhost 可互通

例如：

```text
容器A:
localhost:8080

容器B:
curl localhost:8080
```

可以直接访问。

------

## 3. Pod 内共享存储

通过 Volume：

```text
volume
```

多个容器共享文件。

------

# 四、Pod 架构图

```text
+-----------------------------------+
|              Pod                  |
|                                   |
|   +------------+                  |
|   | nginx      |                  |
|   | 80         |                  |
|   +------------+                  |
|                                   |
|   +------------+                  |
|   | sidecar    |                  |
|   | logs       |                  |
|   +------------+                  |
|                                   |
|   Shared Network Namespace        |
|   Shared Volume                   |
+-----------------------------------+
```

------

# 五、最简单 Pod YAML

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

# 六、Pod YAML 全面解析

------

## 1. apiVersion

```yaml
apiVersion: v1
```

Pod 属于核心 API。

------

## 2. kind

```yaml
kind: Pod
```

表示资源类型。

------

## 3. metadata

```yaml
metadata:
  name: nginx-pod
```

------

## metadata 常见字段

------

### name

Pod 名称。

```yaml
name: nginx-pod
```

------

### namespace

命名空间。

```yaml
namespace: default
```

------

### labels

标签。

```yaml
labels:
  app: nginx
  env: prod
```

用途：

- Service 选择 Pod
- Deployment 管理 Pod
- 分类资源

------

### annotations

注解。

```yaml
annotations:
  description: "生产 nginx"
```

一般用于：

- 说明信息
- Prometheus 采集
- Istio 配置

------

# 七、spec 详解

真正的核心部分。

------

# 八、containers

------

## 基础写法

```yaml
spec:
  containers:
    - name: nginx
      image: nginx:1.25
```

------

## 为什么是数组

因为：

```text
一个 Pod 可以运行多个容器
```

------

# 九、container 核心字段

------

## 1. name

容器名称。

```yaml
name: nginx
```

必须唯一。

------

## 2. image

容器镜像。

```yaml
image: nginx:1.25
```

推荐：

```yaml
image: nginx:1.25.5
```

不要使用：

```yaml
latest
```

------

## 3. imagePullPolicy

镜像拉取策略。

------

## Always

总是拉取。

```yaml
imagePullPolicy: Always
```

------

## IfNotPresent

默认策略。

本地不存在才拉取。

```yaml
imagePullPolicy: IfNotPresent
```

------

## Never

永不拉取。

```yaml
imagePullPolicy: Never
```

用于离线环境。

------

# 十、ports

容器端口。

```yaml
ports:
  - containerPort: 80
```

注意：

```text
仅仅是声明
不会真正暴露端口
```

真正对外访问：

```text
Service
```

------

# 十一、env 环境变量

------

## 普通变量

```yaml
env:
  - name: APP_ENV
    value: prod
```

------

## 引用 Secret

```yaml
env:
  - name: MYSQL_PASSWORD
    valueFrom:
      secretKeyRef:
        name: mysql-secret
        key: password
```

------

## 引用 ConfigMap

```yaml
env:
  - name: LOG_LEVEL
    valueFrom:
      configMapKeyRef:
        name: app-config
        key: level
```

------

# 十二、command 与 args

------

## Docker 对应关系

| Kubernetes | Docker     |
| ---------- | ---------- |
| command    | ENTRYPOINT |
| args       | CMD        |

------

## 示例

```yaml
command: ["sleep"]
args: ["3600"]
```

实际执行：

```bash
sleep 3600
```

------

# 十三、resources 资源限制

生产环境必须配置。

------

## 完整写法

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

## requests

调度时保证资源。

------

## limits

最大允许使用。

超出：

- CPU 被限速
- Memory OOMKilled

------

## CPU 单位

------

## 1000m = 1核

```yaml
500m
```

表示：

```text
0.5 CPU
```

------

## Memory 单位

| 单位 | 含义           |
| ---- | -------------- |
| Mi   | 1024*1024      |
| Gi   | 1024*1024*1024 |

------

# 十四、Volume 存储

------

## 为什么需要 Volume

容器重启后：

```text
容器文件会丢失
```

因此需要持久化。

------

# 十五、volumeMounts

挂载目录。

```yaml
volumeMounts:
  - name: data
    mountPath: /data
```

------

# 十六、volumes

定义 Volume。

------

## emptyDir

Pod 删除即销毁。

```yaml
volumes:
  - name: cache
    emptyDir: {}
```

------

## hostPath

宿主机目录。

```yaml
volumes:
  - name: logs
    hostPath:
      path: /data/logs
```

危险：

```text
强依赖节点
```

生产少用。

------

## PVC

持久卷。

```yaml
volumes:
  - name: mysql-data

    persistentVolumeClaim:
      claimName: mysql-pvc
```

------

# 十七、探针（Probe）

[探针详解](./Kubernetes中Pod探针（Probe）详解.md)

# 十九、重启策略 restartPolicy

------

## Always

默认。

```yaml
restartPolicy: Always
```

------

## OnFailure

失败才重启。

------

## Never

永不重启。

------

# 二十、Pod 生命周期

------

## 阶段流程

```text
Pending
↓
ContainerCreating
↓
Running
↓
Succeeded / Failed
```

------

## Pod 状态详解

------

### Pending

等待调度。

原因：

- 无节点
- 资源不足
- PVC 未绑定

------

### Running

运行中。

------

### Succeeded

成功结束。

常见于 Job。

------

### Failed

运行失败。

------

### CrashLoopBackOff

反复崩溃。

最常见。

------

# 二十一、Pod 常见问题

------

## 1. ImagePullBackOff

镜像拉取失败。

检查：

- 镜像名
- 仓库权限
- 网络

------

## 2. CrashLoopBackOff

容器不断重启。

检查：

```bash
kubectl logs
```

------

## 3. OOMKilled

内存超限。

增加：

```yaml
limits.memory
```

------

# 二十二、Init Container

初始化容器。

主容器启动前执行。

------

## 示例

```yaml
initContainers:
  - name: init-db
    image: busybox

    command:
      - sh
      - -c
      - "echo init"
```

------

# 二十三、Sidecar 容器

辅助容器。

典型：

- 日志采集
- envoy
- 监控

------

# 二十四、多容器 Pod 示例

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: multi-container

spec:
  containers:
    - name: nginx
      image: nginx

    - name: sidecar
      image: busybox

      command:
        - sh
        - -c
        - tail -f /dev/null
```

------

# 二十五、Pod 网络原理

每个 Pod：

```text
拥有独立 IP
```

不是容器 IP。

------

## Pod 通信

------

### 同 Pod

```text
localhost
```

------

### 不同 Pod

```text
Pod IP
Service
```

------

# 二十六、为什么生产不用裸 Pod

因为：

```text
Pod 删除后不会自动恢复
```

------

## 正确方式

使用：

```text
Deployment
StatefulSet
DaemonSet
```

管理 Pod。

------

# 二十七、查看 Pod 命令

------

## 查看 Pod

```bash
kubectl get pods
```

------

## 查看详细信息

```bash
kubectl describe pod nginx
```

------

## 查看 YAML

```bash
kubectl get pod nginx -o yaml
```

------

## 查看日志

```bash
kubectl logs nginx
```

------

## 进入容器

```bash
kubectl exec -it nginx -- bash
```

------

# 二十八、完整生产级 Pod YAML

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-prod
  labels:
    app: nginx
    env: prod
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
      volumeMounts:
        - name: html
          mountPath: /usr/share/nginx/html
  volumes:
    - name: html
      emptyDir: {}
```

------

# 二十九、面试高频 Pod 知识

重点掌握：

------

## Pod 为什么存在

------

## Pod 共享什么

- Network Namespace
- IPC
- Volume

------

## Probe 区别

| Probe     | 作用         |
| --------- | ------------ |
| liveness  | 是否活着     |
| readiness | 是否接流量   |
| startup   | 是否启动完成 |

------

## requests 与 limits

------

## Pod 为什么不直接使用

因为不会自愈。

------

# 三十、学习建议

下一步建议学习：

1. Deployment
2. Service
3. Ingress
4. Volume/PVC
5. StatefulSet
6. CNI 网络
7. 调度器
8. kube-proxy
9. CoreDNS
10. Helm