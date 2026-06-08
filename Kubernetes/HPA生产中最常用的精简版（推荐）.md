```yaml
## ==========================================
## HPA (HorizontalPodAutoscaler) 标准模板
## API版本：
## autoscaling/v2   （生产环境推荐）
##
## 支持：
## 1. CPU/Memory
## 2. 自定义指标
## 3. External指标
## 4. behavior扩缩容策略
## ==========================================
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  ## HPA资源名称
  name: order-service-hpa
  ## 所属命名空间
  namespace: default
  ## 标签（用于管理/筛选）
  labels:
    app: order-service
  ## 注解（可用于说明）
  annotations:
    description: "订单服务HPA"
spec:
  ## ==========================================
  ## scaleTargetRef
  ## 指定：
  ## HPA控制哪个资源
  ##
  ## 最常见：
  ## Deployment
  ##
  ## 也支持：
  ## StatefulSet
  ## ReplicaSet
  ## ==========================================
  scaleTargetRef:
    ## 目标资源API版本
    apiVersion: apps/v1
    ## 目标资源类型
    kind: Deployment
    ## 目标资源名称
    name: order-service
  ## ==========================================
  ## 最小Pod数量
  ##
  ## 即使没流量
  ## 也至少保留几个Pod
  ##
  ## 生产建议：
  ## 不要设置为1
  ##
  ## 常见：
  ## 2~3
  ## ==========================================
  minReplicas: 2
  ## ==========================================
  ## 最大Pod数量
  ##
  ## 防止：
  ## 无限扩容
  ##
  ## 非常重要！
  ##
  ## 如果不限制：
  ## 可能导致：
  ## - 数据库打崩
  ## - Node资源耗尽
  ## - 云资源费用暴涨
  ## ==========================================
  maxReplicas: 20
  ## ==========================================
  ## metrics
  ##
  ## 定义：
  ## HPA根据什么指标扩缩容
  ##
  ## 支持：
  ##
  ## 1. Resource
  ##    CPU/Memory
  ##
  ## 2. Pods
  ##    Pod自定义指标
  ##
  ## 3. Object
  ##    K8s对象指标
  ##
  ## 4. External
  ##    外部指标
  ## ==========================================
  metrics:
  ## ==========================================================
  ## 示例1：
  ## CPU扩缩容（最常见）
  ## ==========================================================
  - type: Resource
    resource:
      ## 资源名称
      ##
      ## 支持：
      ## cpu
      ## memory
      name: cpu
      target:
        ## 目标类型
        ##
        ## Utilization:
        ##   使用率（最常见）
        ##
        ## AverageValue:
        ##   平均值
        ##
        ## Value:
        ##   总值
        type: Utilization
        ## 平均CPU利用率
        ##
        ## 计算公式：
        ##
        ## 实际CPU使用量
        ## ----------------
        ## requests.cpu
        ##
        ## 例如：
        ##
        ## requests.cpu=500m
        ## 实际CPU=250m
        ##
        ## 利用率：
        ## 250/500 = 50%
        ##
        ## 超过50%
        ## 开始扩容
        averageUtilization: 70
  ## ==========================================================
  ## 示例2：
  ## Memory扩缩容
  ## ==========================================================
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        ## 平均内存利用率
        averageUtilization: 80
  ## ==========================================================
  ## 示例3：
  ## Pod自定义指标
  ##
  ## 例如：
  ## 每Pod QPS
  ## 每Pod连接数
  ## 每Pod线程数
  ##
  ## 需要：
  ## Prometheus Adapter
  ## ==========================================================
  - type: Pods
    pods:
      metric:
        ## Prometheus Adapter暴露的指标名
        ##
        ## 例如：
        ## http_requests
        name: http_requests
      target:
        ## AverageValue：
        ## 每个Pod平均值
        type: AverageValue
        ## 每Pod目标QPS
        ##
        ## 例如：
        ## 平均每Pod超过100QPS
        ## 自动扩容
        averageValue: "100"
  ## ==========================================================
  ## 示例4：
  ## Object指标
  ##
  ## 针对：
  ## 某个K8s对象
  ##
  ## 例如：
  ## Ingress QPS
  ## ==========================================================
  - type: Object
    object:
      metric:
        name: requests_per_second
      describedObject:
        apiVersion: networking.k8s.io/v1
        kind: Ingress
        name: api-gateway
      target:
        type: Value
        ## 整个对象总QPS
        value: "1000"
  ## ==========================================================
  ## 示例5：
  ## External指标（生产核心）
  ##
  ## 与K8s对象无关
  ##
  ## 最常见：
  ## Kafka Lag
  ## Redis Queue
  ## RabbitMQ backlog
  ##
  ## 需要：
  ## Prometheus Adapter
  ## ==========================================================
  - type: External
    external:
      metric:
        ## 外部指标名称
        ##
        ## 例如：
        ## kafka_lag
        name: kafka_lag
        ## 可选：
        ## 指标过滤标签
        selector:
          matchLabels:
            topic: order-topic
      target:
        ## Value：
        ## 指标总值
        type: Value
        ## 目标值
        ##
        ## 例如：
        ## lag超过10000
        ## 自动扩容
        value: "10000"
  ## ==========================================
  ## behavior
  ##
  ## HPA扩缩容行为控制
  ##
  ## 非常重要！
  ##
  ## 用于：
  ## 1. 防止抖动
  ## 2. 限制扩容速度
  ## 3. 限制缩容速度
  ##
  ## 生产环境强烈建议配置
  ## ==========================================
  behavior:
    ## ==========================================
    ## scaleUp
    ##
    ## 扩容策略
    ## ==========================================
    scaleUp:
      ## 稳定窗口
      ##
      ## 表示：
      ## 多久内观察指标
      ##
      ## 0：
      ## 立即扩容
      stabilizationWindowSeconds: 0
      ## 选择策略
      ##
      ## Max：
      ## 取最大策略
      ##
      ## Min：
      ## 取最小策略
      ##
      ## Disabled：
      ## 禁止扩容
      selectPolicy: Max
      ## 扩容规则
      policies:
      ## --------------------------------------
      ## 策略1：
      ## 按百分比扩容
      ## --------------------------------------
      - type: Percent
        ## 最大扩容比例
        ##
        ## 100：
        ## 最多翻倍
        value: 100
        ## 时间窗口
        ##
        ## 表示：
        ## 60秒内
        periodSeconds: 60
      ## --------------------------------------
      ## 策略2：
      ## 按Pod数量扩容
      ## --------------------------------------
      - type: Pods
        ## 最多增加4个Pod
        value: 4
        periodSeconds: 60
    ## ==========================================
    ## scaleDown
    ##
    ## 缩容策略
    ## ==========================================
    scaleDown:
      ## 缩容稳定窗口
      ##
      ## 非常重要！
      ##
      ## 默认：
      ## 300秒（5分钟）
      ##
      ## 用于：
      ## 防止频繁扩缩容
      stabilizationWindowSeconds: 300
      ## 策略选择
      selectPolicy: Max
      policies:
      ## --------------------------------------
      ## 每60秒最多缩50%
      ## --------------------------------------
      - type: Percent
        value: 50
        periodSeconds: 60
      ## --------------------------------------
      ## 每60秒最多减少2个Pod
      ## --------------------------------------
      - type: Pods
        value: 2
        periodSeconds: 60
```

