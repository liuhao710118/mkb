# Kubernetes Pod 生命周期详解（原理 + 时序 + 排障要点）

## 1. 从“外部可见状态”看：Pod Phase（阶段）

Pod 的 **Phase** 只是一个高层汇总状态（不适合做精细判断），常见取值与含义：

| Phase       | 含义                                                         |
| ----------- | ------------------------------------------------------------ |
| `Pending`   | API 已接受，但尚未完成调度/镜像拉取/启动前置条件（仍在初始化） |
| `Running`   | Pod 已被**调度到某节点**，且至少**一个容器还在运行**（但不代表“服务可用”） |
| `Succeeded` | 所有容器**正常退出（exit 0. **，且不会重启（典型：Job）      |
| `Failed`    | 至少一个容器以**非 0 退出**，或被系统“失败处理”（Evicted / 容器无法启动等） |
| `Unknown`   | 节点失联，apiserver 拿不到状态（通信/心跳问题）              |

> 实际排障更该看：`Pod.status.conditions`、`Pod.status.containerStatuses`、事件（`kubectl describe pod`）。

------

## 2. Pod 的完整生命周期：从 YAML 到消失（关键时序）

### A. 准入 / 写入 API（etcd）

1. `kubectl apply/create`→ APIServer 认证/鉴权/RuntimeClass/Mutating+Validating Webhook → 写入 etcd

2. Pod 对象刚出现：**Phase=Pending**，通常还没有 `nodeName`

此时能看到一些 **Conditions** 开始变化（后面会说）。

------

### B. 调度（Scheduler）

- Scheduler 发现 `spec.nodeName == ""`的 Pod，按 **requests/limits、亲和性、污点与容忍、拓扑约束、volume 限制** 等算分，选出节点并写回 `nodeName`。

- 若暂时不可调度：Pod 会卡在 `Pending`，事件里常见：

  -   `0/3 nodes are available: ...`

  -   `Insufficient cpu/memory`

  -   `node(s) had taint ...`

> 调度完成 ≠ 容器启动；只是“这个 Pod 归属某个 kubelet”。

------

### C. 节点侧：Kubelet 把 Pod 变成“运行时”

当 kubelet 发现自己名下有新 Pod，会走一个**顺序执行链**（非常关键）：

#### C1. 准备“环境层”

大致顺序（实现层面可能合并/并行部分步骤，但逻辑依赖不变）：

1. **准入检查/预准备**：checkNodeCondition、内存阈值、磁盘阈值（会影响后续 Eviction）

2. **卷挂载准备（VolumeMount/CSI）**：secret/configMap/PV→stage/publish，等存储就绪

3. **获取/拉镜像**（按 `imagePullPolicy`+ image 是否存在）

4. **创建 sandbox（Pod 沙箱）**：通常是 pause/infra 容器建好 **network namespace（CNI）**

5. **设置 DNS / sysctl 等**

#### C2. Init Containers（严格串行）

`spec.initContainers`**按顺序执行，前一个成功(exit 0)才跑下一个**：

- 任何 initContainer 失败 → 按 `restartPolicy`（大多是 Always/OnFailure）决定是否重试

- 在 init 跑完之前：

  -   **主容器不会启动**

  -   Pod 可能长期 `Init:N/M`、`CrashLoopBackOff`（init 里）

常见 init 用途：等待依赖、迁移/初始化数据、模板配置。

#### C3. 主容器（containers）启动 + 探针开始工作

当 init 全成功：

- kubelet 启动主容器（并行/按 runtime 实现）

- 开始评估你配置的探针：

  -   `startupProbe`：存在时 **先接管“启动保护”**（失败就杀容器重启），防止慢启动被 liveness 误杀

  -   `readinessProbe`：决定 **Ready 与否**（影响 Service 是否转发流量）

  -   `livenessProbe`：失败 → **杀死容器触发重启**（不关心 Ready）

------

### D. 运行期：状态字段怎么映射成你看到的文字

你在 `kubectl get pods`看到的花式状态，来自这些字段的组合：

- **Pod Phase = Running**，但容器可能没 Ready、可能 CrashLoopBackOff

- 每个容器状态 `state`只有 3 种根形态：

  -   `waiting`（镜像拉不到、CreateContainerConfigError、Back-off restart…）

  -   `running`

  -   `terminated`（exitCode + reason + signal）

常用 reason 关键词：

- `ContainerCreating`/ `ImagePullBackOff`/ `ErrImagePull`

