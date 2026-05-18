# Kubernetes 污点（Taint）与容忍度（Toleration）详解

Kubernetes 的 **Taint（污点）** 和 **Toleration（容忍）** 是非常核心的调度机制。

它解决的是：

> “哪些 Pod 能运行到哪些 Node 上”

它本质上属于：

- **Node 对 Pod 的排斥机制**
- 而不是 Pod 主动选择 Node

很多人容易和：

- nodeSelector
- affinity
- anti-affinity

混淆。

实际上：

| 功能         | 作用方向          |
| ------------ | ----------------- |
| nodeSelector | Pod 主动选择节点  |
| affinity     | Pod 偏向某些节点  |
| taint        | Node 主动拒绝 Pod |
| toleration   | Pod 表示“我能忍”  |

------

# 一、为什么需要污点（Taint）

先理解一个问题：

默认情况下：

> Scheduler 会尽量把 Pod 调度到任何可用 Node

但生产环境中：

有些节点不能随便调度。

例如：

------

## 场景1：GPU 节点

公司有：

- 普通 Node
- GPU Node（价格昂贵）

你不希望：

普通业务 Pod 占用 GPU 节点。

所以：

GPU 节点需要：

> “拒绝普通 Pod”

------

## 场景2：数据库专用节点

数据库：

- IO 高
- CPU 高
- 内存高

你希望：

数据库只运行在指定节点。

------

## 场景3：故障节点

Node 出现：

- 磁盘异常
- 网络异常
- 内存压力

K8s 需要：

- 阻止新 Pod 调度
- 甚至驱逐已有 Pod

------

## 场景4：运维维护节点

Node 升级：

- 内核
- Docker
- kubelet

需要：

- 不再调度新 Pod
- 慢慢迁移已有 Pod

------

# 二、污点（Taint）是什么

污点：

> 是加在 Node 上的一种“排斥属性”

Node 说：

> “我这里有污点，普通 Pod 不准来”

只有：

> 带有对应 toleration 的 Pod 才允许调度。

------

# 三、Taint 的结构

一个污点由：

```text
key=value:effect
```

组成。

例如：

```bash
kubectl taint node node1 gpu=true:NoSchedule
```

含义：

| 字段   | 含义       |
| ------ | ---------- |
| key    | gpu        |
| value  | true       |
| effect | NoSchedule |

意思：

> node1 有 GPU 专属污点

普通 Pod 不允许调度。

------

# 四、effect（效果）详解

最核心的是 effect。

K8s 只有三种 effect：

| effect           | 含义                       |
| ---------------- | -------------------------- |
| NoSchedule       | 不允许调度新 Pod           |
| PreferNoSchedule | 尽量不调度                 |
| NoExecute        | 不仅不调度，还驱逐已有 Pod |

这是面试高频。

------

# 五、NoSchedule（最常用）

## 含义

```text
没有 toleration 的 Pod
不能调度到该节点
```

但是：

> 已经运行的 Pod 不受影响

------

## 示例

给 node1 打污点：

```bash
kubectl taint node node1 env=prod:NoSchedule
```

查看：

```bash
kubectl describe node node1
```

会看到：

```text
Taints: env=prod:NoSchedule
```

------

## 调度行为

### 普通 Pod

不能调度：

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx
```

状态：

```text
Pending
```

事件：

```text
node(s) had taint {env: prod}
```

------

## 加 toleration 后

```yaml
tolerations:
- key: "env"
  operator: "Equal"
  value: "prod"
  effect: "NoSchedule"
