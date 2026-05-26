# Kubernetes 亲和性（Affinity）详解

Kubernetes 的亲和性（Affinity）是：

> “Pod 想去哪里运行”

它属于：

```text
Pod 主动选择 Node 或 Pod
```

而不是：

```text
Node 拒绝 Pod
```

因此它和：

- Taint/Toleration（污点容忍）

是两个方向。

------

# 一、为什么需要亲和性

默认情况下：

Scheduler 只会考虑：

- CPU
- 内存
- 资源是否足够

然后：

随机选择节点。

但生产环境中：

很多业务：

不能随便调度。

------

# 二、典型生产场景

------

## 场景1：数据库必须跑 SSD 节点

例如：

MySQL：

需要：

- 高 IO
- SSD

因此：

Pod 希望：

```text
只调度到 SSD 节点
```

------

## 场景2：日志服务靠近存储节点

例如：

- Elasticsearch
- Kafka

希望：

```text
与存储节点在同一机房
```

减少网络延迟。

------

## 场景3：高可用

3 个副本：

不能都在一个节点。

否则：

一个节点挂了：

全部挂。

因此：

需要：

```text
Pod 分散部署
```

------

## 场景4：GPU 业务

AI 服务：

必须运行在：

```text
gpu=true
```

节点。

------

# 三、Affinity 分类（非常重要）

K8s Affinity 分三大类：

| 类型              | 作用对象     |
| ----------------- | ------------ |
| Node Affinity     | Node 亲和性  |
| Pod Affinity      | Pod 亲和性   |
| Pod Anti-Affinity | Pod 反亲和性 |

这是核心。

------

# 四、Node Affinity（节点亲和性）

这是最常用的。

意思：

```text
Pod 希望运行在哪些 Node
```

本质：

Node 标签匹配。

------

# 五、为什么不用 nodeSelector

早期：

K8s 只有：

```yaml
nodeSelector:
  disktype: ssd
```

但：

功能太弱。

只能：

```text
key=value 精确匹配
```

不能：

- 大于
- 小于
- In
- NotIn
- Exists

因此：

出现了：

```text
NodeAffinity
```

------

# 六、Node Affinity 两种模式

这是重点。

| 类型                                            | 含义   |
| ----------------------------------------------- | ------ |
| requiredDuringSchedulingIgnoredDuringExecution  | 硬亲和 |
| preferredDuringSchedulingIgnoredDuringExecution | 软亲和 |

------

# 七、硬亲和（required）

意思：

```text
必须满足
否则不能调度
```

类似：

```text
硬性要求
```

------

# 八、硬亲和 YAML

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions: # 多个matchExpressions 有任意一个满足就行 是 或 or 的关系
          - key: disktype # 多个key的情况是需要都要满足的 是 且 and 的关系
            operator: In
            values:
            - ssd
          - key: memory
            operator: In
            values:
            - 16G
        - matchExpressions:
          - key: gpu
            operator: In
            values:
            - rtx5060

  containers:
  - name: nginx
    image: nginx
```

------

# 九、含义解析

------

## key

```yaml
key: disktype
```

表示：

匹配 Node 标签：

```text
disktype=?
```

------

## operator

```yaml
operator: In
```

表示：

值必须在：

```yaml
values:
- ssd
```

中。

------

## 最终含义

```text
只允许调度到：
disktype=ssd
的节点
```

------

# 十、IgnoredDuringExecution 是什么意思

这是很多人不会的。

完整名字：

```text
requiredDuringSchedulingIgnoredDuringExecution
```

拆开：

| 部分                   | 含义       |
| ---------------------- | ---------- |
| DuringScheduling       | 调度时检查 |
| IgnoredDuringExecution | 运行后忽略 |

------

## 举例

Pod 已运行。

后来：

Node 标签删除了：

```text
disktype=ssd
```

Pod：

```text
不会被驱逐
```

因为：

```text
IgnoredDuringExecution
```

------

# 十一、软亲和（preferred）

意思：

```text
最好满足
不满足也能调度
```

类似：

```text
尽量
```

------

# 十二、软亲和 YAML

```yaml
affinity:
  nodeAffinity:
    preferredDuringSchedulingIgnoredDuringExecution:
    - weight: 100
      preference:
        matchExpressions:
        - key: disktype
          operator: In
          values:
          - ssd
