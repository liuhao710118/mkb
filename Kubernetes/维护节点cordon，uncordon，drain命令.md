在 Kubernetes 中，`cordon` 和 `uncordon` 是节点（Node）维护中最常用的命令之一。

它们的核心作用：

- `cordon`：把节点标记为“不可调度”
- `uncordon`：恢复节点“可调度”

注意：

> cordon 不会驱逐已经运行的 Pod
> 它只是阻止新的 Pod 调度到这个节点。

------

# 一、先理解 Kubernetes 调度机制

Kubernetes 中：

- Pod 不会自己选择节点
- 是 Scheduler（调度器）负责：
  - 找一个合适 Node
  - 把 Pod 放上去

而 Node 有一个属性：

```yaml
spec:
  unschedulable: true/false
```

调度器会检查：

```text
如果 unschedulable=true
→ 不再往这个节点调度新 Pod
```

这就是 cordon 的本质。

------

# 二、cordon 命令详解

命令：

```bash
kubectl cordon <node-name>
```

例如：

```bash
kubectl cordon node1
```

执行后：

```text
node/node1 cordoned
```

------

# 三、cordon 后到底发生什么（非常重要）

很多人误解 cordon。

它：

## 只影响“未来的新 Pod”

不会影响：

- 已运行 Pod
- Deployment
- Service
- 网络
- 存储

------

## 举例理解

假设：

Node1 当前运行：

```text
pod-a
pod-b
pod-c
```

执行：

```bash
kubectl cordon node1
```

结果：

| Pod   | 状态     |
| ----- | -------- |
| pod-a | 继续运行 |
| pod-b | 继续运行 |
| pod-c | 继续运行 |

但是：

以后新建 Pod：

```bash
kubectl apply -f deploy.yaml
```

Scheduler：

```text
不会再把 Pod 调度到 node1
```

会去其它 Node。

------

# 四、查看节点是否被 cordon

命令：

```bash
kubectl get nodes
```

正常：

```text
NAME    STATUS   ROLES
node1   Ready
```

cordon 后：

```text
NAME    STATUS                     ROLES
node1   Ready,SchedulingDisabled
```

重点：

```text
SchedulingDisabled
```

表示：

```text
已经禁止调度
```

但：

```text
Ready
```

说明：

```text
节点还是健康的
```

------

# 五、cordon 的底层原理

cordon 本质：

修改 Node 对象：

```yaml
spec:
  unschedulable: true
```

你可以查看：

```bash
kubectl get node node1 -o yaml
```

会看到：

```yaml
spec:
  unschedulable: true
```

------

# 六、uncordon 命令详解

恢复调度：

```bash
kubectl uncordon node1
```

结果：

```text
node/node1 uncordoned
```

本质：

```yaml
spec:
  unschedulable: false
```

Scheduler 又可以往这里调度 Pod。

------

# 七、典型使用场景（重点）

------

## 场景1：节点维护

最经典。

例如：

- 升级内核
- 升级 Docker
- 升级 kubelet
- 修复磁盘
- 修复网络

操作流程：

### 第一步：禁止新 Pod 调度

```bash
kubectl cordon node1
```

### 第二步：迁移已有 Pod（通常配合 drain）

```bash
kubectl drain node1
```

### 第三步：维护节点

例如：

```bash
reboot
```

### 第四步：恢复调度

```bash
kubectl uncordon node1
```

------

## 场景2：节点资源异常

例如：

- CPU 100%
- 内存爆了
- IO 爆炸
- 网络异常

为了防止：

```text
更多 Pod 被调度进来
```

先：

```bash
kubectl cordon node1
```

避免雪崩。

------

## 场景3：节点即将下线

比如：

- 准备删除机器
- 云主机到期
- 数据中心迁移

先 cordon。

------

# 八、cordon 与 drain 的区别（面试高频）

很多人分不清。

------

## cordon

仅：

```text
禁止新 Pod 调度
```

不会：

```text
驱逐已有 Pod
```

------

## drain

会：

## 1. 自动 cordon

先禁止调度。

## 2. 驱逐已有 Pod

把 Pod 迁移到其它 Node。

------

## 对比表

