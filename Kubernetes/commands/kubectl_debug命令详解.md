# kubectl debug 命令详解

`kubectl debug` 是 Kubernetes 提供的：

```text
在线调试工具
```

用于：

- 进入异常 Pod 排障
- 给运行中的 Pod 临时加调试容器
- 排查网络问题
- 排查文件系统问题
- 排查镜像中没有 shell 的问题
- 排查 CrashLoopBackOff

它本质基于：

```text
Ephemeral Container（临时容器）
```

------

# 一、为什么需要 kubectl debug

------

## 1. 传统 kubectl exec 的问题

例如：

```bash
kubectl exec -it nginx -- sh
```

很多镜像：

- distroless
- scratch
- alpine-mini

里面可能没有：

```text
bash
sh
curl
ping
netstat
tcpdump
```

甚至：

```text
没有 shell
```

导致：

```text
无法排障
```

------

## 2. CrashLoopBackOff 问题

Pod 一直重启：

```text
exec 根本来不及进去
```

------

## 3. kubectl debug 解决方案

动态注入：

```text
调试容器
```

例如：

- busybox
- ubuntu
- netshoot

进入 Pod 内部排障。

------

# 二、kubectl debug 工作原理

------

## 本质

K8s 动态创建：

```text
Ephemeral Container
```

加入 Pod。

------

## 架构

```text
原 Pod
 ├── app container
 └── debug container
```

共享：

- Network Namespace
- IPC Namespace
- PID Namespace（可选）

------

# 三、基本命令

------

## 1. 给 Pod 添加调试容器

```bash
kubectl debug pod-name -it --image=busybox
```

------

### 含义

| 参数    | 说明     |
| ------- | -------- |
| -it     | 交互终端 |
| --image | 调试镜像 |

------

### 示例

```bash
kubectl debug nginx-pod -it --image=busybox
```

进入：

```text
busybox 调试容器
```

------

# 四、推荐调试镜像

------

## 1. busybox

轻量。

适合：

- ls
- cat
- nslookup

------

## 2. Ubuntu

完整 Linux 环境。

------

## 3. netshoot（强烈推荐）

最强网络排障镜像：

```text
nicolaka/netshoot
```

包含：

- tcpdump
- dig
- curl
- netstat
- iproute2
- ping
- traceroute
- ss

------

## 示例

```bash
kubectl debug pod/nginx \
  -it \
  --image=nicolaka/netshoot
```

------

# 五、进入 CrashLoopBackOff Pod

------

## 1. 问题

Pod 不断重启：

```text
kubectl exec 失败
```

------

## 2. 解决方案

```bash
kubectl debug pod-name -it --image=busybox
```

即使业务容器崩溃：

```text
debug container 仍能进入
```

------

# 六、复制 Pod 调试（非常重要）

------

`kubectl debug --copy-to`的作用是：**在不影响原 Pod 的前提下，复制一个一模一样的 Pod 专门用于调试**。

你可以把它理解为：

👉 **“给我克隆一个 Pod，让我随便折腾，原 Pod 照常跑。”**

------

## 一、它到底做了什么？

执行这条命令时：

```
kubectl debug mypod -it --copy-to=mypod-debug --image=busybox
```

Kubernetes 会：

1. **复制原 Pod 的所有配置**

   -    容器镜像

   -    环境变量

   -    Volume 挂载

   -    Labels / Annotations

   -    调度规则（节点选择器、亲和性等）

2. **生成一个新 Pod**

   -    名字是你指定的：`mypod-debug`

   -    与原 Pod **完全独立**

3. **在新 Pod 里做调试**

   -    可以加调试容器

   -    可以直接 `exec`进去

   -    可以随意改配置、重启、试错

4. **原 Pod 不受影响**

   -    继续接收流量

   -    不中断服务 ✅

------

## 二、为什么要用 `--copy-to`？

### ✅ 适合这些场景