```

------

# 十三、weight 权重

```yaml
weight: 100
```

范围：

```text
1-100
```

权重越高：

Scheduler 越倾向该节点。

------

# 十四、调度逻辑

软亲和：

不是过滤。

而是：

```text
打分机制
```

Scheduler：

会优先选择：

得分高的节点。

------

# 十五、matchExpressions（重点）

这是亲和性的核心。

------

## 结构

```yaml
matchExpressions:
- key:
  operator:
  values:
```

------

# 十六、operator 详解（面试高频）

------

## 1. In

```yaml
operator: In
```

值必须在列表中。

例如：

```yaml
values:
- ssd
- nvme
```

------

## 2. NotIn

```yaml
operator: NotIn
```

不能在列表中。

------

## 3. Exists

```yaml
operator: Exists
```

只要 key 存在即可。

不关心 value。

------

## 4. DoesNotExist

```yaml
operator: DoesNotExist
```

key 不存在。

------

## 5. Gt

大于。

```yaml
operator: Gt
values:
- "2"
```

------

## 6. Lt

小于。

------

# 十七、多个条件关系（非常重要）

------

## matchExpressions 内部

```yaml
matchExpressions:
- key: cpu
- key: mem
```

关系：

```text
AND
```

必须全部满足。

------

## nodeSelectorTerms 之间

```yaml
nodeSelectorTerms:
- ...
- ...
```

关系：

```text
OR
```

满足一个即可。

------

# 十八、完整逻辑图

```text
nodeSelectorTerms:
    term1 OR term2

每个 term:
    expression1 AND expression2
```

这是面试经典。

------

# 十九、Pod Affinity（Pod 亲和）

这是：

```text
Pod 喜欢和哪些 Pod 在一起
```

------

# 二十、典型场景

例如：

Web 服务：

需要频繁访问 Redis。

希望：

```text
Web Pod 和 Redis Pod 在同一个 Node
```

减少网络延迟。

------

# 二十一、Pod Affinity YAML

```yaml
affinity:
  podAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
    - labelSelector:
        matchExpressions:
        - key: app
          operator: In
          values:
          - redis

      topologyKey: kubernetes.io/hostname
```

------

# 二十二、含义解析

------

## labelSelector

匹配 Pod 标签：

```text
app=redis
```

------

## topologyKey（超级重要）

这是：

```text
拓扑域
```

意思：

```text
“靠近”的范围
```

------

# 二十三、常见 topologyKey

------

## 1. kubernetes.io/hostname

表示：

```text
同一个节点
```

------

## 2. topology.kubernetes.io/zone

表示：

```text
同一个可用区
```

------

## 3. topology.kubernetes.io/region

表示：

```text
同一个地域
```

------

# 二十四、Pod Anti-Affinity（反亲和）

这是生产最重要的。

意思：

```text
Pod 不想和哪些 Pod 在一起
```

------

# 二十五、为什么重要

高可用核心。

例如：

3 个 nginx 副本。

如果：

都在 node1。

node1 宕机：

全部挂。

因此：

必须：

```text
分散部署
```

------

# 二十六、反亲和 YAML

```yaml
affinity:
  podAntiAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
    - labelSelector:
        matchExpressions:
        - key: app
          operator: In
          values:
          - nginx
      topologyKey: kubernetes.io/hostname
