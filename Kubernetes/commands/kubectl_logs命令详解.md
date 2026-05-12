# kubectl logs 命令详解

在 Kubernetes 中：

```bash
kubectl logs
```

用于：

```text
查看容器标准输出日志（stdout/stderr）
```

这是 Kubernetes 排障最常用命令之一。

------

# 一、为什么 logs 很重要

容器里通常不会写本地日志文件。

而是：

```text
输出到 stdout/stderr
```

例如：

```java
System.out.println()
print()
echo
```

Kubernetes 会收集这些日志。

因此：

```bash
kubectl logs
```

就能查看。

------

# 二、最基本用法

------

## 查看 Pod 日志

```bash
kubectl logs pod-name
```

示例：

```bash
kubectl logs nginx
```

------

# 三、查看指定容器日志

因为：

```text
一个 Pod 可以有多个容器
```

所以需要：

```bash
-c
```

------

## 示例

```bash
kubectl logs nginx-pod -c nginx
```

------

# 四、实时查看日志（最常用）

类似：

```bash
tail -f
```

------

## 使用 -f

```bash
kubectl logs -f nginx
```

实时输出日志。

------

# 五、查看 Deployment 日志

实际上：

```text
Deployment 本身没有日志
```

日志来自 Pod。

------

## 推荐方式

```bash
kubectl logs deploy/nginx
```

或者：

```bash
kubectl logs deployment/nginx
```

Kubernetes 会自动选择 Pod。

------

# 六、查看 ReplicaSet 日志

```bash
kubectl logs rs/nginx
```

------

# 七、查看 previous 日志

非常重要。

用于：

```text
容器崩溃后查看上一次日志
```

------

## 使用 --previous

```bash
kubectl logs nginx --previous
```

或者：

```bash
kubectl logs nginx -p
```

------

# 八、CrashLoopBackOff 必看

当 Pod：

```text
不断重启
```

当前日志可能为空。

这时必须：

```bash
kubectl logs pod-name --previous
```

------

# 九、查看最近日志

------

## tail

```bash
kubectl logs nginx --tail=100
```

只看最后 100 行。

------

# 十、查看指定时间日志

------

## since

```bash
kubectl logs nginx --since=1h
```

查看：

```text
最近 1 小时
```

------

## since-time

```bash
kubectl logs nginx --since-time=2025-01-01T00:00:00Z
```

------

# 十一、查看带时间戳日志

------

## timestamps

```bash
kubectl logs nginx --timestamps
```

输出：

```text
2025-08-01T12:00:00 stdout xxx
```

------

# 十二、多容器 Pod 日志

------

# 查看所有容器

```bash
kubectl logs nginx-pod --all-containers=true
```

------

# 十三、同时查看多个 Pod 日志

Kubernetes 原生命令：

```text
不擅长多 Pod 聚合
```

通常：

- stern
- kubetail
- k9s

更好用。

------

# 十四、通过 Label 查看日志

------

## 示例

```bash
kubectl logs -l app=nginx
```

查看：

```text
所有 app=nginx 的 Pod
```

------

# 十五、限制并发数量

------

## max-log-requests

```bash
kubectl logs -l app=nginx --max-log-requests=10
```

避免同时打开太多流。

------

# 十六、常见日志排障流程

------

# 1. Pod 起不来

先：

```bash
kubectl get pods
```

查看状态。

------

# 2. 查看详细信息

```bash
kubectl describe pod nginx
```

------

# 3. 查看日志

```bash
kubectl logs nginx
```

------

# 4. 如果重启

```bash
kubectl logs nginx --previous
```

------

# 十七、常见错误分析

------

# 1. CrashLoopBackOff

```bash
kubectl logs pod --previous
```

重点看：

- Java 异常
- 配置错误
- 端口冲突

------

# 2. ImagePullBackOff

logs 通常没用。

因为：

```text
容器根本没启动
```

需要：

