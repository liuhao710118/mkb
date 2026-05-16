# kubectl label 命令详解

`kubectl label` 用于给 Kubernetes 资源添加、修改、删除 **Label（标签）**。

Label 本质上是：

```text
key=value
```

形式的键值对元数据。

例如：

```text
app=nginx
env=prod
version=v1
team=ops
```

Kubernetes 大量核心功能都依赖 Label：

- Pod 筛选
- Service 服务发现
- Deployment 管理 Pod
- Node 调度
- NetworkPolicy
- HPA
- 运维分类管理

可以理解为：

```text
Label = Kubernetes 的标签系统
Selector = Kubernetes 的筛选器
```

------

# 一、为什么需要 Label

假设集群中有：

```text
1000个 Pod
```

如果没有标签：

```text
nginx-6754f4f65-jx1
nginx-6754f4f65-jx2
mysql-5c7d77d4b
redis-54f75
```

很难按业务分类。

加 Label 后：

```text
PodA:
app=nginx
env=prod

PodB:
app=mysql
env=test

PodC:
app=redis
env=prod
```

这样可以：

```text
按标签筛选资源
```

------

# 二、查看 Label

## 查看 Pod 标签

```bash
kubectl get pods --show-labels
```

示例：

```text
NAME      READY   STATUS

nginx     1/1     Running

LABELS
app=nginx,env=prod
```

------

## 查看指定资源标签

```bash
kubectl get pod nginx \
-o yaml
```

找到：

```yaml
metadata:
  labels:
    app: nginx
    env: prod
```

------

## 自定义显示标签

```bash
kubectl get pods -L app
```

输出：

```text
NAME      READY STATUS APP

nginx     1/1   Running nginx
```

多个：

```bash
kubectl get pods -L app,env,version
```

------

# 三、添加 Label

基本格式：

```bash
kubectl label 资源类型 资源名 key=value
```

例如：

```bash
kubectl label pod nginx env=prod
```

执行后：

```text
pod/nginx labeled
```

结果：

```yaml
labels:
  env: prod
```

------

## 一次添加多个标签

```bash
kubectl label pod nginx \
app=nginx \
env=prod \
version=v1
```

结果：

```yaml
labels:
    app: nginx
    env: prod
    version: v1
```

------

# 四、修改 Label

假设已有：

```yaml
labels:
   env: test
```

修改：

```bash
kubectl label pod nginx env=prod --overwrite
```

参数：

```text
--overwrite
```

作用：

```text
允许覆盖已有标签
```

否则：

```text
error:
'env' already exists
```

------

# 五、删除 Label

删除方式：

```bash
kubectl label pod nginx env-
```

注意：

```text
减号 (-)
```

不是：

```bash
kubectl delete label
```

执行后：

```yaml
labels:
   app: nginx
```

env 消失。

------

# 六、批量操作 Label

------

## 给所有 Pod 加标签

```bash
kubectl label pods --all env=test
```

------

## 给命名空间下所有 Pod 加标签

```bash
kubectl label pods -n default \
--all team=ops
```

------

## 根据 Selector 批量加标签

```bash
kubectl label pod \
-l app=nginx \
version=v2
```

含义：

```text
所有 app=nginx 的 Pod
增加：

version=v2
```

------

# 七、Label Selector（重点）

Label 的价值在：

```text
Selector
```

即标签选择器。

分两类：

1. 等值选择
2. 集合选择

------

# 八、等值选择

支持：

```text
=
==
!=
```

------

## 查询 app=nginx

```bash
kubectl get pods \
-l app=nginx
```

------

## 查询 env!=prod

```bash
kubectl get pods \
-l env!=prod
```

------

## 多条件

```bash
kubectl get pods \
-l app=nginx,env=prod
```

相当于：

```text
app=nginx AND env=prod
```

------

# 九、集合选择

支持：

```text
in
notin
exists
!key
```

------

## in

```bash
kubectl get pods \
-l 'env in (prod,test)'
```

含义：

```text
env=prod
或者
env=test
```

------

## notin

```bash
kubectl get pods \
-l 'env notin(prod)'
```

------

## exists

```bash
kubectl get pods \
-l app
```

意思：

```text
存在 app 标签
```

------

## 不存在

```bash
kubectl get pods \
-l '!app'
```

------

# 十、资源类型都能打 Label

不仅 Pod。

支持：

```text
Node
Namespace
Deployment
Service
PVC
PV
ConfigMap
Secret
```

------

## Node 示例

```bash
kubectl label node worker01 disktype=ssd
```

结果：

```yaml
labels:
    disktype: ssd
```

