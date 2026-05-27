# Kubernetes Pod 探针（Probe）详解

Kubernetes 中的 **Probe（探针）** 是 kubelet 用来检测容器健康状态的一种机制。
它可以帮助 Kubernetes 自动完成：

- 判断容器是否启动完成
- 判断容器是否健康
- 判断容器是否可以接收流量
- 自动重启异常容器
- 避免未准备好的 Pod 接收请求

------

# 一、Probe 分类

K8s 共有三种探针：

| 探针类型       | 作用     | 失败后结果                 |
| -------------- | -------- | -------------------------- |
| livenessProbe  | 存活探针 | 重启容器                   |
| readinessProbe | 就绪探针 | 从 Service Endpoint 中移除 |
| startupProbe   | 启动探针 | 启动失败则重启容器         |

------

# 二、livenessProbe（存活探针）

## 1. 作用

用于判断：

> “容器是不是还活着”

如果探针失败达到阈值：

- kubelet 会认为容器已经异常
- 自动重启容器

------

## 2. 典型场景

例如：

- Java 死锁
- 线程卡死
- 内存泄漏
- 程序假死
- 服务不响应

虽然进程还在，但已经无法工作。

------

## 3. 工作流程

```text
探针失败
    ↓
达到 failureThreshold
    ↓
kubelet 重启容器
```

------

## 4. 示例

```yaml
livenessProbe:
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 30
  periodSeconds: 10
```

含义：

| 参数                | 说明                       |
| ------------------- | -------------------------- |
| path                | 检测路径                   |
| port                | 检测端口                   |
| initialDelaySeconds | 容器启动后等待多久开始检测 |
| periodSeconds       | 每隔多久检测一次           |

------

## 5. 运行机制

```text
HTTP GET http://PodIP:8080/health
```

如果：

- 返回 200~399
  → success

否则：

- failure

------

# 三、readinessProbe（就绪探针）

## 1. 作用

用于判断：

> “Pod 是否已经准备好接收流量”

------

## 2. 与 liveness 最大区别

| 探针      | 失败后             |
| --------- | ------------------ |
| liveness  | 重启容器           |
| readiness | 不重启，只摘除流量 |

------

## 3. 工作流程

```text
readiness 失败
    ↓
Pod 从 Service Endpoints 删除
    ↓
不会接收请求
```

恢复后：

```text
readiness success
    ↓
重新加入 Service
```

------

## 4. 常见场景

### SpringBoot 启动很慢

虽然进程已经启动：

```text
java process 已存在
```

但：

- 缓存没加载
- 数据库没连接
- MQ 没连接
- 配置没加载

这时不应该接收流量。

------

## 5. 示例

```yaml
readinessProbe:
  httpGet:
    path: /ready
    port: 8080
  initialDelaySeconds: 10
  periodSeconds: 5
```

------

# 四、startupProbe（启动探针）

## 1. 为什么需要 startupProbe

很多服务启动很慢：

例如：

- Java
- Elasticsearch
- Jenkins
- Kafka

如果：

```text
livenessProbe 太早执行
```

会导致：

```text
容器还没启动完成
↓
liveness 失败
↓
反复重启
↓
CrashLoopBackOff
```

------

## 2. startupProbe 作用

告诉 K8s：

> “启动期间不要做 liveness 检测”

只有 startupProbe 成功后：

- livenessProbe 才会开始工作

------

## 3. 示例

```yaml
startupProbe:
  httpGet:
    path: /startup
    port: 8080
  failureThreshold: 30
  periodSeconds: 10
```

含义：

```text
最多允许失败：
30 × 10 = 300秒
```

即：

给应用 5 分钟启动时间。

------

# 五、三种探针关系

------

## 1. 组合逻辑

```text
startupProbe 成功
        ↓
开始 livenessProbe
        ↓
开始 readinessProbe
```

------

## 2. 执行顺序

| 阶段       | 探针           |
| ---------- | -------------- |
| 启动阶段   | startupProbe   |
| 运行阶段   | livenessProbe  |
| 接流量阶段 | readinessProbe |

------

# 六、Probe 支持的检测方式

K8s 支持三种探针方式：

| 类型      | 说明           |
| --------- | -------------- |
| httpGet   | HTTP 请求      |
| tcpSocket | TCP 端口       |
| exec      | 容器内执行命令 |

------

# 七、HTTP Probe

------

> **响应状态码 200–399**：成功 其它状态码 / 超时 / 连接拒绝：失败

## 示例

```yaml
livenessProbe:
  httpGet:
    path: /healthz
    port: 8080
    scheme: HTTP
```

------

## 支持 HTTPS

```yaml
scheme: HTTPS
```

------

