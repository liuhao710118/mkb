# kubectl exec 命令详解

在 Kubernetes 中：

```bash
kubectl exec
```

用于：

```text
进入容器执行命令
```

这是 Kubernetes 排障最核心命令之一。

类似于：

```bash
docker exec
```

------

# 一、kubectl exec 能做什么

常见用途：

- 进入容器
- 查看文件
- 检查环境变量
- 测试网络
- 查看进程
- 调试应用
- 手动执行命令

------

# 二、最常用命令

------

## 进入容器 Shell

```bash
kubectl exec -it nginx -- /bin/bash
```

如果没有 bash：

```bash
kubectl exec -it nginx -- /bin/sh
```

------

# 三、命令参数详解

------

# 1. exec

表示：

```text
执行命令
```

------

# 2. -i

interactive

保持标准输入。

允许输入命令。

------

# 3. -t

tty

分配终端。

否则：

```text
无法交互
```

------

# 4. --

非常重要。

表示：

```text
后面是容器内部命令
```

例如：

```bash
kubectl exec nginx -- ls /
```

------

# 四、执行单条命令

不进入 Shell。

------

## 查看目录

```bash
kubectl exec nginx -- ls /
```

------

## 查看环境变量

```bash
kubectl exec nginx -- env
```

------

## 查看进程

```bash
kubectl exec nginx -- ps -ef
```

------

## 查看端口

```bash
kubectl exec nginx -- netstat -tnlp
```

有些镜像没有：

```text
netstat
```

可以：

```bash
ss -tnlp
```

------

# 五、进入多容器 Pod

因为：

```text
一个 Pod 可以有多个容器
```

需要：

```bash
-c
```

指定容器。

------

## 示例

```bash
kubectl exec -it nginx-pod -c nginx -- bash
```

------

# 六、进入 Deployment 的 Pod

实际上：

```text
Deployment 本身不能 exec
```

因为：

```text
exec 进入的是 Pod
```

------

# 正确方式

先查看：

```bash
kubectl get pods
```

再：

```bash
kubectl exec -it pod-name -- bash
```

------

# 七、常见容器 Shell

------

# Linux 常见

------

## bash

```bash
/bin/bash
```

------

## sh

```bash
/bin/sh
```

------

## ash（Alpine）

```bash
/bin/ash
```

------

# 八、为什么很多容器没有 bash

因为：

```text
生产镜像通常极简
```

例如：

- Alpine
- Distroless

减少：

- 体积
- 漏洞
- 攻击面

------

# 九、调试网络（非常重要）

------

# 测试 DNS

```bash
kubectl exec nginx -- nslookup kubernetes.default
```

------

# 测试 Service

```bash
kubectl exec nginx -- curl http://svc-name
```

------

# 测试 Pod 网络

```bash
kubectl exec nginx -- ping 10.244.1.5
```

------

# 十、查看容器文件

------

## 查看配置

```bash
kubectl exec nginx -- cat /etc/nginx/nginx.conf
```

------

## 查看日志文件

```bash
kubectl exec nginx -- tail -f /app/logs/app.log
```

注意：

```text
生产不推荐写文件日志
```

推荐：

```text
stdout/stderr
```

------

# 十一、查看资源使用

------

## 查看内存

```bash
kubectl exec nginx -- free -m
```

------

## 查看磁盘

```bash
kubectl exec nginx -- df -h
```

------

## 查看 CPU

```bash
kubectl exec nginx -- top
```

------

# 十二、数据库操作

------

# MySQL

```bash
kubectl exec -it mysql -- mysql -uroot -p
```

------

# Redis

```bash
kubectl exec -it redis -- redis-cli
```

------

# Kafka

```bash
kubectl exec -it kafka -- kafka-topics.sh
```

你之前做过：

```text
Kafka + SkyWalking
```

这种场景经常需要 exec 排障。

------

# 十三、Java 应用排障

非常高频。

------

# 查看 JVM 参数

```bash
kubectl exec java-app -- jps
```

------

# 查看线程

```bash
kubectl exec java-app -- jstack 1
```

------

# 查看堆

```bash
kubectl exec java-app -- jmap -heap 1
```

------

# 查看 GC

```bash
kubectl exec java-app -- jstat -gc 1
```

------

# 十四、临时安装工具

很多镜像：

```text
没有 curl
没有 ping
没有 netstat
```

