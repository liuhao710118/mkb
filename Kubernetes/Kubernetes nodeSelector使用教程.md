# Kubernetes `nodeSelector`使用教程

## 一句话概括

`nodeSelector`是 Kubernetes 中最简单的 **Pod → 节点绑定方式**：给节点打标签（Label），然后在 Pod 的 `spec.nodeSelector`里声明"我只要跑在带有这些标签的节点上"，调度器就只会把 Pod 调度到 **同时满足所有标签条件** 的节点。

------

## 整体流程（3 步走）

```
Step 1：给目标节点打 Label
Step 2：在 Pod / Deployment 的 YAML 里写 nodeSelector
Step 3：apply 并验证 Pod 落在预期节点上
```

------

## Step 1：给节点打标签

### 查看现有节点和已有标签

```
kubectl get nodes
kubectl get nodes --show-labels
```

输出示例：

```
NAME            STATUS   ROLES           AGE   VERSION
k8s-master      Ready    control-plane   10d   v1.28.x
k8s-worker-01   Ready    <none>          10d   v1.28.x
k8s-worker-02   Ready    <none>          10d   v1.28.x
```

### 给指定节点打标签

语法：

```
kubectl label nodes <节点名> <label-key>=<label-value>
```

例如，把 `k8s-worker-01`标记为有 SSD 磁盘：

```
kubectl label nodes k8s-worker-01 disktype=ssd
```

再比如，按"业务区域/机型/GPU"分组：

```
kubectl label nodes k8s-worker-01 zone=prod-us-east gpu=true
kubectl label nodes k8s-worker-02 zone=prod-us-east gpu=false
```

### 验证标签是否生效

```
kubectl describe node k8s-worker-01 | grep -A 10 "Labels:"
# 或
kubectl get node k8s-worker-01 --show-labels
```

### 修改 / 删除标签

```
# 覆盖修改（需要 --overwrite）
kubectl label nodes k8s-worker-01 disktype=hdd --overwrite

# 删除标签（key 后面加个 - ）
kubectl label nodes k8s-worker-01 disktype-
```

------

## Step 2：在 Pod / Deployment 中使用 nodeSelector

### 示例一：裸 Pod

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    env: test
spec:
  containers:
  - name: nginx
    image: nginx:latest
  # 👇 关键在这里
  nodeSelector:
    disktype: ssd
```

这个 Pod **只会**被调度到包含标签 `disktype=ssd`的节点上。

### 示例二：Deployment（更常见）

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deploy
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      # 👇 nodeSelector 写在 template.spec 下
      nodeSelector:
        zone: prod-us-east
        gpu: "true"
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
```

> ⚠️ 注意层级：`nodeSelector`在 **`PodSpec`** 层级，也就是 `spec.template.spec.nodeSelector`，不是在 Deployment 顶层。

### 示例三：GPU 节点典型用法

```
nodeSelector:
  accelerator: nvidia-tesla-p100
```

对应的节点需要提前执行：

```
kubectl label nodes <gpu-node> accelerator=nvidia-tesla-p100
```

------

## Step 3：部署 & 验证

```
kubectl apply -f pod-or-deploy.yaml

# 看 Pod 落在哪个节点
kubectl get pods -o wide

# 如果 Pod 一直 Pending，看原因
kubectl describe pod <pod-name>
```

如果 **没有任何节点匹配标签**，Pod 会处于 `Pending`状态，事件里会出现类似：

```
0/X nodes are available: ... node(s) didn't match node selector.
```

------

## 快速对照表

| 操作       | 命令                                                         |
| ---------- | ------------------------------------------------------------ |
| 查看节点   | `kubectl get nodes --show-labels`                            |
| 打标签     | `kubectl label nodes <node> key=value`                       |
| 改标签     | 加 `--overwrite`                                             |
| 删标签     | `kubectl label nodes <node> key-`                            |
| Pod 写哪里 | `spec.nodeSelector`（裸 Pod） / `spec.template.spec.nodeSelector`（Deploy） |

------

## nodeSelector vs 亲和性（该选哪个？）

| 维度                   | `nodeSelector`   | `nodeAffinity`                      |
| ---------------------- | ---------------- | ----------------------------------- |
| 语法                   | 简单 kv 等值匹配 | 支持 `In / NotIn / Gt / Lt`等表达式 |
| 软约束（优先但不强制） | ❌ 不支持         | ✅ `preferredDuringScheduling...`    |
| 多条件 OR 逻辑         | ❌ 只能 AND       | ✅                                   |
| 学习成本               | **极低**         | 稍高                                |

**结论**：如果你只需要"这个 Pod 必须跑在带某几个标签的节点上"，`nodeSelector`是最简洁的选择；如果需要"尽量跑在 SSD 节点，没有也行"或"不能跑在某个 zone"，就升级到 `nodeAffinity`。

------

如果你愿意，把你当前的 `kubectl get nodes --show-labels`输出贴出来，我可以帮你设计一套合理的标签命名方案（比如按 zone / 机型 / 专用角色拆分），以及给你一份贴合你集群的 YAML 模板。