------

# 一、生产中最常用的精简版（推荐）

实际生产：

最常见：

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: order-service-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: order-service
  minReplicas: 2
  maxReplicas: 20
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  behavior:
    scaleUp:
      stabilizationWindowSeconds: 0
    scaleDown:
      stabilizationWindowSeconds: 300
```

------

# 二、生产中的标准组合（非常重要）

现代生产：

最常见：

```text
Deployment
↓
HPA
↓
Prometheus
↓
Cluster Autoscaler
```

------

# 三、字段重要性排序（面试重点）

------

## 必须掌握

### 1、scaleTargetRef

控制谁。

------

### 2、minReplicas

最少 Pod。

------

### 3、maxReplicas

最大 Pod。

------

### 4、metrics

扩容依据。

------

### 5、behavior

扩缩容行为控制。

------

# 四、最重要的几个指标类型

------

## Resource

最基础：

```text
CPU
Memory
```

------

## Pods

最常见生产：

```text
每Pod QPS
```

------

## External

真正生产核心：

```text
Kafka Lag
Redis Queue
RabbitMQ
```

------

# 五、生产最容易出问题的字段

------

## 1、averageUtilization

依赖：

```yaml
requests:
  cpu:
```

没有 requests：

HPA 失效。

------

## 2、maxReplicas

太小：

扩不起来。

太大：

可能雪崩。

------

## 3、behavior

不配置：

容易：

```text
频繁扩缩容
```

------

# 六、HPA 最终本质（核心）

一句话：

```text
HPA本质就是：
根据metrics计算期望副本数，
然后修改Deployment replicas。
```

核心公式：

```text
期望副本数 =
当前副本数 × 当前指标 / 目标指标
```

------

# 七、生产级推荐模板（真正建议）

生产最推荐：

```text
CPU + QPS 双指标
```

例如：

```yaml
metrics:

- cpu 70%

- 每Pod QPS 1000
```

因为：

CPU：

不一定真实反映压力。

QPS：

更接近业务真实负载。