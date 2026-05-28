# 一、DaemonSet 是什么

在 Kubernetes 中：

```text
Deployment：保证有多少个 Pod
StatefulSet：保证有状态 Pod
DaemonSet：保证每个节点都有一个 Pod
```

DaemonSet（守护进程集）最核心的作用：

> 在集群中的每个 Node 上运行一个 Pod

或者：

> 在符合条件的每个 Node 上运行且仅运行一个 Pod

------

# 二、为什么需要 DaemonSet

很多系统组件：

```text
必须每个节点都运行一份
```

例如：

- 日志采集
- 监控
- 网络插件
- 存储插件
- kube-proxy

这些组件：

```text
不是业务 Pod
而是节点级别的“守护进程”
```

所以：

Kubernetes 提供：

```text
DaemonSet
```

来管理它们。

------

# 三、最经典的 DaemonSet 场景

------

## 1、日志采集

例如：

- Fluentd
- Filebeat
- Vector

因为：

每个节点都有：

```text
/var/log
```

必须：

```text
每台机器部署一个日志采集器
```

------

## 2、监控 Agent

例如：

- node-exporter
- cadvisor
- Datadog Agent

因为：

每个节点：

- CPU
- 内存
- 磁盘
- 网络

都不同。

所以：

```text
每个 Node 都必须运行一个监控 Pod
```

------

## 3、网络插件（非常重要）

例如：

- Calico
- Flannel
- Cilium

因为：

每个节点都需要：

```text
网络转发能力
```

所以：

网络组件几乎全是 DaemonSet。

------

## 4、存储插件

例如：

- CSI Node Plugin

因为：

每个节点都需要：

```text
挂载卷
```

------

# 四、DaemonSet 的核心特点（非常重要）

------

## 特点1：每个节点一个 Pod

假设：

集群：

```text
node1
node2
node3
```

DaemonSet：

```text
自动创建：

pod-node1
pod-node2
pod-node3
```

------

## 特点2：新增节点自动创建

如果：

后来新增：

```text
node4
```

DaemonSet 会自动：

```text
创建 pod-node4
```

无需手工操作。

------

## 特点3：删除节点自动删除

如果：

```text
node2 被移除
```

对应 Pod 也会自动删除。

------

## 特点4：不能随意扩缩容

Deployment：

```bash
kubectl scale deployment
```

可以扩容。

但 DaemonSet：

```text
副本数 = 节点数
```

不能直接指定 replicas。

------

# 五、DaemonSet 工作原理（重点）

普通 Pod：

```text
Scheduler 负责调度
```

而 DaemonSet：

```text
DaemonSet Controller 负责
```

它会：

### 1、监控 Node

发现：

```text
哪个节点没有 Pod
```

### 2、自动创建 Pod

并绑定：

```yaml
nodeName: xxx
```

直接指定节点。

------

# 六、DaemonSet 创建示例

最简单例子：

```yaml
apiVersion: apps/v1
kind: DaemonSet

metadata:
  name: node-exporter

spec:
  selector:
    matchLabels:
      app: node-exporter

  template:
    metadata:
      labels:
        app: node-exporter

    spec:
      containers:
      - name: exporter
        image: prom/node-exporter
```

------

# 七、创建后会发生什么

假设：

集群：

```text
3 个节点
```

则自动：

```text
生成 3 个 Pod
```

查看：

```bash
kubectl get pods -o wide
```

可能：

```text
NAME                  NODE
node-exporter-abc     node1
node-exporter-def     node2
node-exporter-ghi     node3
```

------

# 八、DaemonSet 与 Deployment 的本质区别（面试高频）

------

## Deployment

目标：

```text
保证 Pod 数量
```

例如：

```text
我要 3 个 nginx
```

至于在哪：

```text
Scheduler 决定
```

------

## DaemonSet

目标：

```text
保证每个节点都有一个 Pod
```

不是固定副本数。

------

## 对比表（重点）

| 对比项   | Deployment    | DaemonSet            |
| -------- | ------------- | -------------------- |
| 管理目标 | Pod 数量      | 节点覆盖             |
| Pod 数量 | 固定 replicas | 等于节点数           |
| 调度方式 | Scheduler     | DaemonSet Controller |
| 典型场景 | 业务服务      | 节点守护进程         |
| 新节点   | 不一定部署    | 自动部署             |

------

# 九、DaemonSet 如何选择节点

默认：

```text
所有 Node
```

都部署。

但可以限制。

------

## 方法1：nodeSelector

例如：

```yaml
spec:
  template:
    spec:
      nodeSelector:
        disk: ssd
```

表示：

```text
只部署到带 disk=ssd 标签的节点
```

------

## 方法2：nodeAffinity（更高级）

例如：

```yaml
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

表示：

```text
只部署 GPU 节点
```

------

## 方法3：污点容忍（非常重要）

很多系统节点：

有污点：

```text
NoSchedule
```

普通 Pod 无法进入。

但系统 DaemonSet：

需要运行。

所以：

必须：

```yaml
tolerations:
- operator: Exists
```

------

# 十、为什么 DaemonSet 经常有 tolerations

例如：

Master 节点：

默认：

```text
NoSchedule
```

普通 Pod 不允许进入。

但：

```text
kube-proxy
```

必须运行。

因此：

DaemonSet 通常：

```yaml
tolerations:
- effect: NoSchedule
  operator: Exists