```

Pod 就能调度。

------

# 六、PreferNoSchedule

这是：

> “软限制”

意思：

```text
尽量不要调度
但实在没地方也能调度
```

类似：

- preferredDuringScheduling

而不是：

- requiredDuringScheduling

------

## 使用场景

例如：

测试节点。

你希望：

- 正常情况下不调度
- 资源不足时允许调度

------

## 示例

```bash
kubectl taint node node1 test=true:PreferNoSchedule
```

------

## 行为

Scheduler：

- 优先避开该节点
- 但不是绝对禁止

------

# 七、NoExecute（最重要）

这是最强的 effect。

它会：

## 1. 禁止新 Pod 调度

## 2. 驱逐已有 Pod

这是和 NoSchedule 最大区别。

------

# 八、NoExecute 驱逐机制

例如：

```bash
kubectl taint node node1 down=true:NoExecute
```

会发生：

| Pod 类型   | 结果     |
| ---------- | -------- |
| 新 Pod     | 不能调度 |
| 已运行 Pod | 被驱逐   |

------

# 九、tolerationSeconds

这是 NoExecute 的核心。

可以：

> 延迟驱逐

------

## 示例

```yaml
tolerations:
- key: "down"
  operator: "Equal"
  value: "true"
  effect: "NoExecute"
  tolerationSeconds: 60
```

含义：

```text
Pod 能容忍 60 秒
60 秒后被驱逐
```

------

## 常见用途

节点短暂异常：

例如：

- 网络抖动
- kubelet 重启

避免：

- 立刻驱逐 Pod

------

# 十、operator 详解

容忍规则中的 operator：

| operator | 含义                   |
| -------- | ---------------------- |
| Equal    | key/value 必须完全匹配 |
| Exists   | 只要 key 存在即可      |

------

# 十一、Equal

## 示例

```yaml
tolerations:
- key: "gpu"
  operator: "Equal"
  value: "true"
  effect: "NoSchedule"
```

只能匹配：

```text
gpu=true
```

------

# 十二、Exists

## 示例

```yaml
tolerations:
- key: "gpu"
  operator: "Exists"
  effect: "NoSchedule"
```

可以匹配：

```text
gpu=true
gpu=false
gpu=abc
```

只看 key。

------

# 十三、污点和容忍的匹配规则

匹配条件：

| 项     | 必须匹配     |
| ------ | ------------ |
| key    | 必须         |
| effect | 必须         |
| value  | Equal 时必须 |

------

# 十四、多个污点如何工作

Node 可以有多个污点。

例如：

```text
gpu=true:NoSchedule
env=prod:NoExecute
```

------

## 调度规则

Pod：

> 必须容忍所有“阻止性污点”

否则：

不能调度。

------

# 十五、官方内置污点（重点）

K8s 会自动添加很多系统污点。

面试非常爱问。

------

# 十六、node.kubernetes.io/not-ready

Node：

```text
NotReady
```

时自动添加：

```text
node.kubernetes.io/not-ready
```

通常 effect：

```text
NoExecute
```

------

# 十七、node.kubernetes.io/unreachable

Node：

失联时：

```text
node.kubernetes.io/unreachable
```

------

# 十八、memory-pressure

节点内存不足：

```text
node.kubernetes.io/memory-pressure
```

------

# 十九、disk-pressure

磁盘不足：

```text
node.kubernetes.io/disk-pressure
```

------

# 二十、pid-pressure

PID 不足：

```text
node.kubernetes.io/pid-pressure
```

------

# 二十一、unschedulable

执行：

```bash
kubectl cordon node1
```

后：

自动添加：

```text
node.kubernetes.io/unschedulable
```

effect：

```text
NoSchedule
```

------

# 二十二、Master 节点默认污点

老版本：

Master 默认：

```text
node-role.kubernetes.io/master:NoSchedule
```

新版本：

```text
node-role.kubernetes.io/control-plane:NoSchedule
```

因此：

普通 Pod 不会调度到 Master。

------

# 二十三、查看污点

## 查看节点污点

```bash
kubectl describe node node1
```

查看：

```text
Taints:
```

------

## json 查看

```bash
kubectl get node node1 -o json
```

------

# 二十四、添加污点

```bash
kubectl taint node node1 gpu=true:NoSchedule
```

------

# 二十五、删除污点

注意最后有减号：

```bash
kubectl taint node node1 gpu=true:NoSchedule-
```

------

# 二十六、Pod 如何添加 toleration

## Deployment 示例

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: gpu-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: gpu
  template:
    metadata:
      labels:
        app: gpu
    spec:
      tolerations:
      - key: "gpu"
        operator: "Equal"
        value: "true"
        effect: "NoSchedule"

      containers:
      - name: nginx
        image: nginx
```

------

# 二十七、污点 ≠ 节点选择

