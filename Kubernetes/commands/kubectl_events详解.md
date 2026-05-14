# kubectl events 详解

在 Kubernetes 中：

```bash
kubectl events
```

用于：

```text
查看 Kubernetes 事件（Event）
```

它是 Kubernetes 排障非常重要的命令。

------

# 一、什么是 Event

Event（事件）本质：

```text
Kubernetes 组件产生的运行记录
```

例如：

- Pod 调度失败
- 镜像拉取失败
- 容器重启
- 节点异常
- 探针失败
- PVC 绑定失败

------

# 二、为什么 Event 很重要

很多问题：

```text
logs 看不到
```

例如：

- Pod 根本没启动
- 调度失败
- 镜像拉取失败

这时：

```bash
kubectl events
```

非常关键。

------

# 三、kubectl events 与 describe 区别

很多人混淆。

------

## describe

```bash
kubectl describe pod nginx
```

会显示：

```text
资源详细信息 + Events
```

------

## kubectl events

专门查看：

```text
整个集群事件流
```

更适合：

- 全局排障
- 时间线分析
- 实时观察

------

# 四、最基本用法

------

## 查看事件

```bash
kubectl events
```

输出类似：

```text
LAST SEEN   TYPE      REASON      OBJECT       MESSAGE
10s         Normal    Scheduled   pod/nginx    Successfully assigned
5s          Normal    Pulled      pod/nginx    Container image pulled
```

------

# 五、实时查看事件（非常常用）

------

## 使用 --watch

```bash
kubectl events --watch
```

类似：

```bash
tail -f
```

实时查看。

------

# 六、查看指定 namespace

------

## 示例

```bash
kubectl events -n kube-system
```

------

# 七、按时间排序

------

## 默认可能无序

推荐：

```bash
kubectl events --sort-by=.lastTimestamp
```

------

# 八、查看指定对象事件

------

## 查看 Pod

```bash
kubectl events --for pod/nginx
```

------

## 查看 Deployment

```bash
kubectl events --for deployment/nginx
```

------

## 查看 Node

```bash
kubectl events --for node/master
```

------

# 九、查看 Warning 事件

生产最常用。

------

## 使用类型过滤

```bash
kubectl events --types=Warning
```

------

## 常见 Warning

| 类型             | 含义         |
| ---------------- | ------------ |
| FailedScheduling | 调度失败     |
| BackOff          | 容器反复重启 |
| FailedMount      | 挂载失败     |
| Unhealthy        | 探针失败     |
| FailedPull       | 拉镜像失败   |

------

# 十、Event 字段详解

------

## 示例

```text
LAST SEEN   TYPE      REASON      OBJECT       MESSAGE
```

------

## LAST SEEN

最近发生时间。

------

## TYPE

事件级别。

------

## Normal

正常事件。

------

## Warning

异常事件。

------

## REASON

事件原因。

例如：

- Scheduled
- Pulling
- Failed
- BackOff

------

## OBJECT

关联对象。

例如：

```text
pod/nginx
```

------

## MESSAGE

详细信息。

------

# 十一、最常见事件详解

------

## 1. FailedScheduling

调度失败。

------

## 示例

```text
0/3 nodes are available: insufficient memory
```

含义：

```text
节点资源不足
```

------

## 解决

- 增加节点
- 降低 requests
- 扩容集群

------

# 十二、ImagePullBackOff

------

## Event 示例

```text
Failed to pull image
```

------

## 原因

- 镜像不存在
- 仓库认证失败
- 网络问题

------

## 排查

```bash
kubectl describe pod
```

或者：

```bash
kubectl events --for pod/nginx
```

------

# 十三、CrashLoopBackOff

------

## Event 示例

```text
Back-off restarting failed container
```

------

## 含义

容器：

```text
不断崩溃
```

------

## 排查

```bash
kubectl logs pod --previous
```

------

# 十四、FailedMount

------

## Event 示例

```text
Unable to attach or mount volumes
```

------

## 原因

- PVC 未绑定
- NFS 不通
- 权限问题

------

# 十五、Unhealthy

探针失败。

------

## 示例

```text
Readiness probe failed
```

------

## 原因

- 应用未启动
- 接口异常
- 超时

------

# 十六、NodeNotReady