| 场景                                | 原因              |
| ----------------------------------- | ----------------- |
| 原 Pod 是**单副本、不能重启**       | 直接调试风险太高  |
| 原 Pod **没 shell / 是 distroless** | 复制后可以换镜像  |
| 想**复现问题但不影响线上**          | 在副本上试        |
| 需要**修改启动命令或环境变量**      | 原 Pod 动不了     |
| 临时容器加不进去                    | 用 copy-to 更稳定 |

------

## 三、和“直接 debug”的区别（重点）

| 对比项                | `kubectl debug pod`（临时容器） | `kubectl debug --copy-to` |
| --------------------- | ------------------------------- | ------------------------- |
| 是否新建 Pod          | ❌ 否                            | ✅ 是                      |
| 原 Pod 是否受影响     | ✅ 无影响                        | ✅ 无影响                  |
| 能否改镜像 / 启动参数 | ❌ 不行                          | ✅ 可以                    |
| 能否复现复杂问题      | ⚠️ 受限                          | ✅ 非常适合                |
| 调试完是否残留        | ⚠️ 临时容器残留在原 Pod          | ✅ 只有新 Pod，可整体删    |

一句话总结：

> **临时容器 = 在原 Pod 里“插一根针”**
>
> **`--copy-to`= 给原 Pod “生个双胞胎”**

------

## 四、一个很常见的实战用法

### 原 Pod 是 distroless / 没 shell，但你要调试

```
kubectl debug mypod \
  --copy-to=mypod-debug \
  --image=nicolaka/netshoot \
  -it -- sh
```

这样你就得到了一个：

- 和 `mypod`**一模一样环境**

- 但 **自带完整调试工具** 的新 Pod

------

## 五、调试完一定要做的事

`--copy-to`创建的是**完整的 Pod**，不会自动删除：

```
kubectl delete pod mypod-debug
```

✅ 原 Pod 仍然安好

✅ 集群不会留下垃圾资源

------

## 六、什么时候不该用 `--copy-to`

- 只是想快速看一眼进程 / 网络 → 用临时容器就够了

- 节点级问题 → 用 `kubectl debug node/xxx`

- 对资源非常敏感的小集群 → 复制 Pod 会占双倍资源

------

如果你愿意，可以告诉我：

👉 **你现在要调试的是什么问题（连不上？起不来？没日志？）**

我可以帮你选最合适的 debug 方式，甚至给你一条“直接可用”的命令。

------

# 七、修改启动命令调试

------

## 示例

```bash
kubectl debug mypod \
  --copy-to=mypod-debug \
  --container=mycontainer \
  -- sh
```

------

## 作用

覆盖：

```text
ENTRYPOINT/CMD
```

方便排障。

------

# 八、共享进程 Namespace

------

你问到了 **`kubectl debug`与「进程共享/PID namespace」的关系**，这是它能“看到原容器进程”的关键机制。下面按模式讲清楚：**什么时候能共享、什么时候不能、怎么让它工作**。

------

## 1. 先给结论（最重要的一句）

> **只有“临时容器模式”（不加 `--copy-to`）才有可能共享原容器的进程；
>
> `--copy-to`复制出来的新 Pod 是“另一个世界”，永远看不到原 Pod 的进程。**

------

## 2. 临时容器模式：`--target`与 PID namespace 共享

### A. 默认前提：Pod 内是否共享 PID namespace？

Kubernetes 有个 Pod 级字段：

```
spec:
  shareProcessNamespace: true   # 默认是 false
```

- **`shareProcessNamespace: true`**：Pod 内所有容器（含以后注入的临时容器）处在同一 PID namespace → 你进任意一个容器都能 `ps aux`看到彼此的进程。

- **`shareProcessNamespace: false`（多数默认情况）**：每个容器有自己的 PID namespace，彼此“默认看不见”。

### B. `kubectl debug -it pod --target=<container>`干了什么？

典型命令：

```
kubectl debug -it your-pod \
  --image=nicolaka/netshoot \
  --target=app   # app 是你的业务容器名
```

这里的 `--target=app`含义是：