这是非常容易误解的。

污点：

```text
只是允许你进去
不代表一定调度进去
```

很多人误以为：

加 toleration 后：

Pod 就会调度到该节点。

错。

------

# 二十八、正确理解

Taint/Toleration：

```text
只是“门禁”
```

NodeAffinity：

```text
才是“选择”
```

------

# 二十九、生产环境经典组合

生产里：

通常：

```text
Taint + NodeAffinity
```

一起使用。

------

## 示例

GPU 节点：

### Node 标签

```text
gpu=true
```

### Node 污点

```text
gpu=true:NoSchedule
```

------

## Pod 配置

### toleration

允许进入。

### affinity

指定必须去 GPU 节点。

------

# 三十、完整生产 YAML

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

# 三十一、调度流程（非常重要）

Scheduler 工作：

## 第一步：过滤（Filter）

检查：

- CPU
- 内存
- 污点
- 亲和性

如果：

Pod 不容忍污点：

直接淘汰节点。

------

## 第二步：打分（Score）

在剩余节点中：

- 打分
- 选择最优节点

------

# 三十二、污点的本质

本质：

```text
Node 维度的访问控制
```

类似：

Linux：

```text
iptables
```

或者：

```text
门禁系统
```

------

# 三十三、污点最佳实践

------

## 1. 专用节点必须加污点

例如：

- GPU
- Elasticsearch
- Kafka
- 数据库

------

## 2. 不要只用 toleration

因为：

toleration 不会主动选择节点。

必须结合：

```text
nodeAffinity
```

------

## 3. NoExecute 谨慎使用

因为：

会驱逐 Pod。

容易：

- 服务中断
- 大规模漂移

------

## 4. PreferNoSchedule 不可靠

它只是：

```text
尽量避免
```

不是强限制。

------

# 三十四、常见问题排查

------

## Pod Pending

查看：

```bash
kubectl describe pod pod-name
```

事件：

```text
had taint that the pod didn't tolerate
```

说明：

没加 toleration。

------

## Pod 被驱逐

查看：

```bash
kubectl describe node
```

看看：

```text
NoExecute
```

污点。

------

# 三十五、面试高频问题

------

## Q1：NoSchedule 和 NoExecute 区别

| 类型       | 新 Pod | 已有 Pod |
| ---------- | ------ | -------- |
| NoSchedule | 禁止   | 不影响   |
| NoExecute  | 禁止   | 驱逐     |

------

## Q2：toleration 是否等于 nodeSelector？

不是。

| 功能         | 作用     |
| ------------ | -------- |
| toleration   | 允许进入 |
| nodeSelector | 主动选择 |

------

## Q3：Pod 加 toleration 后一定会调度吗？

不会。

只是：

```text
允许调度
```

不是：

```text
必须调度
```

------

## Q4：为什么 Master 默认不调度 Pod？

因为：

默认存在：

```text
NoSchedule
```

污点。

------

# 三十六、实战案例（生产常见）

------

# 场景：日志系统专属节点

日志系统：

- ES
- Kafka
- Loki

非常吃 IO。

------

## 第一步：打标签

```bash
kubectl label node node1 log=true
```

------

## 第二步：加污点

```bash
kubectl taint node node1 log=true:NoSchedule
```

------

## 第三步：Pod 配置

```yaml
spec:
  tolerations:
  - key: "log"
    operator: "Equal"
    value: "true"
    effect: "NoSchedule"

  nodeSelector:
    log: "true"
```

------

## 结果

只有日志系统：

- 能进入
- 且必须进入

普通业务：

无法占用该节点。

------

# 三十七、总结（核心记忆）

------

# 一句话理解

## 污点

```text
Node 拒绝 Pod
```

## 容忍

```text
Pod 表示我能接受
```

------

# 三种 effect

| effect           | 特点         |
| ---------------- | ------------ |
| NoSchedule       | 禁止新调度   |
| PreferNoSchedule | 尽量不调度   |
| NoExecute        | 驱逐已有 Pod |

------

# 最重要认知

## toleration：

不是选择节点。

只是：

```text
允许进入节点
```

真正指定节点：

还是：

- nodeSelector
- nodeAffinity