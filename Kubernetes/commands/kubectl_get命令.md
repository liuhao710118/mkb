我来为你详细讲解 **`kubectl get`** 命令，这是 Kubernetes 中最基础、最常用的命令之一，用于**查询和列出集群中的资源**。

## 一、基础语法

```
kubectl get [资源类型] [资源名称] [选项]
```

## 二、常用资源类型

| 资源类型                 | 缩写     | 说明           |
| ------------------------ | -------- | -------------- |
| `pods`                   | `po`     | 查看Pod        |
| `deployments`            | `deploy` | 查看部署       |
| `services`               | `svc`    | 查看服务       |
| `replicasets`            | `rs`     | 查看副本集     |
| `daemonsets`             | `ds`     | 查看守护进程集 |
| `statefulsets`           | `sts`    | 查看有状态集   |
| `jobs`                   | -        | 查看作业       |
| `cronjobs`               | `cj`     | 查看定时作业   |
| `nodes`                  | `no`     | 查看节点       |
| `namespaces`             | `ns`     | 查看命名空间   |
| `configmaps`             | `cm`     | 查看配置映射   |
| `secrets`                | -        | 查看密钥       |
| `persistentvolumes`      | `pv`     | 查看持久卷     |
| `persistentvolumeclaims` | `pvc`    | 查看持久卷声明 |

## 三、常用输出格式

### 1. 默认格式（表格）

```
kubectl get pods
```

### 2. 宽格式（显示更多信息）

```
kubectl get pods -o wide
# 显示：POD IP、所在节点等信息
```

### 3. YAML格式

```
kubectl get pod <pod-name> -o yaml
```

### 4. JSON格式

```
kubectl get pod <pod-name> -o json
```

### 5. 自定义列

```
# 自定义显示的列
kubectl get pods -o custom-columns=NAME:.metadata.name,STATUS:.status.phase,NODE:.spec.nodeName

# 使用模板
kubectl get pods -o go-template='{{range .items}}{{.metadata.name}}{{"\t"}}{{.status.phase}}{{"\n"}}{{end}}'
```

## 四、筛选和过滤

### 1. 按标签筛选

```
# 选择具有特定标签的Pod
kubectl get pods -l app=nginx
kubectl get pods -l "app in (nginx, redis)"
kubectl get pods -l "app,environment=production"
```

### 2. 按字段选择器

```
# 选择特定节点上的Pod
kubectl get pods --field-selector spec.nodeName=node-1

# 选择运行中的Pod
kubectl get pods --field-selector status.phase=Running
```

### 3. 按命名空间

```
# 指定命名空间
kubectl get pods -n default
kubectl get pods -n kube-system

# 所有命名空间
kubectl get pods -A
kubectl get pods --all-namespaces
```

## 五、高级用法

### 1. 持续监控（watch）

```
# 实时监控Pod变化
kubectl get pods -w

# 每秒刷新一次
kubectl get pods -w -s 1
```

### 2. 显示标签

```
kubectl get pods --show-labels
```

### 3. 排序

```
# 按名称排序
kubectl get pods --sort-by=.metadata.name

# 按创建时间排序
kubectl get pods --sort-by=.metadata.creationTimestamp
```

### 4. 跨资源查询

```
# 查看所有资源
kubectl get all

# 查看特定命名空间的所有资源
kubectl get all -n default
```

## 六、实用示例

### 示例1：查看Pod详细信息

```
# 查看所有命名空间的Pod
kubectl get pods -A -o wide

# 查看特定Pod的详细信息
kubectl get pod nginx-pod -o yaml
```

### 示例2：按状态查看Pod

```
# 查看运行中的Pod
kubectl get pods --field-selector=status.phase=Running

# 查看非运行中的Pod
kubectl get pods --field-selector=status.phase!=Running
```

### 示例3：资源监控仪表板

```
# 创建一个实时监控面板
watch -n 2 "kubectl get pods,svc,deploy"
```

### 示例4：输出到文件

```
# 将输出保存到文件
kubectl get pods -o yaml > pods.yaml
kubectl get all -o json > all-resources.json
```

## 七、组合命令技巧

### 1. 快速查看资源状态

```
# 查看所有资源状态概览
kubectl get all,cm,secret,pv,pvc -A

# 查看节点资源情况
kubectl get nodes -o custom-columns="NAME:.metadata.name,CPU:.status.capacity.cpu,MEMORY:.status.capacity.memory"
```

### 2. 故障排查常用组合

```
# 查看Pod、服务、端点
kubectl get pods,svc,endpoints

# 查看事件
kubectl get events --sort-by=.lastTimestamp
```

## 八、最佳实践建议

1. **使用别名提高效率**

```
alias kgp="kubectl get pods"
alias kgs="kubectl get services"
alias kgd="kubectl get deployments"
```

1. **结合`jq`处理JSON输出**

```
kubectl get pods -o json | jq '.items[].metadata.name'
```

1. **使用`kubectl describe`获取详细信息**

```
# get 用于列表，describe 用于详情
kubectl get pod nginx-pod
kubectl describe pod nginx-pod
```

## 总结

`kubectl get`是 Kubernetes 管理的基础，掌握它的各种选项和组合能极大提高工作效率。记住：

- **查询资源用 `get`**，**查看详情用 `describe`**

- 善用 `-o`选项控制输出格式

- 使用标签选择器精准筛选资源

- 掌握 `-A`查看所有命名空间
