## Kubernetes HPA（Horizontal Pod Autoscaler）超详细讲解

HPA 是 Kubernetes 中最核心的“自动扩缩容”能力之一。

它的作用：

> 根据监控指标（CPU、内存、自定义指标等）自动增加或减少 Pod 数量。

简单理解：

- 流量大 → 自动扩容 Pod
- 流量小 → 自动缩容 Pod

它解决的问题：

- 避免高峰期服务扛不住
- 避免低峰期资源浪费
- 不需要人工 `kubectl scale`

------

# 一、先理解：K8s 为什么需要 HPA

假设：

你的 Java 微服务：

```bash
Deployment: order-service
副本数: 2
```

正常时：

```text
2 个 Pod
CPU 20%
```

晚上秒杀：

```text
CPU 95%
请求大量超时
```

如果不扩容：

- Pod CPU 打满
- 响应慢
- OOM
- 服务崩

传统方式：

```bash
kubectl scale deployment order-service --replicas=10
```

问题：

- 需要人工
- 无法实时
- 夜里没人值班

所以：

K8s 引入：

## HPA（水平自动扩容）

自动：

```text
CPU高 → Pod增加
CPU低 → Pod减少
```

------

# 二、什么是“水平扩容”

这是很多人最容易混淆的点。

------

### 1、水平扩容（HPA）

增加 Pod 数量。

例如：

```text
原来:
2个Pod

扩容后:
10个Pod
```

叫：

## Horizontal Scaling（横向扩容）

------

### 2、垂直扩容（VPA）

增加 Pod 配置。

例如：

```text
CPU:
500m -> 2核

内存:
1Gi -> 4Gi
```

叫：

## Vertical Scaling（纵向扩容）

------

### 3、Cluster Autoscaler

节点不够时：

自动增加 Node。

例如：

```text
原来:
3个Node

自动变:
10个Node
```

------

# 三、HPA 的本质是什么

本质：

## HPA = 一个控制器（Controller）

它会：

循环执行：

```text
获取指标
↓
计算目标副本数
↓
修改 Deployment replicas
↓
Deployment 创建/删除 Pod
```

注意：

HPA：

## 不直接创建 Pod

真正创建 Pod 的：

是 Deployment / ReplicaSet。

------

# 四、HPA 工作流程（极其重要）

## 完整流程

```text
用户请求
    ↓
Pod CPU升高
    ↓
metrics-server采集指标
    ↓
HPA读取指标
    ↓
HPA计算需要几个Pod
    ↓
修改Deployment replicas
    ↓
Deployment创建新Pod
    ↓
流量被分摊
```

------

# 五、HPA 架构详解

核心组件：

```text
HPA
metrics-server
kube-controller-manager
Deployment
```

------

# 六、metrics-server 是什么

这是 HPA 最关键组件。

很多人：

```text
HPA 不生效
```

本质：

## metrics-server 没装

------

### 1、它负责什么

采集：

```text
Pod CPU
Pod Memory
Node CPU
Node Memory
```

来源：

```text
kubelet
```

------

### 2、没有 metrics-server 会怎样

执行：

```bash
kubectl top pod
```

会报错：

```text
Metrics API not available
```

HPA：

也无法工作。

------

# 七、安装 metrics-server

官方：