> **把临时容器“挂到”目标容器 `app`的 PID namespace（以及通常也同 cgroup/某些 namespace），让你能看见 `app`里跑的进程树。**

所以即使 Pod 没有设置 `shareProcessNamespace: true`，只要：

- 你的集群版本支持 ephemeral containers（≥1.23 基本稳定）

- 你用 `--target`

那你进去后大概率就能：

```
ps aux
# 看到 app 容器的主进程（例如 java/python/nginx 等）
```

还能进一步做：

- `ls /proc/<pid>/fd`

- `cat /proc/<pid>/cmdline`

- 甚至用 `nsenter`切到目标容器的 mount/network namespace 做更深排查（netshoot 很适合干这个）

> ⚠️ 注意：临时容器 **不是** `exec`进原容器，它是“邻居”，但通过 PID namespace 共享拿到了“看进程”的能力。

------

## 3. `--copy-to`为什么“进程一定不在/也不共享”

```
kubectl debug your-pod --copy-to=your-pod-dbg --image=nicolaka/netshoot -it --
```

- 这会生成一个**新 Pod 对象**

- 新 Pod 有：

  -   同样的 volumes/env/网络形态（按 spec 拷贝）

  -   **但跑的是 netshoot 的进程**

- **原 Pod 的进程活在另一个 Pod 的另一个 PID namespace 里，两者不可能互通**

所以它叫“克隆配置/环境”，不叫“克隆运行时”。

------

## 4. 什么时候就算共享也“看不到进程”？常见坑

1. **你没给 `--target`**

   -    临时容器可能落在“自己的 PID namespace”，只能看到自己。

2. **目标容器本身是 pid=1 做了严格 reaper/很小的基础镜像**

   -    你能看到它，但缺少 `ps`/`procfs`工具（所以才用 netshoot）

3. **SELinux / 运行时加固 / 某些沙箱（gVisor/Kata 等）**

   -    可能限制 `/proc`可见性或 `ptrace`，导致即便 PID namespace 共享也“看不全/attach 不了”。

------

## 5. 一张表速记

| 模式                                        | 是否能看到原容器进程 | 关键开关                    |
| ------------------------------------------- | :------------------: | --------------------------- |
| `kubectl debug -it pod --target=X`          |    ✅（设计目标）     | `--target`+ 临时容器注入    |
| 同上，Pod 设了 `shareProcessNamespace:true` |       ✅更容易        | 不用 `--target`也可能互相见 |
| `kubectl debug pod --copy-to=newpod`        |        ❌ 绝不        | 这是另一套 Pod              |

------

## 6. 最小可用“看进程”示例（你照抄就能试）

```
# 假设业务容器名叫 app
kubectl debug -it your-pod \
  --image=nicolaka/netshoot \
  --target=app \
  --
```

进去后先确认：

```
ps aux
ls /proc
```

如果你愿意，把你现在的 **Pod 类型是裸 Pod / Deployment？** 以及 **容器运行时（docker/containerd？）** 说一下，我可以按你的环境把“最可能一次成功的参数组合”精确到你这版 K8s 的行为边界（比如需不需要提前打开 `shareProcessNamespace`）。

------

# 九、节点调试（超重要）

------

`kubectl debug node`是 Kubernetes 提供的**节点级“急救箱”**。它通过在**目标节点上启动一个特权调试 Pod**，让你拥有该节点的“上帝视角”，从而排查单纯在 Pod 层面无法发现的问题。

------

## 一、核心原理：它是怎么工作的？

当你执行：

```
kubectl debug node/my-node -it --image=busybox
```

Kubernetes 会在 `my-node`上创建一个特殊的 Pod（通常命名为 `node-debugger-my-node-xxxxx`）。这个 Pod 的特殊之处在于：

1. **`hostPID: true`**：能看到节点上的所有进程。
2. **`hostNetwork: true`**：使用和节点一样的网络栈（拥有节点 IP）。
3. **`hostIPC: true`**：共享节点的 IPC 命名空间。
4. **挂载节点根文件系统**：通常会将节点的 `/`挂载到容器的 `/host`（或类似路径）。

