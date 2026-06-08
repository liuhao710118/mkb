## HPA + Prometheus 自定义指标详解（生产级）

这部分是真正的：

## Kubernetes 生产自动扩容核心

因为：

现实生产里：

## 仅靠 CPU 扩容远远不够

例如：

------

## 场景1：Kafka 消费者

CPU：

```text
10%
```

但是：

```text
Lag 1000万
```

消息严重堆积。

CPU 不高：

HPA 根本不会扩容。

------

## 场景2：Nginx 网关

CPU：

```text
20%
```

但：

```text
QPS 10万
连接数爆炸
```

已经扛不住。

------

## 场景3：Java 服务

CPU：

```text
30%
```

但是：

```text
线程池满
接口超时
```

CPU 不是瓶颈。

------

所以：

生产里真正常见的是：

## 根据业务指标扩容

例如：

- QPS
- Kafka Lag
- 请求数
- 队列长度
- 活跃连接
- RT
- Redis 队列
- RabbitMQ backlog

这就是：

## HPA + Prometheus 自定义指标

------

# 一、整体架构（必须理解）

先看完整链路：

```text
应用暴露metrics
        ↓
Prometheus采集
        ↓
Prometheus Adapter转换
        ↓
K8s Custom Metrics API
        ↓
HPA读取指标
        ↓
自动扩容
```

------

# 二、完整组件详解

核心组件：

------

## 1、应用 Metrics

应用暴露：

```text
/metrics
```

例如：

```text
http_requests_total
kafka_lag
redis_queue_size
```

------

## 2、Prometheus

负责：

```text
采集指标
存储指标
PromQL查询
```

------

## 3、Prometheus Adapter（核心）

这是：

## HPA 与 Prometheus 的桥梁

作用：

把：

```text
Prometheus指标
```

转换成：

```text
K8s Custom Metrics API
```

因为：

HPA：

## 不会直接读 Prometheus

它只能读：

```text
Custom Metrics API
```

------

## 4、HPA

读取：

```text
custom.metrics.k8s.io
```

然后：

自动扩容。

------

# 三、为什么必须有 Adapter

很多人：

最容易懵。

------

## HPA 不认识 Prometheus

HPA 只能调用：

```text
K8s Metrics API
```

例如：

```text
metrics.k8s.io
custom.metrics.k8s.io
external.metrics.k8s.io
```

Prometheus：

只是数据库。

所以：

必须有：

## Prometheus Adapter

进行“翻译”。

------

# 四、完整架构图（重点）

```text
                +-------------------+
                |      HPA          |
                +---------+---------+
                          |
                          | 查询指标
                          |
          custom.metrics.k8s.io
                          |
                +---------v---------+
                | Prometheus Adapter|
                +---------+---------+
                          |
                          | PromQL
                          |
                +---------v---------+
                |   Prometheus      |
                +---------+---------+
                          |
                          | scrape
                          |
                +---------v---------+
                |    Application    |
                +-------------------+
```

------

# 五、HPA 支持三类指标（生产重点）

------

## 1、Resource Metrics

资源指标：

```text
CPU
Memory
```

来源：

```text
metrics-server
```

------

## 2、Custom Metrics

K8s 对象相关指标。

例如：

```text
每个Pod QPS
```

例如：

```text
http_requests_per_second
```

------

## 3、External Metrics

外部指标。

例如：

```text
Kafka Lag
Redis Queue
RabbitMQ Queue
```

这些：

## 不属于 Pod

------

# 六、Custom Metrics 与 External Metrics 区别

这是面试高频。

------

## Custom Metrics

与 K8s 对象绑定。

例如：

```text
Pod QPS
```

每个 Pod：

有自己的指标。

------

## External Metrics

与对象无关。

例如：

```text
Kafka Topic Lag
```

这是全局指标。

------

# 七、生产最常见方案

------

## 方案1：按 QPS 扩容

例如：

```text
单Pod超过1000QPS
自动扩容
```

------

## 方案2：按 Kafka Lag 扩容

例如：

```text
Lag > 10000
自动增加消费者
```

------

## 方案3：按队列长度扩容

例如：

```text
Redis队列长度 > 5000
扩容Worker
```

------

# 八、部署完整链路（实战）

下面：

用：

## Prometheus + Adapter + HPA

做完整案例。