```

这样：

```text
即使节点有污点
也允许部署
```

------

# 十一、DaemonSet 与 cordon 的关系（高频）

你前面问过 cordon。

这里特别重要。

------

## 普通 Pod

cordon 后：

```text
不会再调度
```

------

## DaemonSet Pod

即使节点：

```text
SchedulingDisabled
```

DaemonSet 仍可能创建 Pod。

因为：

```text
DaemonSet 不依赖普通 Scheduler
```

这也是：

为什么：

```text
node-exporter
```

在 cordon 节点仍存在。

------

# 十二、DaemonSet 更新机制（重点）

------

## RollingUpdate

默认滚动更新。

例如：

```text
升级 node-exporter 镜像
```

会：

```text
逐节点更新
```

不是一次全删。

------

## 示例

```yaml
updateStrategy:
  type: RollingUpdate
```

------

## 更新过程

例如：

```text
node1 更新完成
→ node2 更新
→ node3 更新
```

------

## OnDelete

另一种：

```yaml
updateStrategy:
  type: OnDelete
```

表示：

```text
不会自动更新
```

只有：

```text
Pod 被删
```

才会重建新版本。

------

# 十三、DaemonSet 删除会发生什么

删除：

```bash
kubectl delete ds node-exporter
```

结果：

```text
所有相关 Pod 全部删除
```

因为：

Pod 属于 DaemonSet。

------

# 十四、实际生产中的典型 DaemonSet

------

## kube-proxy

负责：

```text
Service 转发
iptables/ipvs
```

每节点一个。

------

## Calico Node

负责：

```text
容器网络
```

每节点一个。

------

## node-exporter

负责：

```text
Prometheus 节点监控
```

每节点一个。

------

## Fluentd/Filebeat

负责：

```text
日志采集
```

每节点一个。

------

# 十五、DaemonSet 常见问题（非常重要）

------

## 问题1：为什么 Pod 没有部署到所有节点

通常：

### 原因1：节点标签不匹配

```yaml
nodeSelector
```

限制了。

------

### 原因2：污点没容忍

节点：

```text
NoSchedule
```

但没有：

```yaml
tolerations
```

------

### 原因3：节点 NotReady

DaemonSet 不会部署到异常节点。

------

## 问题2：为什么 cordon 后还能创建 Pod

因为：

```text
DaemonSet 绕过普通调度器
```

这是特性。

------

## 问题3：DaemonSet Pod 可以多个吗

默认：

```text
每节点一个
```

但：

多个不同 DaemonSet：

```text
可以同时存在
```

------

# 十六、静态 Pod 与 DaemonSet 区别（高频）

很多人混。

------

## 静态 Pod

由：

```text
kubelet 本地直接创建
```

配置文件：

```text
/etc/kubernetes/manifests
```

------

## DaemonSet

由：

```text
APIServer + Controller
```

统一管理。

------

## 对比

| 对比               | 静态 Pod | DaemonSet  |
| ------------------ | -------- | ---------- |
| 创建者             | kubelet  | Controller |
| 是否经过 APIServer | 间接     | 是         |
| 是否自动覆盖节点   | 否       | 是         |
| 管理方式           | 本地文件 | YAML       |

------

# 十七、DaemonSet 生命周期

------

## 创建

Controller 监听 Node。

------

## 调度

为每个 Node 创建 Pod。

------

## 节点新增

自动补 Pod。

------

## 节点删除

自动删 Pod。

------

## 更新

滚动替换。

------

## 删除 DS

全部删除。

------

# 十八、生产经验（很重要）

------

## 1、DaemonSet 非常适合 Agent

例如：

- 安全 Agent
- 监控 Agent
- 日志 Agent

因为：

```text
天然一机一实例
```

------

## 2、DaemonSet 容易占资源

因为：

```text
每个节点都部署
```

100 节点：

```text
= 100 Pod
```

所以：

资源请求必须合理。

------

## 3、DaemonSet 故障影响全局

例如：

错误配置：

```text
所有节点网络全挂
```

因为：

网络插件本身是 DaemonSet。

所以：

更新必须谨慎。

------

# 十九、面试标准答案（建议背）

------

## 什么是 DaemonSet

```text
DaemonSet 是 Kubernetes 中的一种工作负载资源，
用于保证集群中的每个节点（或指定节点）
都运行一个 Pod 副本。
```

------

## DaemonSet 的典型场景

```text
日志采集、监控 Agent、网络插件、存储插件等。
```

------

## DaemonSet 与 Deployment 区别

```text
Deployment 管理 Pod 副本数，
DaemonSet 管理节点覆盖率，
通常每个节点运行一个 Pod。
```

------

## 为什么 cordon 后 DaemonSet 还能调度

```text
因为 DaemonSet 不依赖普通 Scheduler，
它会直接为节点创建 Pod。
```