**一句话总结**：这个 Pod 就像一个**寄生在节点上的特权容器**，它几乎拥有了节点的所有权限，但又是通过 K8s API 管理的。

------

## 二、典型使用场景

| 场景                  | 操作示例                                                     |
| --------------------- | ------------------------------------------------------------ |
| **查看节点日志**      | `ls /host/var/log/pods`或 `journalctl`（如果挂载了 journal 目录） |
| **检查节点网络**      | 使用 `ip addr`, `tcpdump`, `iptables`查看节点层面的网络配置  |
| **排查磁盘/文件系统** | `df -h`, `lsblk`, 检查 `/host`下的挂载点                     |
| **查看节点进程**      | `ps aux`查看 kubelet、容器运行时（containerd/docker）状态    |
| **检查内核参数**      | `sysctl -a`或查看 `/host/proc/sys/`                          |

------

## 三、实战演示：调试一个节点

### 1. 启动调试会话

```
kubectl debug node/worker-node-1 -it --image=nicolaka/netshoot -- sh
```

> **建议使用 `nicolaka/netshoot`**，因为它内置了 `tcpdump`, `iperf`, `curl`等大量网络和系统工具。

### 2. 进入后的操作技巧

进入容器后，你会发现自己在容器内，但节点就在脚下。

- **访问节点文件系统**：

  通常节点根目录被挂载到了 `/host`。

  ```
  chroot /host
  ```

  执行 `chroot /host`后，你就像直接 SSH 登录到了节点一样，可以执行节点上的二进制文件（如 `kubelet`）。

- **不 chroot 直接查看**：

  ```
  # 查看节点上的容器列表（通过 crictl 或 docker）
  /host/usr/bin/crictl ps
  
  # 查看节点 kubelet 日志
  cat /host/var/log/syslog | grep kubelet
  ```

### 3. 退出与清理（非常重要）

退出后，**Pod 不会自动删除**，它会一直占用节点资源（虽然是暂停状态）。

```
# 查看调试 Pod
kubectl get pods -A | grep node-debugger

# 删除它（必须手动）
kubectl delete pod node-debugger-worker-node-1-abcde -n default
```

> **警告**：如果不删除，每次调试都会留下一个“僵尸”Pod，长期累积会污染集群。

------

## 四、对比：Node Debug vs Pod Debug

| 特性                    | `kubectl debug node`       | `kubectl debug pod`         |
| ----------------------- | -------------------------- | --------------------------- |
| **目标**                | 物理机/虚拟机（节点）      | 容器/Pod                    |
| **权限**                | **极高**（接近 root）      | 取决于 Pod 权限             |
| **能否看节点日志**      | ✅ 能                       | ❌ 不能（除非挂载 hostPath） |
| **能否看 kubelet 状态** | ✅ 能                       | ❌ 不能                      |
| **适用场景**            | 节点宕机、网络不通、磁盘满 | 应用崩溃、配置错误          |

------

## 五、替代方案：SSH 登录节点

在过去，大家习惯 SSH 到节点去排查。现在 `kubectl debug node`提供了更好的替代方案：

| 方式                   | 优点                                                         | 缺点                                                         |
| ---------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **SSH**                | 熟悉、直接                                                   | 需要开放 SSH 端口、需要节点 IP、难以审计、可能被安全策略禁止 |
| **kubectl debug node** | **无需节点 IP/SSH**、**操作留痕（审计）**、**符合云原生理念** | 需要集群管理员权限、需要能调度 Pod 到该节点                  |

------

## 六、注意事项

1. **权限要求**：你需要有 `create pods`和 `nodes/debug`的权限，通常需要集群管理员（cluster-admin）权限。
2. **资源消耗**：虽然调试 Pod 通常很快会进入休眠（使用很小的资源），但它毕竟是一个 Pod，会占用 IP 和一点内存。
3. **安全边界**：这个 Pod 权限极大，不要在生产环境中随意用它做破坏性操作（如 `rm -rf`），除非你知道自己在做什么。