[metrics-server GitHub](https://github.com/kubernetes-sigs/metrics-server?utm_source=chatgpt.com)

常用：

```bash
kubectl apply -f \
https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

很多实验环境：

还需要：

```yaml
args:
- --kubelet-insecure-tls
```

否则：

TLS 证书校验失败。

------

# 八、检查 metrics-server 是否正常

------

### 1、查看 Pod

```bash
kubectl get pod -n kube-system
```

看：

```text
metrics-server
Running
```

------

### 2、查看指标

```bash
kubectl top node
kubectl top pod
```

有数据：

说明成功。

------

# 九、HPA 支持哪些指标

HPA v2 支持很多指标。

------

## 1、Resource Metrics（最常用）

资源指标：

- CPU
- Memory

例如：

```yaml
cpu: 80%
```

意思：

```text
CPU平均使用率超过80%
开始扩容
```

------

## 2、Pod Metrics

每个 Pod 自定义指标。

例如：

```text
QPS
连接数
线程数
```

------

## 3、Object Metrics

某个对象指标。

例如：

```text
Ingress QPS
```

------

## 4、External Metrics

外部监控系统。

例如：

- Prometheus
- Kafka lag
- RabbitMQ queue
- Redis连接数

------

# 十、HPA 最核心原理（必须掌握）

## HPA 计算公式

最核心：

```text
期望副本数 =
当前副本数 × 当前指标值 / 目标指标值
```

------

## 举例

当前：

```text
Deployment:
2个Pod

目标CPU:
50%

实际CPU:
100%
```

计算：

```text
2 × 100 / 50 = 4
```

所以：

```text
扩容到4个Pod
```

------

## 再举例

当前：

```text
10个Pod
目标CPU: 50%
实际CPU: 25%
```

计算：

```text
10 × 25 / 50 = 5
```

缩容：

```text
10 -> 5
```

------

# 十一、为什么 HPA 必须设置 requests

这是面试高频。

------

## HPA CPU 利用率依据：

```text
实际CPU使用量 / requests.cpu
```

不是 limits。

------

## 举例

Pod：

```yaml
resources:
  requests:
    cpu: 500m
```

实际：

```text
CPU使用:
250m
```

则：

```text
CPU利用率:
250 / 500 = 50%
```

------

## 如果不设置 requests

HPA 无法计算。

报错：

```text
missing request for cpu
```

------

# 十二、第一个 HPA 示例（超详细）

------

## Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  replicas: 2
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
        resources:
          requests:
            cpu: 100m
          limits:
            cpu: 500m
```

------

## 创建 HPA

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: nginx-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nginx
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
```

------

## 含义

------

### 1、scaleTargetRef

目标对象：

```text
对谁扩容
```

这里：

```text
Deployment/nginx
```

------

### 2、minReplicas

最少：

```text
至少2个Pod
```

即使没流量。

------

### 3、maxReplicas

最多：

```text
最多10个Pod
```

防止无限扩容。

------

### 4、averageUtilization

目标 CPU 利用率：

```text
50%
```

超过：扩容。

低于：缩容。

------

# 十三、创建 HPA

------

### 1、apply

```bash
kubectl apply -f hpa.yaml
```

------

### 2、查看

```bash
kubectl get hpa
```

结果：

```text
NAME        REFERENCE          TARGETS   MINPODS   MAXPODS   REPLICAS
nginx-hpa   Deployment/nginx   20%/50%  2         10        2
```

------

## TARGETS 含义

```text
当前CPU / 目标CPU
```

例如：

```text
80%/50%
```

表示：

```text
当前CPU 80%
目标50%
```

会触发扩容。

------

# 十四、压测触发扩容

------

## 启动压测 Pod

```bash
kubectl run -it --rm busybox \
--image=busybox /bin/sh
```

循环请求：

```bash
while true; do wget -q -O- http://nginx; done
```

------

## 查看 HPA

```bash
kubectl get hpa -w
```

会看到：

```text
REPLICAS:
2 -> 4 -> 6
```

------

## 查看 Deployment

```bash
kubectl get pod
```

Pod 增加。

------

# 十五、HPA 缩容为什么慢

很多人发现：

```text
扩容很快
缩容很慢
```

这是故意设计。

因为：

## 防止抖动（Thrashing）

------

## 什么叫抖动

例如：

```text
CPU:
49%
51%
49%
51%
```

如果立刻扩缩：

```text
5 -> 6 -> 5 -> 6
```

系统震荡。

------

## Kubernetes 默认：

缩容会等待。

默认：

```text
5分钟
```

------

# 十六、behavior（生产核心）

autoscaling/v2：

支持行为控制。

------

## 示例

```yaml
behavior:
  scaleUp:
    stabilizationWindowSeconds: 0

  scaleDown:
    stabilizationWindowSeconds: 300
```

------

## 含义

------

### scaleUp

扩容稳定窗口：

```text
0秒
```

立即扩容。

------

### scaleDown

缩容稳定窗口：

```text
300秒
```

5分钟后再缩。

------

# 十七、限制每次扩容数量

避免：

```text
2个Pod
瞬间扩容到100
```

------

## 示例

```yaml
behavior:
  scaleUp:
    policies:
    - type: Percent
      value: 100
      periodSeconds: 60
```

------

## 含义

60秒最多扩：

```text
100%
```

例如：

```text
2 -> 4
4 -> 8
```

而不是：

```text
2 -> 100
```

------

# 十八、HPA 与 Deployment 的关系

HPA：

本质修改：

```yaml
spec.replicas
```

所以：

不要：

```bash
kubectl scale
```

和 HPA 同时长期混用。

因为：

HPA 会覆盖。

------

# 十九、HPA 与 requests/limits 的关系（重点）

很多人：

不会调 requests。

导致：

HPA 完全失效。

------

## 举例

Pod 实际：

```text
CPU常态:
500m
```

但你设置：

```yaml
requests:
  cpu: 50m
```

则：

```text
CPU利用率:
500 / 50 = 1000%
```

HPA：

疯狂扩容。

------

## 正确思路

requests：

应该接近：

```text
正常平均值
```

而不是乱填。

------

# 二十、HPA 为什么可能不生效

这是生产最重要部分。

------

## 1、metrics-server 未安装

最常见。

------

## 2、没设置 requests

报错：

```text
missing request for cpu
```

------

## 3、CPU太低

例如：

```text
目标50%
实际20%
```

不会扩。

------

## 4、达到 maxReplicas

例如：

```yaml
maxReplicas: 10
```

已经10个。

不能再扩。

------

## 5、Pod Pending

节点资源不足。

新 Pod 起不来。

------

## 6、Cluster Autoscaler 未开启

Pod 扩了。

但：

Node 不够。

------

# 二十一、HPA 与 Cluster Autoscaler 配合

这是生产环境核心。

------

## 场景

HPA：

```text
2 -> 20 Pod
```

Node 放不下。

于是：

Pod Pending。

------

## Cluster Autoscaler

发现：

```text
资源不足
```

自动：

```text
增加Node
```

------

## 完整链路

```text
流量上涨
↓
HPA扩Pod
↓
Node不够
↓
CA扩Node
↓
Pod成功调度
```

------

# 二十二、HPA 与 VPA 冲突

通常：

## 不建议同时控制 CPU/Memory

因为：

------

## HPA：

增加 Pod 数量

------

## VPA：

增加单 Pod 资源

------

会互相影响。

例如：

```text
HPA觉得CPU高
准备扩Pod

VPA提高CPU request

结果CPU利用率又下降
```

会震荡。

------

# 二十三、生产最佳实践（非常重要）

------

## 1、一定设置 requests

否则：

HPA 无法工作。

------

## 2、不要乱设 requests

否则：

利用率失真。

------

## 3、设置合理 maxReplicas

防止：

```text
数据库被打爆
```

------

## 4、设置 behavior

避免：

扩缩容震荡。

------

## 5、监控扩容速度

防止：

```text
Pod还没启动
HPA继续扩
```

------

## 6、Java 应用特别注意

Java：

启动慢。

可能：

```text
HPA已经扩了
但Pod还没Ready
```

需要：

- readinessProbe
- 合理 JVM
- 预热

------

# 二十四、HPA v1 与 v2 区别

------

## v1

只支持：

```text
CPU
```

------

## v2

支持：

- CPU
- Memory
- 自定义指标
- behavior

生产：

## 基本都用 v2

------

# 二十五、查看 HPA 详细状态

------

### describe

```bash
kubectl describe hpa nginx-hpa
```

能看到：

- 当前指标
- 期望副本
- 扩缩容事件
- 是否失败

非常重要。

------

# 二十六、生产最常见架构

现在最常见：

```text
Prometheus
↓
Adapter
↓
HPA
```

例如：

根据：

- QPS
- Kafka Lag
- Redis Queue

自动扩容。

------

# 二十七、经典面试题

------

## 1、HPA 工作原理？

答：

```text
HPA周期性获取metrics-server指标，
根据目标值计算期望副本数，
再修改Deployment replicas实现扩缩容。
```

------

## 2、为什么 HPA 必须设置 requests？

答：

```text
CPU利用率=
实际CPU使用量 / requests.cpu

不设置requests无法计算利用率。
```

------

## 3、HPA 为什么缩容慢？

答：

```text
防止抖动，
K8s默认有缩容稳定窗口。
```

------

## 4、HPA 扩 Pod 了但业务还是慢？

可能：

- Pod 未 Ready
- Java 启动慢
- 数据库瓶颈
- Node 不够
- 限流
- GC

------

## 5、HPA 与 CA 区别？

HPA：

```text
扩Pod
```

CA：

```text
扩Node
```

------

# 二十八、一张图彻底理解 HPA

```text
                 +----------------+
                 | metrics-server |
                 +--------+-------+
                          |
                          | CPU/Memory
                          |
+--------+       +--------v--------+
|  HPA   +-------> 计算目标副本数   |
+--------+       +--------+--------+
                          |
                          | 修改 replicas
                          |
                 +--------v--------+
                 |   Deployment    |
                 +--------+--------+
                          |
                    创建/删除Pod
                          |
                 +--------v--------+
                 |      Pod        |
                 +-----------------+
```

------

# 二十九、生产中的真实问题（很重要）

------

## 问题1

```text
CPU 100%
HPA没扩容
```

原因：

```text
requests设置错误
```

------

## 问题2

```text
HPA疯狂扩容
```

原因：

```text
requests太小
```

------

## 问题3

```text
Pod扩了还是扛不住
```

原因：

```text
数据库瓶颈
```

HPA：

只能解决：

## 计算层瓶颈

不能解决：

- DB
- Redis
- MQ

------

# 三十、总结（核心思想）

## HPA 核心本质

一句话：

```text
根据监控指标自动调整Pod数量
```

核心链路：

```text
metrics-server
↓
HPA
↓
Deployment replicas
↓
Pod扩缩容
```

核心公式：

```text
期望副本数 =
当前副本数 × 当前指标 / 目标指标
```

生产核心：

```text
requests
behavior
metrics-server
Cluster Autoscaler
```

如果你愿意，我还能继续给你深入讲：

1. HPA + Prometheus 自定义指标详解
2. HPA 源码级工作原理
3. HPA 生产故障排查大全
4. HPA + JVM 调优
5. HPA 与 VPA/CA 三者关系
6. HPA 全链路实战案例
7. metrics-server 原理详解
8. HPA 如何根据 QPS 自动扩容
9. HPA 为什么会扩容失控
10. HPA 面试题大全（高级）