节点异常。

------

## 示例

```text
Node not ready
```

------

## 可能原因

- kubelet 挂了
- 网络断开
- 磁盘满

------

# 十七、Evicted

Pod 被驱逐。

------

## 示例

```text
The node had condition: DiskPressure
```

------

## 原因

- 内存不足
- 磁盘不足

------

# 十八、OOMKilled

虽然主要是容器状态。

但 Event 也可能出现：

```text
Container killed due to memory
```

------

# 十九、kubectl events 与 logs 配合

生产排障核心流程。

------

## Step1 查看状态

```bash
kubectl get pods
```

------

## Step2 查看 Event

```bash
kubectl events --for pod/nginx
```

------

## Step3 查看 logs

```bash
kubectl logs nginx
```

------

## Step4 查看 describe

```bash
kubectl describe pod nginx
```

------

# 二十、Event 生命周期

注意：

```text
Event 不会永久保存
```

------

## 默认：

```text
约 1 小时
```

（与 apiserver 配置有关）

------

# 二十一、为什么 Event 会丢失

因为：

```text
Event 数量巨大
```

Kubernetes 不适合长期保存。

------

# 二十二、生产 Event 方案

很多公司会：

```text
采集 Kubernetes Event
```

发送到：

- Elasticsearch
- Loki
- Kafka

------

# Event Exporter

常见：

```text
event-exporter
```

------

# 二十三、事件来源

Event 可能来自：

| 组件               | 作用         |
| ------------------ | ------------ |
| kubelet            | 节点事件     |
| scheduler          | 调度事件     |
| controller-manager | 控制器事件   |
| ingress controller | ingress 事件 |
| CSI                | 存储事件     |

------

# 二十四、查看全部 Warning

生产非常高频。

------

## 示例

```bash
kubectl events --types=Warning
```

------

# 实时监控

```bash
kubectl events --types=Warning --watch
```

------

# 二十五、查看节点事件

------

## 示例

```bash
kubectl events --for node/node1
```

------

## 常见：

- Ready
- DiskPressure
- MemoryPressure

------

# 二十六、查看 PVC 事件

------

## 示例

```bash
kubectl events --for pvc/mysql-pvc
```

------

## 常见问题

- Pending
- ProvisioningFailed

------

# 二十七、事件与状态区别

------

## Status

当前状态。

例如：

```text
Running
Pending
```

------

## Event

发生过什么。

例如：

```text
FailedScheduling
Pulled
Started
```

------

# 二十八、完整排障案例

------

## Pod Pending

------

## 查看 Pod

```bash
kubectl get pod
```

状态：

```text
Pending
```

------

## 查看 Event

```bash
kubectl events --for pod/nginx
```

输出：

```text
0/3 nodes available: insufficient cpu
```

------

## 原因

CPU requests 太高。

------

## 解决

修改：

```yaml
resources:
  requests:
    cpu: 100m
```

------

# 二十九、kubectl get events（旧方式）

以前常用：

```bash
kubectl get events
```

------

## 新版本推荐

```bash
kubectl events
```

功能更强。

------

# 三十、生产最佳实践

------

## 1. Warning 必须监控

尤其：

- FailedScheduling
- OOMKilled
- BackOff

------

## 2. Event 接入日志平台

便于审计。

------

## 3. 与 Prometheus 告警联动

非常重要。

------

## 4. 保持时间同步

否则：

```text
事件时间线混乱
```

------

# 三十一、面试高频问题

------

## 1. Event 是什么

Kubernetes 运行事件记录。

------

## 2. Event 与 logs 区别

| Event           | logs     |
| --------------- | -------- |
| Kubernetes 事件 | 程序日志 |

------

## 3. Event 与 describe 区别

| events     | describe   |
| ---------- | ---------- |
| 全局事件流 | 单资源详情 |

------

## 4. Pod Pending 如何排查

重点看：

```bash
kubectl events
```

------

## 5. Event 为什么会丢失

因为：

```text
不会长期保存
```

------

# 三十二、建议继续学习

推荐继续：

1. kubectl describe
2. kubectl top
3. kubectl debug
4. CoreDNS
5. kube-proxy
6. CNI 网络
7. StatefulSet
8. PV/PVC
9. Helm
10. HPA