------

# Alpine

```bash
apk add curl
```

------

# Debian/Ubuntu

```bash
apt update
apt install curl
```

注意：

```text
容器重建后会丢失
```

仅临时调试。

------

# 十五、kubectl exec 工作原理

流程：

```text
kubectl
    ↓
apiserver
    ↓
kubelet
    ↓
container runtime
    ↓
container process
```

最终：

```text
进入容器 namespace
```

------

# 十六、exec 与 attach 区别

| 命令   | 作用       |
| ------ | ---------- |
| exec   | 新执行命令 |
| attach | 连接主进程 |

------

# attach 示例

```bash
kubectl attach pod-name
```

通常：

```text
不如 exec 常用
```

------

# 十七、kubectl cp

和 exec 配套。

------

# 从 Pod 拷贝文件

```bash
kubectl cp nginx:/tmp/a.log ./a.log
```

------

# 拷贝到 Pod

```bash
kubectl cp ./test.txt nginx:/tmp/
```

------

# 十八、exec 常见错误

------

# 1. container not found

```text
Pod 多容器
```

需要：

```bash
-c
```

------

# 2. bash not found

因为：

```text
镜像没有 bash
```

改用：

```bash
/bin/sh
```

------

# 3. Forbidden

RBAC 权限不足。

需要：

```text
pods/exec
```

权限。

------

# 4. unable to upgrade connection

通常：

- apiserver 问题
- kubelet 问题
- 网络问题

------

# 十九、生产最佳实践

------

# 1. 尽量少进生产容器

原因：

```text
破坏不可变基础设施
```

------

# 2. 使用只读根文件系统

提高安全。

------

# 3. 禁止 root

```yaml
securityContext:
  runAsUser: 1000
```

------

# 4. 使用 debug Pod

推荐：

```bash
kubectl debug
```

而不是直接修改业务容器。

------

# 二十、kubectl debug（新方式）

现代 Kubernetes 推荐：

```bash
kubectl debug
```

------

# 示例

```bash
kubectl debug -it nginx --image=busybox
```

创建：

```text
临时调试容器
```

------

# 二十一、完整排障案例

------

# Pod 无法访问 MySQL

------

## 进入容器

```bash
kubectl exec -it app -- sh
```

------

## 测试 DNS

```bash
nslookup mysql
```

失败。

------

## 查看 CoreDNS

```bash
kubectl get pods -n kube-system
```

发现：

```text
coredns CrashLoopBackOff
```

------

## 查看日志

```bash
kubectl logs coredns -n kube-system
```

定位问题。

------

# 二十二、生产高频命令

------

# 进入容器

```bash
kubectl exec -it pod -- sh
```

------

# 查看环境变量

```bash
kubectl exec pod -- env
```

------

# 查看进程

```bash
kubectl exec pod -- ps -ef
```

------

# 查看端口

```bash
kubectl exec pod -- ss -tnlp
```

------

# 测试 HTTP

```bash
kubectl exec pod -- curl http://svc
```

------

# 测试 DNS

```bash
kubectl exec pod -- nslookup kubernetes.default
```

------

# 查看文件

```bash
kubectl exec pod -- cat /tmp/a.txt
```

------

# 二十三、exec 与 logs 区别

| 命令 | 作用     |
| ---- | -------- |
| logs | 查看日志 |
| exec | 进入容器 |

------

# logs 是：

```text
被动查看
```

------

# exec 是：

```text
主动调试
```

------

# 二十四、面试高频问题

------

# 1. kubectl exec 原理

通过：

```text
apiserver → kubelet → runtime
```

进入容器 namespace。

------

# 2. 为什么需要 ---

区分：

```text
kubectl 参数
和
容器命令
```

------

# 3. 为什么 bash 不存在

因为：

```text
生产镜像极简化
```

------

# 4. exec 与 attach 区别

| exec   | attach     |
| ------ | ---------- |
| 新进程 | 连接主进程 |

------

# 5. exec 为什么危险

因为：

- 可修改容器
- 可删除文件
- 可泄露数据

因此生产需要 RBAC 控制。

------

# 二十五、建议下一步学习

推荐继续：

1. kubectl describe
2. kubectl top
3. kubectl debug
4. Service
5. Ingress
6. ConfigMap
7. Secret
8. StatefulSet
9. CNI 网络
10. Helm