------

# 九、安装 Prometheus

生产：

一般：

## kube-prometheus-stack

Helm：

官方：

[kube-prometheus-stack Helm Chart](https://artifacthub.io/packages/helm/prometheus-community/kube-prometheus-stack?utm_source=chatgpt.com)

------

## 安装

```bash
helm repo add prometheus-community \
https://prometheus-community.github.io/helm-charts
helm install prometheus prometheus-community/kube-prometheus-stack
```

会安装：

- Prometheus
- Grafana
- Alertmanager
- Operator

------

# 十、应用暴露 metrics

例如：

Java SpringBoot：

------

## 引入 actuator

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

------

## 暴露 metrics

```yaml
management:
  endpoints:
    web:
      exposure:
        include: prometheus
```

------

## 查看

```bash
curl http://pod-ip:8080/actuator/prometheus
```

会看到：

```text
http_server_requests_seconds_count
jvm_memory_used_bytes
```

------

# 十一、Prometheus 采集

通过：

```text
ServiceMonitor
```

------

## 示例

```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor

metadata:
  name: order-service

spec:
  selector:
    matchLabels:
      app: order-service

  endpoints:
  - port: http
    path: /actuator/prometheus
```

------

# 十二、验证 Prometheus 已采集

进入 Prometheus：

查询：

```promql
http_requests_total
```

有数据：

说明成功。

------

# 十三、安装 Prometheus Adapter（核心）

官方：

[Prometheus Adapter GitHub](https://github.com/kubernetes-sigs/prometheus-adapter?utm_source=chatgpt.com)

------

## Helm 安装

```bash
helm repo add prometheus-community \
https://prometheus-community.github.io/helm-charts
helm install prometheus-adapter \
prometheus-community/prometheus-adapter
```

------

# 十四、Adapter 最核心配置

这是：

## 全篇最重要部分

------

## rules 配置

例如：

```yaml
rules:
- seriesQuery: 'http_requests_total{kubernetes_namespace!="",kubernetes_pod_name!=""}'

  resources:
    overrides:
      kubernetes_namespace:
        resource: namespace

      kubernetes_pod_name:
        resource: pod

  name:
    matches: "^(.*)_total"
    as: "${1}"

  metricsQuery: |
    sum(rate(http_requests_total{<<.LabelMatchers>>}[2m]))
    by (<<.GroupBy>>)
```

------

# 十五、这段到底是什么意思（超详细）

这是：

## Prometheus Adapter 的映射规则

作用：

告诉 Adapter：

```text
Prometheus 哪个指标
对应 K8s 哪个对象
```

------

## 1、seriesQuery

```yaml
seriesQuery:
```

用于：

```text
发现指标
```

例如：

```text
http_requests_total
```

------

## 2、resources

用于：

```text
Prometheus标签
映射到
K8s资源
```

------

例如：

```yaml
kubernetes_pod_name -> pod
```

表示：

Prometheus 标签：

```text
kubernetes_pod_name
```

对应：

```text
K8s Pod
```

------

## 3、name

定义：

HPA 看到的指标名。

------

## 4、metricsQuery

真正执行的：

## PromQL

例如：

```promql
sum(rate(http_requests_total[2m])) by (pod)
```

------

# 十六、查看 Custom Metrics API

这是：

验证最关键步骤。

------

## 查看 API

```bash
kubectl get --raw \
/apis/custom.metrics.k8s.io/v1beta1
```

如果成功：

会看到：

```json
{
  "resources":[
    {
      "name":"pods/http_requests"
    }
  ]
}
```

说明：

## Adapter 成功了

------

# 十七、创建 HPA（Custom Metrics）

------

## 示例

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler

metadata:
  name: order-hpa

spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: order-service

  minReplicas: 2
  maxReplicas: 20

  metrics:
  - type: Pods

    pods:
      metric:
        name: http_requests

      target:
        type: AverageValue
        averageValue: 100
```

------

# 十八、含义详解

------

## averageValue: 100

表示：

```text
每个Pod平均100QPS
```

超过：

扩容。

------

## 举例

当前：

```text
2个Pod
总QPS:
400
```

平均：

```text
200
```

目标：

```text
100
```

计算：

```text
2 × 200 / 100 = 4
```

扩容：

```text
2 -> 4
```

------

# 十九、External Metrics（Kafka Lag）

这是：

生产最常见。

------

## 场景

Kafka：

```text
消息大量堆积
```

CPU：

却很低。

所以：

按：

## lag 扩容

------

## 示例

```yaml
metrics:
- type: External

  external:
    metric:
      name: kafka_lag

    target:
      type: Value
      value: 10000
```

------

## 含义

```text
Lag > 10000
开始扩容
```

------

# 二十、为什么 Kafka 特别适合 HPA

因为：

消费者天然可并行。

例如：

```text
1个消费者
处理不过来
```

扩：

```text
10个消费者
```

吞吐提升巨大。

------

# 二十一、生产中的 PromQL（核心）

HPA 本质：

最终依赖：

## PromQL

------

## 1、QPS

```promql
sum(rate(http_requests_total[1m]))
```

------

## 2、每Pod QPS

```promql
sum(rate(http_requests_total[1m])) by (pod)
```

------

## 3、Kafka Lag

```promql
sum(kafka_consumergroup_lag)
```

------

## 4、Redis Queue

```promql
redis_queue_size
```

------

# 二十二、HPA 查询周期

默认：

```text
15秒
```

controller-manager：

参数：

```bash
--horizontal-pod-autoscaler-sync-period
```

------

# 二十三、生产中的大坑（非常重要）

------

## 坑1：Prometheus 延迟

Prometheus：

默认：

```text
15s scrape
```

再加：

```text
15s HPA同步
```

可能：

30秒后才扩容。

------

## 坑2：Java 启动慢

扩容了：

但：

```text
Pod 2分钟才Ready
```

流量还是爆。

------

## 解决

- JVM预热
- CDS
- readinessProbe
- 降低启动时间

------

## 坑3：指标抖动

例如：

QPS：

```text
100
10000
100
10000
```

疯狂扩缩。

------

## 解决

behavior：

```yaml
behavior:
  scaleDown:
    stabilizationWindowSeconds: 300
```

------

## 坑4：PromQL 写错

例如：

```promql
sum(http_requests_total)
```

这是累计值。

会：

## 永远增长

HPA：

无限扩容。

------

## 正确：

```promql
rate()
```

------

# 二十四、生产最佳实践（非常重要）

------

## 1、优先按业务指标扩容

不要只看 CPU。

------

## 2、QPS 比 CPU 更真实

因为：

CPU：

不能反映：

- IO等待
- DB阻塞
- 线程池满

------

## 3、一定限制 maxReplicas

否则：

可能：

```text
疯狂扩容
数据库打崩
```

------

## 4、PromQL 一定测试

先在 Prometheus：

验证。

------

## 5、扩容速度不要太快

否则：

```text
雪崩
```

------

# 二十五、完整生产链路（真正核心）

现代生产：

基本：

```text
应用metrics
↓
Prometheus
↓
Adapter
↓
HPA
↓
Deployment
↓
Pod扩容
↓
Cluster Autoscaler扩Node
```

这是：

## 云原生自动弹性核心

------

# 二十六、面试高频题

------

## 1、为什么 HPA 不能直接读 Prometheus？

答：

```text
HPA只能访问K8s Metrics API，
Prometheus Adapter负责转换。
```

------

## 2、Custom Metrics 与 External Metrics 区别？

答：

Custom：

```text
与K8s对象关联
```

External：

```text
外部全局指标
```

------

## 3、为什么 PromQL 要用 rate？

答：

因为：

```text
counter是累计值
```

不用：

```text
rate()
```

会无限增长。

------

## 4、为什么 HPA 扩容慢？

答：

- scrape interval
- HPA sync period
- Pod启动时间

都会导致延迟。

------

# 二十七、生产最常见监控指标

------

## Web 服务

- QPS
- 活跃连接
- RT
- 线程池

------

## MQ 消费

- Kafka Lag
- RabbitMQ backlog

------

## Redis

- queue size

------

## JVM

- heap usage
- gc pause

------

# 二十八、最终理解（核心思想）

## CPU 扩容：

属于基础版。

真正生产：

## 都是业务指标扩容。

核心：

```text
Prometheus负责存储指标
Adapter负责转换API
HPA负责扩缩容
```

最重要：

## PromQL 决定扩容质量

因为：

指标不对：

整个自动扩容都会失控。