| 功能            | cordon | drain    |
| --------------- | ------ | -------- |
| 禁止新 Pod 调度 | YES    | YES      |
| 驱逐已有 Pod    | NO     | YES      |
| 适合维护        | 部分   | 完整维护 |
| 风险            | 小     | 较大     |

------

# 九、drain 为什么危险

因为：

drain 会：

```text
删除 Pod
```

然后：

```text
Deployment 再重新创建
```

期间可能：

- 短暂中断
- 服务抖动
- StatefulSet 问题
- 本地数据丢失

所以：

生产环境通常：

```text
先 cordon
观察
再 drain
```

------

# 十、cordon 后哪些 Pod 还能进入节点

很多人以为：

```text
cordon 后任何 Pod 都进不来
```

错。

------

## 1、DaemonSet Pod 仍可能创建

例如：

- kube-proxy
- node-exporter
- cni 插件

因为：

DaemonSet：

```text
无视 unschedulable
```

------

## 2、静态 Pod 不受影响

例如：

```text
/etc/kubernetes/manifests
```

中的 Pod。

因为：

是 kubelet 本地直接创建。

不是 Scheduler 调度。

------

# 十一、cordon 对已有业务有没有影响

正常情况下：

```text
没有影响
```

因为：

已有 Pod：

- 不会重启
- 不会迁移
- 不会删除

所以：

cordon 是非常安全的操作。

------

# 十二、cordon 的高级理解（生产经验）

------

## 1、cordon 是“流量止血”

它像：

```text
关闭入口
```

但：

```text
里面的人不赶走
```

------

## 2、cordon 常用于故障隔离

例如：

某节点：

- 网络丢包
- DNS 异常
- CPU steal 高

你不希望：

```text
更多业务被调度进去
```

立刻：

```bash
kubectl cordon node1
```

------

## 3、cordon 不影响 Service 流量

Service 仍会：

```text
转发到 node1 上的 Pod
```

因为：

Pod 还活着。

------

# 十三、常见误区（重要）

------

## 误区1：cordon 会迁移 Pod

错误。

它不会。

------

## 误区2：cordon 会停止服务

错误。

已有 Pod 正常运行。

------

## 误区3：uncordon 会恢复原来的 Pod

错误。

uncordon 只是：

```text
重新允许调度
```

不会：

```text
把 Pod 调回来
```

------

# 十四、实际生产标准流程（非常重要）

生产维护节点：

## 推荐流程

```bash
# 1 禁止调度
kubectl cordon node1

# 2 驱逐 Pod
kubectl drain node1 --ignore-daemonsets

# 3 维护节点
reboot

# 4 节点恢复
kubectl uncordon node1
```

------

# 十五、drain 常用参数详解

------

## --ignore-daemonsets

因为：

DaemonSet Pod 无法驱逐。

必须加：

```bash
kubectl drain node1 --ignore-daemonsets
```

------

## --delete-emptydir-data

允许删除：

```text
emptyDir
```

数据。

否则 drain 可能失败。

------

## --force

强制驱逐。

用于：

```text
裸 Pod（非控制器管理）
```

------

# 十六、如何模拟实验（建议你亲自练）

你可以：

## 1 查看节点

```bash
kubectl get nodes
```

## 2 cordon

```bash
kubectl cordon node1
```

## 3 再创建 deployment

```bash
kubectl create deployment nginx --image=nginx
```

## 4 查看 Pod 调度

```bash
kubectl get pods -o wide
```

会发现：

```text
不会调度到 node1
```

------

# 十七、面试标准答案（推荐背）

------

## cordon 是什么

```text
cordon 用于将 Kubernetes Node 标记为不可调度状态，
Scheduler 不会再向该节点调度新的 Pod，
但不会影响当前已经运行的 Pod。
```

------

## uncordon 是什么

```text
uncordon 用于恢复节点的可调度状态，
允许 Scheduler 再次向该节点调度 Pod。
```

------

## cordon 和 drain 区别

```text
cordon 仅禁止新 Pod 调度，
drain 会在 cordon 基础上驱逐已有 Pod。
```

------

# 十八、再深入一点（源码级思想）

Scheduler 在调度时：

会过滤：

```text
NodeUnschedulable
```

这个 Filter Plugin。

如果：

```yaml
unschedulable: true
```

节点直接淘汰。

这就是 cordon 的实现核心。