------

## Namespace 示例

```bash
kubectl label namespace dev team=backend
```

------

# 十一、Node Label（生产重点）

Node Label 经常用于：

```text
节点调度
```

例如：

```bash
kubectl label node node1 zone=beijing
```

Pod：

```yaml
spec:
  nodeSelector:
    zone: beijing
```

调度过程：

```text
Pod
 ↓
nodeSelector
 ↓
匹配 Label
 ↓
调度到 Node
```

------

# 十二、Deployment 与 Label

Deployment 使用：

```yaml
selector:
  matchLabels:
    app: nginx
```

以及：

```yaml
template:
  metadata:
    labels:
      app: nginx
```

完整：

```yaml
apiVersion: apps/v1
kind: Deployment

spec:
  selector:
    matchLabels:
      app: nginx

  template:
    metadata:
      labels:
        app: nginx
```

------

## 注意（高频坑）

下面不能不一致：

```yaml
selector:
    app=nginx

template:
    app=tomcat
```

否则：

```text
Deployment 无法管理 Pod
```

甚至创建失败：

```text
selector does not match template labels
```

------

# 十三、Service 与 Label

Service 靠标签找 Pod：

```yaml
selector:
   app: nginx
```

工作流程：

```text
Service
   ↓
Selector
   ↓
匹配 Pod Label
   ↓
生成 Endpoint
```

------

示例：

Pod：

```yaml
labels:
   app: nginx
```

Service：

```yaml
selector:
   app: nginx
```

匹配成功：

```text
请求
 ↓
Service
 ↓
Pod
```

------

# 十四、标签命名规范

K8s 官方建议：

```text
app.kubernetes.io/name
app.kubernetes.io/version
app.kubernetes.io/component
app.kubernetes.io/instance
app.kubernetes.io/part-of
app.kubernetes.io/managed-by
```

示例：

```yaml
labels:
    app.kubernetes.io/name: nginx
    app.kubernetes.io/version: v1
    app.kubernetes.io/component: web
```

------

# 十五、自定义前缀

格式：

```text
prefix/key=value
```

例如：

```yaml
company.com/team=devops
```

------

官方保留：

```text
kubernetes.io/*
k8s.io/*
```

不要乱用：

```yaml
kubernetes.io/test=true
```

------

# 十六、生产环境标签设计建议

建议至少：

```yaml
labels:
    app: order-service
    env: prod
    version: v2
    team: backend
    region: singapore
```

作用：

| 标签    | 用途     |
| ------- | -------- |
| app     | 应用名称 |
| env     | 环境     |
| version | 版本     |
| team    | 团队     |
| region  | 区域     |

------

# 十七、常见错误

### 1. 修改 Deployment 的 selector

错误：

```yaml
spec:
   selector:
```

大部分情况：

```text
不可修改
```

因为：

```text
immutable field
```

------

### 2. Pod 标签与 Service 不匹配

现象：

```text
Service 无 Endpoint
```

查看：

```bash
kubectl get ep
```

------

### 3. 滥用标签

错误：

```yaml
labels:
   createTime: 20260515100000
```

Label 不适合频繁变化字段。

应使用：

```yaml
annotations
```

------

# 十八、Label 与 Annotation 区别（面试高频）

| 项目          | Label    | Annotation |
| ------------- | -------- | ---------- |
| 用途          | 分类筛选 | 描述信息   |
| 支持 Selector | √        | ×          |
| 查询性能      | 高       | 低         |
| 频繁变化      | 不推荐   | 可以       |

------

例子：

Label：

```yaml
labels:
   app: nginx
```

Annotation：

```yaml
annotations:
   description: nginx web service
```

------

# 十九、面试高频问题

### 1. Label 和 Selector 的关系？

```text
Label 是标签
Selector 是筛选规则
```

------

### 2. Service 如何找到 Pod？

```text
Service
 ↓
Selector
 ↓
Pod Label
 ↓
Endpoint
```

------

### 3. Label 和 Annotation 区别？

```text
Label 用于筛选
Annotation 用于描述
```

------

### 4. 为什么 Node Label 很重要？

用于：

```text
节点调度
```

例如：

```yaml
nodeSelector:
```

------

# 二十、总结

```text
Label = 给资源贴标签
Selector = 根据标签找资源
```

核心应用：

```text
Deployment 管理 Pod
Service 找 Pod
Node 调度
批量运维
资源分类
```

生产环境中最常见的标签体系：

```yaml
app: xxx
env: prod
version: v1
team: devops
region: singapore
```

这套设计基本能覆盖大多数集群管理场景。