## StatefulSet 的 Rollout & 回滚 —— 它和 Deployment 完全是两套逻辑

### 一句话抓住本质

> **Deployment 的 rollout 靠的是"换 ReplicaSet"；StatefulSet 的 rollout 靠的是"换 `.spec.template`，然后按序数从后往前挨个删掉旧 Pod、让控制器按新模板重建"，历史版本则存在一个叫 `ControllerRevision`的资源里。**

------

## 1) 触发 rollout 的条件（和 Deployment 一样窄）

StatefulSet **只关心一个东西有没有变**：

```
.spec.template   ← Pod 模板（镜像、env、resource、volumeMounts…）
```

只有改到 `spec.template`里的内容才算"一次发布"，才会触发滚动更新。`replicas`扩缩容不算 rollout；改 `volumeClaimTemplates`也不算（而且 PVC 不会自动重建/迁移，这是个经典坑）。

------

## 2) 更新策略：`RollingUpdate`vs `OnDelete`

```
spec:
  updateStrategy:
    type: RollingUpdate   # 默认
    rollingUpdate:
      maxUnavailable: 1   # 1.7+ 支持，默认 1（但不是"并行"，见下文）
```

| type                        | 行为                                                         |
| --------------------------- | ------------------------------------------------------------ |
| **`RollingUpdate`**（默认） | 改 `spec.template`后，控制器**自动**开始更新                 |
| **`OnDelete`**              | 改模板后**什么都不做**，你必须手动 `kubectl delete pod sts-name-2`才会用新模板重建那个 Pod |

**`RollingUpdate`的更新顺序是最关键的区别：**

```
web-2 ↓ 先删/重建等 Ready
web-1 ↓ 再处理
web-0   最后
```

也就是：**从最高序数 → 最低序数（reverse ordinal order）**，每个 Pod 必须 `Running && Ready`之后才碰下一个。

这和 Deployment 的"并行滚动"完全不同——StatefulSet 强调**有序性**，因为它假设你的有状态应用（mysql 主从、zk、kafka…）需要按拓扑顺序升级。

------

## 3) 版本历史靠什么存：`ControllerRevision`

每次你改 `spec.template`，StatefulSet 控制器会把这个模板的**快照**序列化后塞进一个 API 对象：

```
kubectl get controllerrevisions
# NAME              CONTROLLER              REVISION   AGE
# mysql-7c9d8f4c56  statefulset.apps/mysql  1          2d
# mysql-5b4c2a1d78  statefulset.apps/mysql  2          1h
```

这个 `ControllerRevision`就是 StatefulSet 的"Git commit 记录"。

- 保留多少条：`.spec.revisionHistoryLimit`（默认 **10**，超出就 GC 最老的）

- 每条记录的本质：**那个时间点的 `spec.template`快照**（不是完整的 StatefulSet YAML）

------

## 4) `kubectl rollout undo sts/xxx`到底做了什么？

### 执行链路

```
你执行：kubectl rollout undo sts/mysql --to-revision=2
  │
  ▼
kubectl 向 API Server 发请求：
  "把 sts 的 spec.template 替换为 revision 2 存的那个模板快照"
  │
  ▼
sts.spec.template 被改回旧版
  │
  ▼
StatefulSet 控制器感知到 template 变了
  → 启动 RollingUpdate：从 mysql-2 → mysql-1 → mysql-0 逐个重建
  │
  ▼
同时新建一个 ControllerRevision（revision 号继续递增）
所以"回滚"不是删历史，而是：用旧模板生成一个新版本号
```

### 你日常用的三条命令

```
# 看历史（列出版本号 + 变更原因）
kubectl rollout history sts/mysql
kubectl rollout history sts/mysql --revision=2   # 看某版本的具体模板差异

# 回滚到上一个
kubectl rollout undo sts/mysql

# 回滚到指定版本
kubectl rollout undo sts/mysql --to-revision=2

# 等它滚完
kubectl rollout status sts/mysql --timeout=300s
```

------

## 5) ⚠️ 最大的坑：坏模板卡死（forced rollback 问题）

文档专门写了一节叫 **Forced rollback / broken state**，这个坑必须知道：

### 场景复现

1. 你改了镜像/配置 → 新模板有问题（起不来、探针永远 fail）

2. StatefulSet 更新到 `mysql-2`（最高序数），发现它 **永远不会 Ready**

3. 于是它 **停在那儿等你**——不继续往前、不自动退回

4. 你意识到错了，赶紧把模板改回"上次好使的镜像"

5. **但光改回去没用！** 因为 `mysql-2`那个 Pod 还在用坏模板跑着（CrashLoopBackOff），StatefulSet 的逻辑是：*"我在等 `mysql-2`Ready 才能继续"*

### 解法（官方给的）

```
# 第1步：把模板改回好版本（git 也好，kubectl edit 也好）
# 第2步：手动把卡住的 Pod 删掉，逼控制器用"新改回的好模板"重建
kubectl delete pod mysql-2
```

删掉后，StatefulSet 会用**当前 spec.template（好版本）**重建 `mysql-2`，一旦 Ready 了，才会继续处理 `mysql-1`→ `mysql-0`。

> 这一点和 Deployment 不一样：Deployment 是新旧两个 ReplicaSet 互相 scale，来回切、相对自愈；StatefulSet 是有序串行 + 强依赖 Ready，所以卡住时需要你"踹一脚"。

------

## 6) 和 Deployment 的核心差异速览

|              | **Deployment**                      | **StatefulSet**                                              |
| ------------ | ----------------------------------- | ------------------------------------------------------------ |
| 版本载体     | 旧的 ReplicaSet（scale 0/1）        | `ControllerRevision`（模板快照对象）                         |
| 更新顺序     | 并行滚动（比例可控）                | **严格倒序**：N-1 → … → 0（逐个等 Ready）                    |
| 回滚本质     | 把 `.spec.template`指回旧 RS 的模板 | 把 `.spec.template`**覆写为某 revision 存的模板**            |
| 卡死时       | 通常会自动退（新老 RS 博弈）        | **可能卡住**，需手动 `delete pod`破局                        |
| PVC 怎么处理 | 一般不用（stateless）               | `volumeClaimTemplate`生成的 PVC **不受模板变更影响**，Pod 重建后原 PVC 重新挂载回去 |

------

## 7) 一个完整实操示例

```
# 当前镜像
kubectl get sts mysql -o jsonpath='{.spec.template.spec.containers[0].image}'

# 改镜像触发 rollout
kubectl set image sts/mysql mysql= mysql:8.0.36

# 观察（有序：mysql-2 → 1 → 0）
kubectl rollout status sts/mysql --timeout=300s
kubectl get pods -l app=mysql -w

# 如果翻车卡住了
kubectl rollout history sts/mysql
kubectl rollout undo sts/mysql              # 回到上一版
# 还不行就：
kubectl delete pod mysql-2                   # 踢掉卡住的序号

# 查看存了哪些版本快照
kubectl get controllerrevisions -o wide
kubectl get controllerrevision <name> -o yaml
```

------

**一句话总结：** StatefulSet 的 rollout 没有"蓝绿切流量"那套花活，它就是**有序地（N→0）把 Pod 一个个按新模板重建**，回滚就是把模板指针覆写成历史快照再重建一遍——而 `ControllerRevision`是它的"版本控制数据库"，不是 git，但扮演了类似角色。