# 八、TCP Probe

## 原理

检测端口是否能建立 TCP 连接。

------

## 示例

```yaml
readinessProbe:
  tcpSocket:
    port: 3306
```

------

## 适用场景

适合：

- MySQL
- Redis
- Kafka

这种没有 HTTP 接口的服务。

------

# 九、Exec Probe

## 原理

在容器内部执行命令。

返回值：

```text
exit code = 0
```

代表成功。

------

## 示例

```yaml
livenessProbe:
  exec:
    command:
    - cat
    - /tmp/healthy
```

------

## 常见用法

### 检查进程

```yaml
exec:
  command:
  - sh
  - -c
  - ps -ef | grep java
```

------

# 十、核心参数详解

------

## 1. initialDelaySeconds

启动后延迟多久开始探测。

```yaml
initialDelaySeconds: 30
```

------

## 2. periodSeconds

多久执行一次。

```yaml
periodSeconds: 10
```

------

## 3. timeoutSeconds

超时时间。

```yaml
timeoutSeconds: 3
```

超过 3 秒没响应：

```text
probe failure
```

------

## 4. successThreshold

成功几次才算成功。

默认：

```text
1
```

readinessProbe 常用。

------

## 5. failureThreshold

失败几次才算失败。

```yaml
failureThreshold: 3
```

含义：

```text
连续失败3次
才认为失败
```

------

# 十一、完整 YAML 示例

------

## SpringBoot 应用最佳实践

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: springboot-app
spec:
  containers:
  - name: app
    image: myapp:v1
    ports:
    - containerPort: 8080
    startupProbe:
      httpGet:
        path: /actuator/health
        port: 8080
      failureThreshold: 30
      periodSeconds: 10
    livenessProbe:
      httpGet:
        path: /actuator/health
        port: 8080
      initialDelaySeconds: 30
      periodSeconds: 10
      timeoutSeconds: 3
    readinessProbe:
      httpGet:
        path: /actuator/health
        port: 8080
      initialDelaySeconds: 10
      periodSeconds: 5
      timeoutSeconds: 2
```

------

# 十二、实际生产建议

------

## 1. Java 应用必须加 startupProbe

否则：

```text
容易反复重启
```

------

## 2. readinessProbe 一定要有

否则：

```text
服务未启动完成就接流量
```

会导致：

- 502
- 超时
- 雪崩

------

## 3. livenessProbe 不要过于敏感

错误示例：

```yaml
timeoutSeconds: 1
failureThreshold: 1
```

容易误杀。

------

## 4. 健康检查接口要轻量

不要：

- 查数据库
- 查 Redis
- 调第三方接口

否则：

```text
依赖波动
→ Pod 被误判异常
```

------

# 十三、查看 Probe 状态

------

## 查看 Pod 事件

```bash
kubectl describe pod pod-name
```

------

## 常见事件

### readiness 失败

```text
Readiness probe failed
```

------

### liveness 失败

```text
Liveness probe failed
```

------

### 容器重启

```text
Killing container
```

------

# 十四、常见问题

------

## 1. CrashLoopBackOff

原因：

- liveness 配置错误
- 应用启动太慢
- startupProbe 缺失

------

## 2. Pod 一直 0/1 Ready

原因：

- readiness 失败
- 健康检查接口错误
- 依赖未准备好

------

## 3. 探针误杀

原因：

- timeout 太短
- CPU 高负载
- GC STW

------

# 十五、面试高频问题

------

## 1. readiness 和 liveness 区别？

核心：

```text
readiness 控制流量
liveness 控制重启
```

------

## 2. 为什么需要 startupProbe？

解决：

```text
慢启动应用被 liveness 误杀
```

------

## 3. 探针失败后发生什么？

| Probe     | 结果         |
| --------- | ------------ |
| readiness | 摘流         |
| liveness  | 重启         |
| startup   | 启动失败重启 |

------

# 十六、生产环境推荐配置

------

## Java 微服务推荐

```yaml
startupProbe:
  failureThreshold: 30
  periodSeconds: 10

livenessProbe:
  periodSeconds: 10
  timeoutSeconds: 3
  failureThreshold: 3

readinessProbe:
  periodSeconds: 5
  timeoutSeconds: 2
  failureThreshold: 2
```

------

# 十七、总结

------

## Probe 本质

Probe 是 Kubernetes 的：

```text
容器健康检查机制
```

------

## 三种探针职责

| Probe     | 职责         |
| --------- | ------------ |
| startup   | 是否启动完成 |
| readiness | 是否能接流量 |
| liveness  | 是否需要重启 |

------

## 生产核心原则

```text
startup 防误杀
readiness 防错误流量
liveness 防服务假死
```