```

------

# 二十七、含义

```text
app=nginx 的 Pod
不能在同一个节点
```

------

# 二十八、效果

3 副本：

会自动分散到：

```text
node1
node2
node3
```

------

# 二十九、生产高可用核心

K8s 高可用：

最核心机制之一：

```text
PodAntiAffinity
```

尤其：

- 数据库
- Redis
- MQ
- 核心服务

------

# 三十、required vs preferred

所有 affinity：

都有：

| 类型      | 含义 |
| --------- | ---- |
| required  | 必须 |
| preferred | 尽量 |

------

# 三十一、required 的问题

如果：

没有满足条件节点。

Pod：

```text
Pending
```

无法调度。

------

# 三十二、preferred 的问题

可能：

最终仍调度到同一节点。

因此：

高可用生产：

很多时候必须：

```text
required
```

------

# 三十三、NodeAffinity 与 Taint 区别（面试高频）

这是核心中的核心。

------

## NodeAffinity

```text
Pod 主动选择节点
```

------

## Taint

```text
Node 主动拒绝 Pod
```

------

## 类比理解

------

## NodeAffinity

像：

```text
我想住海景房
```

------

## Taint

像：

```text
闲人免进
```

------

# 三十四、生产中的经典组合

实际生产：

经常：

```text
Taint + Affinity
```

一起使用。

------

# 三十五、GPU 场景

------

## 节点标签

```bash
kubectl label node node1 gpu=true
```

------

## 节点污点

```bash
kubectl taint node node1 gpu=true:NoSchedule
```

------

## Pod 配置

```yaml
spec:
  tolerations:
  - key: "gpu"
    operator: "Equal"
    value: "true"
    effect: "NoSchedule"

  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: gpu
            operator: In
            values:
            - "true"
```

------

# 三十六、为什么要两者一起

------

## toleration

只是：

```text
允许进入 GPU 节点
```

但：

不会主动去。

------

## affinity

才是：

```text
必须去 GPU 节点
```

------

# 三十七、调度流程（重要）

Scheduler：

------

## 第一步：过滤

检查：

- CPU
- 内存
- 污点
- required affinity

------

## 第二步：打分

检查：

- preferred affinity
- 资源均衡

------

## 第三步：绑定

选择最终 Node。

------

# 三十八、常见错误

------

## 1. 只写 toleration

结果：

Pod 可能去普通节点。

------

## 2. required 条件太严格

结果：

Pod 永远 Pending。

------

## 3. topologyKey 写错

结果：

反亲和失效。

------

# 三十九、查看调度失败原因

```bash
kubectl describe pod pod-name
```

会看到：

```text
0/3 nodes are available:
node(s) didn't match Pod affinity
```

------

# 四十、生产最佳实践

------

## 1. 专用业务

使用：

```text
Taint + NodeAffinity
```

------

## 2. 高可用服务

使用：

```text
PodAntiAffinity
```

------

## 3. 不要滥用 required

否则：

容易：

```text
Pending
```

------

## 4. topologyKey 推荐

节点级：

```text
kubernetes.io/hostname
```

机房级：

```text
topology.kubernetes.io/zone
```

------

# 四十一、面试高频题

------

## Q1：Affinity 和 nodeSelector 区别

| nodeSelector | Affinity         |
| ------------ | ---------------- |
| 简单         | 功能强           |
| 只能 =       | 支持 In/NotIn/Gt |
| 无软硬       | 有软硬           |

------

## Q2：PodAntiAffinity 有什么作用

```text
高可用分散部署
```

------

## Q3：为什么 Pod 会 Pending

可能：

```text
required affinity 无节点满足
```

------

## Q4：为什么 toleration 不够

因为：

```text
它只是允许进入
不是主动选择
```

------

# 四十二、总结（核心记忆）

------

## NodeAffinity

```text
Pod 想去哪个 Node
```

------

## PodAffinity

```text
Pod 想和谁在一起
```

------

## PodAntiAffinity

```text
Pod 不想和谁在一起
```

------

## required

```text
必须满足
```

------

## preferred

```text
尽量满足
```

------

## topologyKey

定义：

```text
“接近”的范围
```

最常见：

```text
kubernetes.io/hostname
```

表示：

```text
同一个节点
```