如果你现在正遇到某个具体的节点问题（比如 `NotReady`、网络抖动、或者某个节点上的 Pod 都连不上），可以告诉我具体情况，我可以帮你定制一条精准的排查命令。

------

# 十、实际生产排障案例

------

## 1. DNS 排障

------

### 使用 netshoot

```bash
kubectl debug pod/app \
  -it \
  --image=nicolaka/netshoot
```

------

### 测试 DNS

```bash
dig kubernetes.default.svc.cluster.local
```

------

## 2. Service 连通性

------

### 测试

```bash
curl http://service-name
```

------

## 3. 抓包

------

tcpdump

```bash
tcpdump -i eth0
```

------

## 4. 查看端口

```bash
ss -lntp
```

------

## 5. 查看路由

```bash
ip route
```

------

# 十一、Ephemeral Container（临时容器）

------

## 1. 特点

| 特性           | 说明     |
| -------------- | -------- |
| 临时           | 不重启   |
| 不参与 Service | 不接流量 |
| 不影响业务容器 | 仅调试   |
| Pod 删除即消失 | 无状态   |

------

## 2. 查看

```bash
kubectl describe pod
```

查看：

```text
Ephemeral Containers
```

------

# 十二、常见参数

------

| 参数              | 说明     |
| ----------------- | -------- |
| --image           | 调试镜像 |
| --target          | 目标容器 |
| --copy-to         | 复制 Pod |
| -it               | 交互终端 |
| --share-processes | 共享进程 |
| --container       | 指定容器 |

------

# 十三、生产最佳实践

------

## 1. 网络排障统一使用 netshoot

推荐：

```text
nicolaka/netshoot
```

------

## 2. 不要在生产 Pod 安装工具

错误方式：

```bash
apt install curl
```

原因：

- 污染镜像
- 不可审计
- 不可复现

------

## 3. CrashLoopBackOff 优先 debug

而不是：

```text
疯狂重启 Pod
```

------

## 4. 使用 copy-to 调试生产问题

避免：

```text
直接修改线上 Pod
```

------

# 十四、kubectl debug vs exec

| 功能           | exec     | debug      |
| -------------- | -------- | ---------- |
| 进入已有容器   | YES      | YES        |
| 添加新容器     | NO       | YES        |
| CrashLoop 排障 | 差       | 强         |
| 网络工具       | 依赖镜像 | 可动态注入 |
| 节点调试       | NO       | YES        |

------

# 十五、典型生产命令（必须会）

------

## 1. 注入 netshoot

```bash
kubectl debug pod/app \
-it \
--image=nicolaka/netshoot
```

------

## 2. 共享 PID

```bash
kubectl debug pod/app \
-it \
--image=busybox \
--target=app
```

------

## 3. 复制 Pod 调试

```bash
kubectl debug app \
--copy-to=app-debug \
-it \
--image=ubuntu
```

------

## 4. 调试 Node

```bash
kubectl debug node/node1 \
-it \
--image=ubuntu
```

------

# 十六、面试高频问题

------

## 1. kubectl debug 原理？

本质：

```text
Ephemeral Container
```

------

## 2. 为什么 debug 比 exec 强？

因为：

```text
可以动态注入工具容器
```

------

## 3. 如何排查 CrashLoopBackOff？

使用：

```bash
kubectl debug
```

或：

```bash
--copy-to
```

------

## 4. 如何调试 Kubernetes Node？

```bash
kubectl debug node/node1
```

然后：

```bash
chroot /host
```

------

# 十七、总结

------

## kubectl debug 本质

```text
K8s 在线诊断工具
```

------

## 核心能力

| 能力           | 说明 |
| -------------- | ---- |
| 动态注入容器   | 排障 |
| CrashLoop 调试 | 强   |
| 网络排障       | 强   |
| Node 调试      | 强   |
| 不修改业务镜像 | 安全 |

------

## 生产环境最重要组合

```text
kubectl debug
+
nicolaka/netshoot
```

几乎可以解决：

```text
90% K8s 网络问题
```