```bash
kubectl describe pod
```

看 Events。

------

# 3. OOMKilled

日志最后可能：

```text
Killed
```

或者：

```text
OutOfMemory
```

------

# 十八、日志来源原理

容器运行时：

- containerd
- Docker
- CRI-O

会把：

```text
stdout/stderr
```

写入：

```text
/var/log/containers/
```

------

# 十九、节点日志位置

通常：

```bash
/var/log/pods/
```

或者：

```bash
/var/log/containers/
```

------

# 二十、日志系统架构

生产环境：

```text
kubectl logs
```

仅适合：

- 临时排障
- 小规模查看

------

# 大规模生产：

通常：

```text
Fluentd
FluentBit
Filebeat
Vector
```

采集。

发送到：

- Elasticsearch
- Loki
- Kafka

------

# 二十一、日志最佳实践

------

# 1. 不写文件

推荐：

```text
stdout/stderr
```

------

# 2. 使用 JSON 日志

方便 ELK 检索。

------

# 3. 日志加 traceId

方便链路追踪。

你之前做的：

```text
SkyWalking
```

就是典型。

------

# 4. 区分日志级别

```text
INFO
WARN
ERROR
```

------

# 二十二、生产常用命令

------

# 实时日志

```bash
kubectl logs -f pod-name
```

------

# 查看最近 200 行

```bash
kubectl logs pod-name --tail=200
```

------

# 查看上次崩溃日志

```bash
kubectl logs pod-name -p
```

------

# 查看所有容器

```bash
kubectl logs pod-name --all-containers
```

------

# 查看 deployment

```bash
kubectl logs deploy/nginx
```

------

# 查看最近 10 分钟

```bash
kubectl logs pod-name --since=10m
```

------

# 带时间戳

```bash
kubectl logs pod-name --timestamps
```

------

# 二十三、kubectl logs 与 describe 区别

| 命令     | 作用               |
| -------- | ------------------ |
| logs     | 看程序输出         |
| describe | 看 Kubernetes 事件 |

------

# 举例

------

## 程序报错

看：

```bash
kubectl logs
```

------

## 调度失败

看：

```bash
kubectl describe
```

------

# 二十四、日志轮转

Kubernetes 会自动轮转日志。

参数：

```text
containerLogMaxSize
containerLogMaxFiles
```

由 kubelet 控制。

------

# 二十五、日志权限问题

有时：

```bash
kubectl logs
```

失败：

```text
Forbidden
```

因为：

```text
RBAC 权限不足
```

需要：

```text
pods/log
```

权限。

------

# 二十六、完整排障案例

------

# Pod 状态异常

```bash
kubectl get pods
```

输出：

```text
CrashLoopBackOff
```

------

# 查看日志

```bash
kubectl logs nginx
```

无内容。

------

# 查看 previous

```bash
kubectl logs nginx --previous
```

发现：

```text
java.lang.OutOfMemoryError
```

------

# 解决

修改：

```yaml
resources:
  limits:
    memory: 1Gi
```

------

# 二十七、面试高频问题

------

# 1. kubectl logs 看什么

查看：

```text
stdout/stderr
```

------

# 2. 为什么 logs 没内容

可能：

- 容器没启动
- 已崩溃
- 日志写文件了

------

# 3. CrashLoopBackOff 如何排查

```bash
kubectl logs --previous
```

------

# 4. logs 与 describe 区别

| logs     | describe        |
| -------- | --------------- |
| 程序日志 | Kubernetes 事件 |

------

# 5. 为什么生产不用 kubectl logs

因为：

```text
无法集中化
无法长期保存
无法检索分析
```

所以：

```text
ELK / Loki
```

更常用。

------

# 二十八、建议继续学习

下一步推荐：

1. kubectl exec
2. kubectl describe
3. kubectl top
4. Service
5. Ingress
6. ConfigMap
7. Secret
8. StatefulSet
9. CNI
10. Helm