- `CrashLoopBackOff`（不是“崩溃原因”，而是**指数退避冷却**）

- `OOMKilled`（资源 limits.memory 触发 cgroup OOM）

- `Completed`（容器正常退出）

------

### E. 终止（Graceful Shutdown）— 最容易踩坑的一段

当你删除 Pod（rollout、hpa、手动 delete、节点 drain 等），流程是：

1. **DELETE Pod** → apiserver 标记删除（开始计时）

2. 如果 Pod 有 `metadata.deletionTimestamp`且被控制器管着（Deployment/Job/StatefulSet），控制器通常会**先建新 Pod 再删旧**（滚动更新策略控制）

3. Kubelet 执行终止序列（关键）：

   1.    **把 Pod 从 Service 端点摘除（EndpointSlice/Endpoints ready=false）**：尽快停止新流量（但不是“瞬时保证”）

   2.    对每个容器：

      -       若定义了 `lifecycle.preStop`：**先跑 preStop**（http/exec/sleep）

        -         preStop **计入 grace period**，别在里面跑无限阻塞

      -       然后发 **SIGTERM** 给 PID 1

      -       **等最多 `spec.terminationGracePeriodSeconds`（默认 30. **

      -       超时未退出 → **SIGKILL**

4. 容器全死 → 清理网络/卷（volume 卸载）→ Pod 对象最终 GC 消失

一句话：**SIGTERM 不等于“立刻退出”，你必须让进程处理 SIGTERM 并在 grace period 内退出**。

------

### F. 重启规则：什么时候会重启？

由 **Pod 级别 `restartPolicy`（Always / OnFailure / Never）** + 容器退出原因决定：

- 绝大多数工作负载（Deploy/RS）给的是 `restartPolicy: Always`

- 容器异常退出 → kubelet **重启容器**（有指数退避，显示为 `CrashLoopBackOff`）

- Job 类常用 `OnFailure/Never`，最终走到 `Succeeded/Failed`

> 注意：重启的是“容器”，不是重建 Pod（直到某些事件/驱逐/节点问题才会换 Pod）。

------

## 3. Pod Conditions：判断“为什么还没 Ready”更靠谱

`Pod.status.conditions`常见类型（不同版本实现会有微调）：

| Condition         | True 含义                             |
| ----------------- | ------------------------------------- |
| `PodScheduled`    | 已分配到节点                          |
| `PodInitialized`  | initContainers 全部成功               |
| `ContainersReady` | 容器级 ready（主要看 readinessProbe） |
| `Ready`           | Pod 可被 Service 转发（综合结果）     |

看它们为 False 时的 `reason/message`，常能直接定位卡点（init 失败、拉镜像、probe not ready）。

------

## 4. 最常见的“终态/半终态”速查

| 你看到的表象                        | 优先看什么                                                   |
| ----------------------------------- | ------------------------------------------------------------ |
| `Pending`很久                       | events：资源不足/taint/affinity/pvc pending                  |
| `Init:0/1`/ `Init:CrashLoopBackOff` | initContainer logs：`kubectl logs pod -c <init-name>`        |
| `ImagePullBackOff`                  | image 名/tag/secret(harbor)/node 网络连通                    |
| `CrashLoopBackOff`（主容器）        | 先看 `kubectl logs pod --previous`，再看 liveness 是否太严   |
| `OOMKilled`                         | `resources.limits.memory`太小 / JVM 没配 `-Xmx`              |
| `Running`但 `0/1 Ready`             | readinessProbe 失败；或者端口没监听/路径返回非 2xx           |
| `Failed / Evicted`                  | 事件是王道：`kubectl get event --for pod/...`，多半磁盘/内存压力/节点资源 |
| `Unknown`                           | 节点 kubelet/apiserver 网络或 kubelet 挂了                   |

------

## 5. 一张“时间线”总结（面试/排障最常用）

```
kubectl apply
  → Pending(调度队列)
    → Scheduled(写 nodeName)
      → 拉镜像 / Volume/CSI / CNI (infra sandbox)
        → InitContainers 串行
          → 主容器启动
            → startupProbe(如有) 保护期
              → readinessProbe 通过 → Pod Ready=true(进 Service)
              → livenessProbe 失败 → kill容器 → 重启
        ← DELETE Pod
          → preStop → SIGTERM → grace period → SIGKILL
            → unmount volumes → removePod → GC
```

------

