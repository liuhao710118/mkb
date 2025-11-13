# 🧭 kubectl create 

## 一、命令概述

`kubectl create` 是 Kubernetes 中的一个基础命令，用于**在集群中创建资源对象（Resource Object）**，例如 Pod、Deployment、Service、ConfigMap、Secret、Namespace 等。

与 `kubectl apply` 不同，`kubectl create` 只负责**新建资源**，如果该资源已存在则会报错。

------

## 二、命令语法

```bash
kubectl create [RESOURCE] [NAME] [FLAGS]
kubectl create -f FILENAME [FLAGS]
```

### 参数说明：

| 参数             | 说明                                                |
| ---------------- | --------------------------------------------------- |
| `RESOURCE`       | 资源类型，如 pod、deployment、service、configmap 等 |
| `NAME`           | 资源名称                                            |
| `-f, --filename` | 指定 YAML 或 JSON 文件路径，或目录                  |
| `--dry-run`      | 模拟执行，不真正创建资源                            |
| `-o yaml/json`   | 以 YAML/JSON 格式输出资源定义                       |
| `--namespace`    | 指定命名空间创建资源                                |

------

## 三、使用方式

### 1️⃣ 从文件创建资源

这是最常见、最推荐的方式：

```bash
kubectl create -f app.yaml
```

若要一次性创建多个资源：

```bash
kubectl create -f pod.yaml -f service.yaml
```

从目录创建：

```bash
kubectl create -f ./manifests/
```

------

### 2️⃣ 从命令行直接创建资源

适合快速测试或临时资源创建。

#### （1）创建 Deployment

```bash
kubectl create deployment my-deploy --image=nginx
```

#### （2）创建 Pod

```bash
kubectl create pod my-pod --image=nginx
```

> ⚠️ 注意：在新版 Kubernetes 中，建议使用 `kubectl run` 创建单个 Pod。

#### （3）创建 Namespace

```bash
kubectl create namespace test
```

#### （4）创建 Service

```bash
kubectl create service clusterip my-service --tcp=80:8080
```

创建 NodePort 类型：

```bash
kubectl create service nodeport my-service --tcp=80:8080 --node-port=30080
```

#### （5）创建 ConfigMap

```bash
kubectl create configmap app-config \
  --from-literal=env=prod \
  --from-literal=version=1.0
```

或从文件：

```bash
kubectl create configmap app-config --from-file=app.properties
```

#### （6）创建 Secret

```bash
kubectl create secret generic db-secret \
  --from-literal=username=admin \
  --from-literal=password=12345
```

#### （7）创建 CronJob

```bash
kubectl create cronjob my-job \
  --image=busybox \
  --schedule="*/5 * * * *" \
  -- /bin/sh -c "echo Hello Kubernetes"
```

------

## 四、生成 YAML 模板（推荐技巧）

`kubectl create` 支持生成资源模板，而不实际创建，可用于编写 YAML 文件。

```bash
kubectl create deployment nginx --image=nginx --dry-run=client -o yaml > deployment.yaml
```

你可以先生成模板，再修改内容，然后用：

```bash
kubectl apply -f deployment.yaml
```

这样资源定义可重复、可版本化。

------

## 五、常用选项说明

| 选项               | 说明                                              |
| ------------------ | ------------------------------------------------- |
| `-f, --filename`   | 指定 YAML/JSON 文件或目录路径                     |
| `--dry-run=client` | 模拟执行，不真正创建资源                          |
| `-o yaml/json`     | 输出资源定义（通常配合 dry-run 使用）             |
| `--save-config`    | 将配置保存到 annotation，方便后续 `kubectl apply` |
| `--namespace`      | 指定命名空间                                      |
| `--from-literal`   | 从命令行直接定义键值对（ConfigMap/Secret）        |
| `--from-file`      | 从文件加载键值对（ConfigMap/Secret）              |

------

## 六、常见资源类型与示例

| 资源类型       | 示例命令                                                     |
| -------------- | ------------------------------------------------------------ |
| Pod            | `kubectl create -f pod.yaml`                                 |
| Deployment     | `kubectl create deployment my-deploy --image=nginx`          |
| Service        | `kubectl create service clusterip my-svc --tcp=80:8080`      |
| Namespace      | `kubectl create namespace dev`                               |
| ConfigMap      | `kubectl create configmap my-config --from-literal=key=value` |
| Secret         | `kubectl create secret generic my-secret --from-literal=password=123` |
| CronJob        | `kubectl create cronjob myjob --image=busybox --schedule="*/5 * * * *"` |
| Role           | `kubectl create role pod-reader --verb=get,list,watch --resource=pods` |
| RoleBinding    | `kubectl create rolebinding read-pods --role=pod-reader --user=devuser --namespace=dev` |
| ServiceAccount | `kubectl create serviceaccount my-sa`                        |

------

## 七、`create` 与 `apply` 区别

| 对比项         | `kubectl create`   | `kubectl apply`             |
| -------------- | ------------------ | --------------------------- |
| 功能           | 创建新资源         | 创建或更新资源              |
| 是否保存配置   | 否                 | 是（保存在 annotation）     |
| 是否可重复执行 | 否（存在即报错）   | 是（自动更新）              |
| 推荐场景       | 初次创建、快速测试 | 持续部署、CI/CD、版本化配置 |

------

## 八、示例：一次创建完整应用

```bash
# 1. 创建命名空间
kubectl create namespace webapp

# 2. 创建 ConfigMap
kubectl create configmap web-config \
  --from-literal=ENV=prod --namespace=webapp

# 3. 创建 Secret
kubectl create secret generic web-secret \
  --from-literal=password=123456 --namespace=webapp

# 4. 创建 Deployment
kubectl create deployment web-deploy --image=nginx --namespace=webapp

# 5. 暴露 Service
kubectl create service nodeport web-svc --tcp=80:80 --namespace=webapp
```

------

## 九、常见错误与排查

| 错误信息        | 原因                 | 解决办法                                  |
| --------------- | -------------------- | ----------------------------------------- |
| `AlreadyExists` | 资源已存在           | 改用 `kubectl apply` 或 `kubectl replace` |
| `Invalid value` | 参数或字段错误       | 使用 `kubectl explain` 检查字段定义       |
| `not found`     | 文件或路径错误       | 检查 `-f` 路径是否正确                    |
| 无权限错误      | 当前用户 RBAC 不允许 | 检查 Role/ClusterRole 权限                |

------

## 十、最佳实践建议

1. ✅ **尽量使用 YAML 文件管理资源**，方便回滚与版本控制。
2. ✅ **结合 `--dry-run=client -o yaml`**，生成模板文件再 apply。
3. ✅ **按命名空间区分环境**（如 dev/test/prod）。
4. ✅ **避免直接在生产环境中用命令式创建**，改用 CI/CD 部署。
5. ✅ 使用 `kubectl explain` 熟悉资源结构。

------

## 📘 参考命令速查表

| 类型            | 示例命令                                                     |
| --------------- | ------------------------------------------------------------ |
| 从文件创建      | `kubectl create -f app.yaml`                                 |
| 创建 Deployment | `kubectl create deployment nginx --image=nginx`              |
| 创建 Service    | `kubectl create service clusterip my-svc --tcp=80:8080`      |
| 创建 ConfigMap  | `kubectl create configmap app-config --from-file=config.properties` |
| 创建 Secret     | `kubectl create secret generic db-secret --from-literal=password=123` |
| 创建 CronJob    | `kubectl create cronjob job1 --image=busybox --schedule="*/5 * * * *"` |
| 模拟执行        | `kubectl create deployment nginx --image=nginx --dry-run=client -o yaml` |

当然可以 ✅
 以下是一份格式规范、适合教学或培训使用的 **《kubectl explain 》** 文档。内容涵盖命令功能、语法结构、常见用法、进阶技巧与实际示例，可直接作为内部培训资料或学习笔记使用。

------

# 🧭 kubectl delete 

## 一、命令概述

`kubectl delete` 是 Kubernetes 中用于**删除资源对象（Resource Object）**的命令。
 它可以删除单个或多个资源（如 Pod、Deployment、Service、Namespace、ConfigMap、Secret 等），
 也可以根据 **标签选择器（label selector）** 或 **资源文件** 批量删除。

------

## 二、命令语法

```bash
kubectl delete (TYPE[.VERSION][.GROUP]) [NAME ...] [flags]
kubectl delete -f FILENAME
```

### 参数说明

| 参数                 | 说明                                    |
| -------------------- | --------------------------------------- |
| `TYPE`               | 资源类型（如 pod、deployment、service） |
| `NAME`               | 资源名称，可多个                        |
| `-f, --filename`     | 指定 YAML/JSON 文件或目录               |
| `-l, --selector`     | 按标签选择要删除的资源                  |
| `--all`              | 删除命名空间下所有该类型资源            |
| `--namespace`        | 指定命名空间                            |
| `--ignore-not-found` | 忽略不存在的资源                        |
| `--force`            | 强制删除（跳过优雅终止）                |
| `--grace-period`     | 优雅终止的等待时间（秒）                |
| `--wait`             | 等待资源删除完成                        |

------

## 三、基本用法

### 1️⃣ 删除单个资源

```bash
kubectl delete pod my-pod
```

删除一个名为 `my-pod` 的 Pod。

------

### 2️⃣ 删除多个资源

```bash
kubectl delete pod my-pod1 my-pod2 my-pod3
```

------

### 3️⃣ 删除指定类型的所有资源

```bash
kubectl delete pods --all
```

删除当前命名空间下的所有 Pod。

------

### 4️⃣ 删除指定命名空间下的资源

```bash
kubectl delete pod --all --namespace=test
```

------

### 5️⃣ 根据标签选择器删除

```bash
kubectl delete pod -l app=nginx
```

删除所有带有标签 `app=nginx` 的 Pod。

可组合多个条件：

```bash
kubectl delete pod -l "app=web,env=prod"
```

------

### 6️⃣ 从文件中删除资源

```bash
kubectl delete -f pod.yaml
```

同时删除多个文件：

```bash
kubectl delete -f pod.yaml -f svc.yaml
```

或删除整个目录下的资源定义：

```bash
kubectl delete -f ./manifests/
```

------

### 7️⃣ 删除 Deployment 及相关资源

```bash
kubectl delete deployment my-deploy
```

若要一并删除由 Deployment 创建的 Pod：

> 注意：删除 Deployment 时，其控制的 Pod 会自动被清理，不需单独删除。

------

### 8️⃣ 删除命名空间

```bash
kubectl delete namespace test
```

⚠️ 删除命名空间会删除该命名空间下的所有资源！

------

### 9️⃣ 删除所有资源（谨慎操作）

```bash
kubectl delete all --all
```

这会删除命名空间内所有可删除的资源对象（包括 Pod、Service、Deployment 等）。

------

## 四、进阶用法

### 1️⃣ 忽略不存在的资源（防止报错）

```bash
kubectl delete pod my-pod --ignore-not-found
```

------

### 2️⃣ 优雅删除（指定宽限期）

```bash
kubectl delete pod my-pod --grace-period=10
```

表示允许容器在 10 秒内完成关闭操作（默认 30 秒）。

------

### 3️⃣ 强制删除卡住的 Pod

如果 Pod 长时间处于 `Terminating` 状态，可使用：

```bash
kubectl delete pod my-pod --grace-period=0 --force
```

> ⚠️ 警告：这会跳过容器优雅关闭，可能导致数据丢失。

------

### 4️⃣ 删除特定类型的所有资源（按命名空间）

```bash
kubectl delete deployments,services --all -n dev
```

------

### 5️⃣ 删除所有被 Label 选中的多类型资源

```bash
kubectl delete all -l app=nginx
```

可同时删除带标签 `app=nginx` 的 Pod、Service、Deployment、ReplicaSet 等。

------

## 五、常用示例

| 场景             | 命令示例                                             |
| ---------------- | ---------------------------------------------------- |
| 删除一个 Pod     | `kubectl delete pod my-pod`                          |
| 删除多个 Pod     | `kubectl delete pod pod1 pod2`                       |
| 删除全部 Pod     | `kubectl delete pods --all`                          |
| 删除 Deployment  | `kubectl delete deployment my-deploy`                |
| 删除 Service     | `kubectl delete service my-svc`                      |
| 删除 Namespace   | `kubectl delete namespace test`                      |
| 删除带标签的资源 | `kubectl delete pod -l app=nginx`                    |
| 从文件删除资源   | `kubectl delete -f deployment.yaml`                  |
| 强制删除         | `kubectl delete pod my-pod --grace-period=0 --force` |
| 删除全部资源     | `kubectl delete all --all`                           |

------

## 六、删除资源后的验证

可以使用以下命令确认资源是否已被删除：

```bash
kubectl get all
```

或针对单一类型：

```bash
kubectl get pod
```

若显示：

```
No resources found
```

则表示资源已被删除。

------

## 七、常见错误与排查

| 错误信息               | 原因                 | 解决方法                        |
| ---------------------- | -------------------- | ------------------------------- |
| `not found`            | 资源不存在           | 检查名称或命名空间              |
| `forbidden`            | 无权限               | 检查 RBAC 权限或使用管理员账户  |
| `Terminating` 状态卡住 | Pod 仍在等待优雅终止 | 使用 `--grace-period=0 --force` |
| 命令卡住不返回         | 默认等待删除完成     | 可加 `--wait=false` 立即返回    |

------

## 八、与其他命令的对比

| 命令              | 功能           | 特点             |
| ----------------- | -------------- | ---------------- |
| `kubectl create`  | 创建资源       | 新建对象         |
| `kubectl apply`   | 创建或更新资源 | 推荐用于配置管理 |
| `kubectl delete`  | 删除资源       | 移除对象         |
| `kubectl replace` | 替换资源       | 删除并重建对象   |

------

## 九、最佳实践建议

1. ✅ **优先使用标签选择器删除**，避免误删。
2. ✅ **生产环境慎用 `--all` 或 `--force`**。
3. ✅ 删除前可用 `kubectl get` 验证要删除的资源。
4. ✅ 删除命名空间会删除其所有资源，操作前需确认。
5. ✅ 对卡住的 Pod 可用 `--force` 清理，但应查明根因（如存储挂载问题）。
6. ✅ 删除后可执行 `kubectl get all` 验证清理结果。

------

## 十、实战案例：清理测试环境资源

```bash
# 1. 删除 test 命名空间下的所有资源
kubectl delete all --all -n test

# 2. 删除对应的 ConfigMap 和 Secret
kubectl delete configmap,secret --all -n test

# 3. 删除命名空间本身
kubectl delete namespace test
```

------

## 📘 常用命令速查表

| 功能             | 命令                                                 |
| ---------------- | ---------------------------------------------------- |
| 删除 Pod         | `kubectl delete pod my-pod`                          |
| 删除多个资源     | `kubectl delete pod pod1 pod2`                       |
| 删除带标签资源   | `kubectl delete pod -l app=nginx`                    |
| 删除所有资源     | `kubectl delete all --all`                           |
| 从文件删除       | `kubectl delete -f app.yaml`                         |
| 强制删除         | `kubectl delete pod my-pod --grace-period=0 --force` |
| 删除命名空间     | `kubectl delete namespace dev`                       |
| 忽略不存在的资源 | `kubectl delete pod my-pod --ignore-not-found`       |

------

## 十一、总结

`kubectl delete` 是 Kubernetes 管理资源生命周期中最常用的清理命令之一。
 掌握其多种删除方式（按名称、文件、标签、命名空间等）与参数选项，
 能够在日常运维、测试、部署清理中更高效、更安全地管理资源。

------





# 🧭 kubectl explain 

> 通常想要知道集群上有哪些资源可以通过kubectl api-resources命令获取到

## 一、命令概述

`kubectl explain` 是 Kubernetes 中用于**查看资源对象及其字段定义**的命令。
 通过它，你可以了解任意 Kubernetes 资源（如 Pod、Deployment、Service 等）的：

- 含义与用途；
- 字段结构；
- 各字段类型及说明；
- 可嵌套字段的层级关系。

该命令非常适合在编写 YAML 清单文件时用来**确认字段用法与数据结构**。

------

## 二、命令语法

```bash
kubectl explain <resource> [FIELD.subField] [flags]
```

### 参数说明

| 参数               | 说明                                               |
| ------------------ | -------------------------------------------------- |
| `<resource>`       | 要查询的资源类型（如 pod、service、deployment 等） |
| `<FIELD.subField>` | 要深入查看的字段（如 `spec.containers.image`）     |
| `--recursive`      | 展开显示所有子字段                                 |
| `--api-version`    | 指定 API 版本（例如 `apps/v1`）                    |

------

## 三、基本用法

### 1️⃣ 查看资源的总体说明

```bash
kubectl explain pod
```

输出示例：

```
KIND:     Pod
VERSION:  v1

DESCRIPTION:
     Pod is a collection of containers that can run on a host...
```

> 说明：
>  这里可以看到该资源的类型（KIND）、版本（VERSION）以及功能描述。

------

### 2️⃣ 查看资源字段结构

```bash
kubectl explain pod.spec
```

输出示例：

```
KIND:     Pod
VERSION:  v1

RESOURCE: spec <Object>

DESCRIPTION:
     Specification of the desired behavior of the pod.

FIELDS:
   containers  <[]Object>
     List of containers belonging to the pod...
```

> 从这里可以看到 `spec` 是一个对象，其中包含 `containers` 数组字段。

------

### 3️⃣ 查看更深层字段

```bash
kubectl explain pod.spec.containers
kubectl explain pod.spec.containers.image
```

输出示例：

```
KIND:     Pod
VERSION:  v1

FIELD:    image <string>

DESCRIPTION:
     Docker image name. For example: nginx:latest
```

> 这能帮助我们准确了解 YAML 文件中字段的名称、类型与用途。

------

### 4️⃣ 递归查看所有字段（推荐）

```bash
kubectl explain pod --recursive
```

该命令会列出 **Pod 的全部字段层级结构**，包括 `metadata`、`spec`、`status` 等。

> ⚠️ 输出非常长，建议结合 `grep` 使用过滤关键字：
>
> ```bash
> kubectl explain pod --recursive | grep containers
> ```

------

### 5️⃣ 指定 API 版本查看

某些资源（如 Deployment）在不同 API 版本下字段结构可能不同，可通过以下命令指定版本：

```bash
kubectl explain deployment --api-version=apps/v1
```

------

## 四、实用示例

| 示例                        | 命令                                        |
| --------------------------- | ------------------------------------------- |
| 查看 Service 的端口字段     | `kubectl explain service.spec.ports`        |
| 查看 Deployment 副本字段    | `kubectl explain deployment.spec.replicas`  |
| 查看 CronJob 调度规则       | `kubectl explain cronjob.spec.schedule`     |
| 查看 Pod 中容器环境变量结构 | `kubectl explain pod.spec.containers.env`   |
| 查看 StatefulSet 模板定义   | `kubectl explain statefulset.spec.template` |

------

## 五、字段类型说明

在命令输出中，会看到字段类型提示：

| 类型                  | 含义                             |
| --------------------- | -------------------------------- |
| `<string>`            | 字符串                           |
| `<integer>`           | 整数                             |
| `<boolean>`           | 布尔值（true/false）             |
| `<[]Object>`          | 对象数组（例如 containers 列表） |
| `<map[string]string>` | 键值对（如 labels, annotations） |
| `<Object>`            | 嵌套对象                         |

了解这些类型有助于正确编写 YAML 清单。

------

## 六、与其他命令的配合

| 命令                    | 功能               | 示例                          |
| ----------------------- | ------------------ | ----------------------------- |
| `kubectl api-resources` | 查看所有资源类型   | `kubectl api-resources`       |
| `kubectl get`           | 查看资源实例       | `kubectl get pod`             |
| `kubectl describe`      | 查看资源运行时详情 | `kubectl describe pod my-pod` |
| `kubectl explain`       | 查看资源字段定义   | `kubectl explain pod.spec`    |

这四个命令经常组合使用，用于理解资源配置与排查问题。

------

## 七、使用技巧

### 1️⃣ 快速查找字段定义

```bash
kubectl explain deployment --recursive | grep replicas
```

### 2️⃣ 对比不同版本字段变化

```bash
kubectl explain deployment --api-version=apps/v1beta1
kubectl explain deployment --api-version=apps/v1
```

### 3️⃣ 结合文档与实际集群验证

使用 `kubectl explain` 验证你编写的 YAML 字段在当前 Kubernetes 版本中是否有效。

------

## 八、常见问题与解决

| 问题                      | 原因                     | 解决方法                                    |
| ------------------------- | ------------------------ | ------------------------------------------- |
| 提示 `resource not found` | 资源类型错误或缩写不正确 | 使用 `kubectl api-resources` 查找正确资源名 |
| 字段找不到                | 字段层级写错             | 使用 `--recursive` 查看完整路径             |
| 输出为空或不全            | Kubernetes 版本较旧      | 检查 API 版本并使用 `--api-version` 参数    |

------

## 九、实战演练示例

以下示例演示如何逐步理解一个 Deployment 对象的结构：

```bash
kubectl explain deployment
kubectl explain deployment.spec
kubectl explain deployment.spec.template
kubectl explain deployment.spec.template.spec.containers
kubectl explain deployment.spec.template.spec.containers.image
```

逐步深入后，你会清楚 YAML 中的每个层级含义：

```yaml
spec:
  replicas: 3
  template:
    spec:
      containers:
      - name: nginx
        image: nginx:latest
```

------

## 十、总结与建议

| 建议                                  | 说明                 |
| ------------------------------------- | -------------------- |
| ✅ 多用 `kubectl explain` 理解字段结构 | 能避免 YAML 配置错误 |
| ✅ 配合 `--recursive` 与 `grep` 使用   | 高效定位字段         |
| ✅ 指定 `--api-version` 检查版本差异   | 避免版本不兼容       |
| ✅ 在学习或编写清单文件时边查边写      | 提升准确性与理解深度 |

------

## 📘 常用命令速查表

| 功能           | 命令                                               |
| -------------- | -------------------------------------------------- |
| 查看资源定义   | `kubectl explain pod`                              |
| 查看子字段     | `kubectl explain pod.spec`                         |
| 查看深层字段   | `kubectl explain pod.spec.containers.image`        |
| 查看所有字段   | `kubectl explain pod --recursive`                  |
| 指定 API 版本  | `kubectl explain deployment --api-version=apps/v1` |
| 结合 grep 过滤 | `kubectl explain pod --recursive                   |

------

# 🧭 kubectl expose 

## 一、命令概述

`kubectl expose` 命令用于**将 Kubernetes 中已存在的资源（如 Pod、Deployment、ReplicaSet、ReplicationController）对外暴露为一个 Service（服务）**。

通过该命令，可以轻松地为内部工作负载创建一个访问入口，实现负载均衡或外部访问。

------

## 二、命令语法

```bash
kubectl expose RESOURCE NAME [--port=port] [--target-port=port] [--type=serviceType] [flags]
```

### 参数说明

| 参数               | 说明                                                         |
| ------------------ | ------------------------------------------------------------ |
| `RESOURCE`         | 要暴露的资源类型，如 pod、deployment、replicaset、rc         |
| `NAME`             | 资源名称                                                     |
| `--port`           | Service 暴露的端口号（客户端访问端口）                       |
| `--target-port`    | 容器内部端口号（容器监听端口）                               |
| `--type`           | Service 类型，常见值有 `ClusterIP`、`NodePort`、`LoadBalancer`、`ExternalName` |
| `--protocol`       | 通信协议（默认 TCP）                                         |
| `--name`           | 指定新建 Service 的名称                                      |
| `--dry-run=client` | 模拟执行，不真正创建（用于生成 YAML）                        |
| `-o yaml/json`     | 输出 YAML/JSON 格式资源定义                                  |

------

## 三、常用 Service 类型

| 类型           | 说明                                   |
| -------------- | -------------------------------------- |
| `ClusterIP`    | （默认）仅在集群内可访问               |
| `NodePort`     | 暴露到每个节点的固定端口，可供外部访问 |
| `LoadBalancer` | 暴露为外部负载均衡服务（云平台场景）   |
| `ExternalName` | 将服务映射到外部域名（DNS 级别）       |

------

## 四、基本用法示例

### 1️⃣ 暴露 Deployment 为 ClusterIP 服务

```bash
kubectl expose deployment my-deploy --port=80 --target-port=8080 --name=my-service
```

说明：

- 为 `my-deploy` 创建名为 `my-service` 的 Service；
- Service 端口 80；
- 容器内部端口 8080；
- 默认类型为 `ClusterIP`。

------

### 2️⃣ 暴露 Deployment 为 NodePort 服务（外部可访问）

```bash
kubectl expose deployment my-deploy --type=NodePort --port=80 --target-port=8080
```

Kubernetes 会自动分配一个 NodePort（通常范围 30000~32767）。

手动指定 NodePort：

```bash
kubectl expose deployment my-deploy --type=NodePort --port=80 --target-port=8080 --name=my-service --node-port=30080
```

------

### 3️⃣ 暴露 Pod 为 Service

```bash
kubectl expose pod nginx-pod --port=80 --target-port=80
```

------

### 4️⃣ 暴露 ReplicaSet 或 ReplicationController

```bash
kubectl expose rs my-rs --port=80 --target-port=8080
kubectl expose rc my-rc --port=80 --target-port=8080
```

------

### 5️⃣ 创建 LoadBalancer 服务（云环境）

```bash
kubectl expose deployment web --type=LoadBalancer --port=80 --target-port=8080
```

在云平台中（如 GCP、AWS、Azure），此命令会自动创建外部负载均衡器并分配公网 IP。

------

### 6️⃣ 暴露为 ExternalName 类型服务

```bash
kubectl expose deployment myapp --type=ExternalName --external-name=example.com
```

这样，访问 `myapp` 服务时会被 DNS 解析到 `example.com`。

------

## 五、生成 YAML 模板（dry-run 模式）

如果只想生成 Service 配置文件，而不立即创建：

```bash
kubectl expose deployment my-deploy \
  --port=80 --target-port=8080 --dry-run=client -o yaml > service.yaml
```

编辑完成后再执行：

```bash
kubectl apply -f service.yaml
```

------

## 六、常用选项说明

| 选项               | 说明                                    |
| ------------------ | --------------------------------------- |
| `--port`           | Service 对外暴露的端口号                |
| `--target-port`    | Pod 内部容器监听的端口                  |
| `--protocol`       | 通信协议（TCP/UDP）                     |
| `--type`           | Service 类型                            |
| `--name`           | 指定 Service 名称                       |
| `--selector`       | 自定义选择标签（不使用原资源的 labels） |
| `--dry-run=client` | 模拟创建（不会提交）                    |
| `-o yaml/json`     | 输出 Service 定义                       |

------

## 七、验证命令结果

创建完成后可使用以下命令验证：

```bash
kubectl get service
```

或查看详细信息：

```bash
kubectl describe service my-service
```

------

## 八、实际案例

### 🎯 案例：暴露一个 Web 应用

1️⃣ 创建 Deployment：

```bash
kubectl create deployment web-deploy --image=nginx
```

2️⃣ 暴露为 NodePort 服务：

```bash
kubectl expose deployment web-deploy --port=80 --type=NodePort
```

3️⃣ 查看 Service 信息：

```bash
kubectl get service web-deploy
```

输出示例：

```
NAME          TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
web-deploy    NodePort    10.98.127.45   <none>        80:30574/TCP   1m
```

4️⃣ 在浏览器中访问节点 IP + 端口：

```
http://<NodeIP>:30574
```

即可访问 nginx 页面。

------

## 九、常见问题与排查

| 问题                  | 原因                       | 解决方法                                    |
| --------------------- | -------------------------- | ------------------------------------------- |
| Service 未分配外部 IP | 本地集群或未启用云负载均衡 | 使用 NodePort 或手动配置 Ingress            |
| 访问不到 Pod          | selector 不匹配            | 使用 `kubectl describe svc` 检查 `Selector` |
| 指定的端口冲突        | 端口被占用或超出范围       | 修改 NodePort 或使用默认分配                |
| `resource not found`  | 对象名称错误               | 先用 `kubectl get` 查确认资源名称           |

------

## 十、与其他命令的区别

| 命令             | 功能                   |
| ---------------- | ---------------------- |
| `kubectl run`    | 创建 Pod               |
| `kubectl create` | 从文件或参数创建资源   |
| `kubectl expose` | 暴露现有资源为 Service |
| `kubectl apply`  | 创建或更新资源         |
| `kubectl delete` | 删除资源               |

------

## 十一、最佳实践建议

1. ✅ **推荐使用 `kubectl expose` 快速创建服务**，尤其在开发与测试阶段。
2. ✅ **生产环境**中建议使用 YAML 文件并通过 `kubectl apply` 管理。
3. ✅ 使用 `--dry-run -o yaml` 生成模板后再提交，可避免错误配置。
4. ✅ 若是外部访问，请明确选择合适的 Service 类型（NodePort / LoadBalancer）。
5. ✅ 定期使用 `kubectl describe svc` 检查端口映射与 selector 是否正确。

------

## 📘 常用命令速查表

| 功能                | 命令                                                         |
| ------------------- | ------------------------------------------------------------ |
| 暴露 Deployment     | `kubectl expose deployment myapp --port=80 --target-port=8080` |
| 暴露 Pod            | `kubectl expose pod mypod --port=80`                         |
| 暴露为 NodePort     | `kubectl expose deployment web --type=NodePort --port=80`    |
| 暴露为 LoadBalancer | `kubectl expose deployment web --type=LoadBalancer --port=80` |
| 暴露为 ExternalName | `kubectl expose deployment web --type=ExternalName --external-name=example.com` |
| 生成 YAML 文件      | `kubectl expose deployment web --port=80 --dry-run=client -o yaml > svc.yaml` |

------

## 十二、总结

`kubectl expose` 是一个功能简洁却非常实用的命令，用于快速生成 Service，从而让应用在集群内部或外部被访问。
 掌握其常用参数和类型（`ClusterIP`、`NodePort`、`LoadBalancer`、`ExternalName`）后，可以灵活应对不同的服务暴露需求



# 🧭 kubectl logs 

## 一、命令概述

`kubectl logs` 是 Kubernetes 中用于**查看 Pod 中容器日志输出**的命令。
 它帮助开发者或运维人员快速排查容器运行问题、调试应用、监控运行状态。

Pod 中每个容器的标准输出（`stdout`）和标准错误输出（`stderr`）都会被 Kubernetes 收集，
 而 `kubectl logs` 就是读取这些日志的主要方式。

------

## 二、命令语法

```bash
kubectl logs [POD_NAME] [-c CONTAINER_NAME] [flags]
```

### 或查看某个 Job/Deployment 对应 Pod 的日志：

```bash
kubectl logs job/<job-name>
kubectl logs deployment/<deploy-name> -l app=myapp
```

------

## 三、常用参数说明

| 参数                                | 说明                                             |
| ----------------------------------- | ------------------------------------------------ |
| `<pod>`                             | 指定 Pod 名称                                    |
| `-c, --container`                   | 指定 Pod 中的容器名称（当 Pod 有多个容器时必须） |
| `-f, --follow`                      | 实时持续输出日志（类似 `tail -f`）               |
| `--tail=N`                          | 仅显示最后 N 行日志                              |
| `--since=5m`                        | 显示最近 5 分钟的日志                            |
| `--since-time=2025-11-13T09:00:00Z` | 显示指定时间之后的日志                           |
| `--limit-bytes=N`                   | 限制输出的日志字节数                             |
| `--timestamps`                      | 在每行日志前显示时间戳                           |
| `-l, --selector`                    | 按标签选择 Pod                                   |
| `--previous`                        | 查看容器上一次崩溃（重启前）的日志               |
| `--namespace`                       | 指定命名空间                                     |

------

## 四、基本用法

### 1️⃣ 查看单个 Pod 的日志

```bash
kubectl logs my-pod
```

显示 Pod 中第一个容器的日志（如果只有一个容器）。

------

### 2️⃣ 指定容器查看日志

```bash
kubectl logs my-pod -c nginx
```

当一个 Pod 含多个容器时，必须使用 `-c` 指定容器名。

------

### 3️⃣ 实时查看日志（持续输出）

```bash
kubectl logs -f my-pod
```

> 类似 Linux 命令 `tail -f`，会持续输出新增日志。

------

### 4️⃣ 查看最近 N 行日志

```bash
kubectl logs my-pod --tail=100
```

仅显示最后 100 行。

------

### 5️⃣ 查看指定时间范围的日志

```bash
kubectl logs my-pod --since=10m
```

显示最近 10 分钟的日志。

------

### 6️⃣ 显示带时间戳的日志

```bash
kubectl logs my-pod --timestamps
```

------

### 7️⃣ 查看崩溃重启前的日志

```bash
kubectl logs my-pod --previous
```

用于调试 Pod 因错误重启的情况（CrashLoopBackOff）。

------

### 8️⃣ 查看命名空间中的 Pod 日志

```bash
kubectl logs my-pod -n test
```

------

### 9️⃣ 查看带标签的多个 Pod 日志

```bash
kubectl logs -l app=nginx
```

或实时查看：

```bash
kubectl logs -f -l app=nginx
```

------

## 五、进阶用法

### 1️⃣ 查看 Deployment 对应 Pod 的日志

```bash
kubectl logs deployment/nginx-deployment
```

或使用标签：

```bash
kubectl logs -l app=nginx
```

------

### 2️⃣ 查看 Job 的日志

```bash
kubectl logs job/my-batch-job
```

------

### 3️⃣ 查看所有 Pod 的日志（带标签）

```bash
kubectl logs -f -l app=myapp --all-containers=true
```

------

### 4️⃣ 同时查看多个容器日志

```bash
kubectl logs my-pod --all-containers=true
```

------

### 5️⃣ 限制输出大小

```bash
kubectl logs my-pod --limit-bytes=50000
```

仅显示最多 50KB 的日志。

------

### 6️⃣ 输出到文件保存

```bash
kubectl logs my-pod > pod.log
```

或带时间戳：

```bash
kubectl logs my-pod --timestamps > pod.log
```

------

## 六、常见错误与解决方案

| 错误信息                                                     | 原因                   | 解决方法                           |
| ------------------------------------------------------------ | ---------------------- | ---------------------------------- |
| `Error from server (BadRequest): container ... not found`    | Pod 有多个容器但未指定 | 使用 `-c` 参数指定容器名           |
| `Error from server (NotFound): pods "xxx" not found`         | Pod 名称或命名空间错误 | 检查命名空间或 Pod 名称            |
| `Error from server (BadRequest): previous terminated container not found` | 容器未重启             | 去掉 `--previous` 参数             |
| `no logs available`                                          | Pod 尚未启动或无输出   | 检查 Pod 状态（`kubectl get pod`） |
| `a container name must be specified`                         | Pod 有多个容器         | 加 `-c` 指定容器名                 |

------

## 七、实战案例

### 🧩 1. 调试崩溃容器

```bash
kubectl get pod my-pod
# 状态为 CrashLoopBackOff
kubectl logs my-pod --previous
```

### 🧩 2. 实时监控服务日志

```bash
kubectl logs -f -l app=web --all-containers
```

### 🧩 3. 获取日志并保存到文件

```bash
kubectl logs -f my-pod > /var/logs/my-pod.log
```

### 🧩 4. 调试命名空间内多个 Pod

```bash
kubectl logs -l app=nginx -n production
```

------

## 八、与其他命令的配合使用

| 命令                   | 功能                | 组合示例                             |
| ---------------------- | ------------------- | ------------------------------------ |
| `kubectl get pods`     | 查看 Pod 名称       | `kubectl get pods -n test`           |
| `kubectl describe pod` | 查看 Pod 事件与状态 | `kubectl describe pod my-pod`        |
| `kubectl exec`         | 进入容器调试        | `kubectl exec -it my-pod -- /bin/sh` |
| `kubectl logs`         | 查看日志输出        | `kubectl logs -f my-pod`             |

------

## 九、最佳实践建议

✅ **使用标签选择器**查看日志，减少手动输入 Pod 名。
 ✅ **配合 `-f` 实时监控**运行状态。
 ✅ **CrashLoopBackOff 调试**时务必使用 `--previous`。
 ✅ **日志量大时使用 `--tail` 或 `--since`** 限制输出。
 ✅ **对多容器 Pod 加 `--all-containers=true`**。
 ✅ **可通过重定向输出保存日志以备分析**。
 ✅ **在生产环境中推荐使用集中式日志系统（如 ELK、Loki）**。

------

## 十、命令速查表

| 功能                 | 命令示例                                    |
| -------------------- | ------------------------------------------- |
| 查看 Pod 日志        | `kubectl logs my-pod`                       |
| 查看指定容器日志     | `kubectl logs my-pod -c nginx`              |
| 实时输出日志         | `kubectl logs -f my-pod`                    |
| 查看最近 100 行日志  | `kubectl logs my-pod --tail=100`            |
| 查看最近 5 分钟日志  | `kubectl logs my-pod --since=5m`            |
| 查看崩溃容器日志     | `kubectl logs my-pod --previous`            |
| 查看带标签的多个 Pod | `kubectl logs -l app=web`                   |
| 查看所有容器日志     | `kubectl logs my-pod --all-containers=true` |
| 输出日志到文件       | `kubectl logs my-pod > my-pod.log`          |

------

## 十一、总结

`kubectl logs` 是 Kubernetes 日常运维和调试中最常用的诊断命令之一。
 它让你能够快速获取容器输出日志，分析服务异常、启动错误、网络故障等问题。

> 📖 一句话总结：
>  **`kubectl logs` 是容器世界里的 “tail -f”，
>  让你直接看到应用在集群中的真实运行状态。**

------



# 🧭 kubectl set 

## 一、命令概述

`kubectl set` 是 Kubernetes 中用于**修改现有资源的属性**的命令集合。
 它不是单一命令，而是一组 **子命令（subcommands）** 的集合，可用来动态修改资源的镜像、环境变量、资源限制、服务账号等配置。

换句话说：

> `kubectl apply` 修改资源是“声明式”的（改 YAML 文件），
>  而 `kubectl set` 修改资源是“命令式”的（直接执行命令）。

------

## 二、命令语法

```bash
kubectl set SUBCOMMAND [OPTIONS]
```

常用子命令包括：

| 子命令           | 功能                                             |
| ---------------- | ------------------------------------------------ |
| `image`          | 更新容器镜像                                     |
| `env`            | 设置或移除环境变量                               |
| `resources`      | 更新 CPU / 内存 限制                             |
| `selector`       | 更新资源的标签选择器                             |
| `serviceaccount` | 指定使用的 ServiceAccount                        |
| `subject`        | 修改 RoleBinding 或 ClusterRoleBinding 的主体    |
| `volume`         | 修改 Pod 的挂载卷                                |
| `probe`          | 配置健康检查（liveness/readiness/startup probe） |

------

## 三、常用子命令详解

------

### 🧩 1️⃣ `kubectl set image`

**功能：** 修改 Pod/Deployment/DaemonSet/StatefulSet 的容器镜像。

#### 基本语法：

```bash
kubectl set image <resource>/<name> <container>=<image> [options]
```

#### 示例：

修改 Deployment 中的镜像：

```bash
kubectl set image deployment/nginx-deploy nginx=nginx:1.25
```

一次修改多个容器：

```bash
kubectl set image deployment/web web=nginx:1.25 sidecar=busybox
```

强制回滚：

```bash
kubectl rollout undo deployment/nginx-deploy
```

验证修改结果：

```bash
kubectl get deployment nginx-deploy -o wide
```

------

### 🧩 2️⃣ `kubectl set env`

**功能：** 设置或删除资源的环境变量。

#### 基本语法：

```bash
kubectl set env <resource>/<name> KEY=VALUE [options]
```

#### 示例：

为 Deployment 添加环境变量：

```bash
kubectl set env deployment/nginx-deploy ENV=prod LOG_LEVEL=info
```

删除环境变量：

```bash
kubectl set env deployment/nginx-deploy ENV-
```

从 ConfigMap 或 Secret 引入环境变量：

```bash
kubectl set env deployment/web --from=configmap/app-config
kubectl set env deployment/web --from=secret/db-secret
```

------

### 🧩 3️⃣ `kubectl set resources`

**功能：** 动态调整容器的资源请求和限制。

#### 基本语法：

```bash
kubectl set resources <resource>/<name> -c <container> \
  --limits=cpu=500m,memory=256Mi --requests=cpu=200m,memory=128Mi
```

#### 示例：

修改 Deployment 中的资源配置：

```bash
kubectl set resources deployment/nginx-deploy -c nginx \
  --limits=cpu=1000m,memory=512Mi --requests=cpu=250m,memory=128Mi
```

------

### 🧩 4️⃣ `kubectl set serviceaccount`

**功能：** 修改 PodTemplate 中使用的 ServiceAccount。

#### 示例：

```bash
kubectl set serviceaccount deployment/nginx-deploy nginx-sa
```

> 用于让 Pod 以特定的 ServiceAccount 身份运行（影响访问权限）。

------

### 🧩 5️⃣ `kubectl set selector`

**功能：** 修改资源的标签选择器（仅部分资源支持）。

#### 示例：

```bash
kubectl set selector service/my-svc app=myapp,env=prod
```

------

### 🧩 6️⃣ `kubectl set subject`

**功能：** 修改 RBAC 中的角色绑定主体（RoleBinding / ClusterRoleBinding）。

#### 示例：

为 RoleBinding 添加新用户：

```bash
kubectl set subject rolebinding admin --user=alice
```

删除某个用户：

```bash
kubectl set subject rolebinding admin --remove --user=bob
```

------

### 🧩 7️⃣ `kubectl set volume`

**功能：** 为 Deployment / PodTemplate 设置或更新卷挂载。

#### 示例：

```bash
kubectl set volume deployment/nginx-deploy --add \
  --name=config-volume --mount-path=/etc/nginx/conf.d --configmap-name=nginx-config
```

删除卷挂载：

```bash
kubectl set volume deployment/nginx-deploy --remove --name=config-volume
```

------

### 🧩 8️⃣ `kubectl set probe`

**功能：** 设置容器健康检查探针（Kubernetes v1.18+）。

#### 示例：

为容器添加 liveness probe：

```bash
kubectl set probe deployment/nginx-deploy --liveness \
  --get-url=http://:80/healthz --initial-delay-seconds=10
```

添加 readiness probe：

```bash
kubectl set probe deployment/nginx-deploy --readiness \
  --get-url=http://:80/ready --initial-delay-seconds=5
```

------

## 四、常用选项（Flags）

| 参数               | 说明                           |
| ------------------ | ------------------------------ |
| `--dry-run=client` | 仅输出修改结果，不实际应用     |
| `-o yaml/json`     | 输出修改后的 YAML/JSON         |
| `--all`            | 作用于所有匹配资源             |
| `-l, --selector`   | 使用标签选择器过滤目标资源     |
| `--record`         | 在 annotation 中记录命令历史   |
| `--overwrite`      | 覆盖已存在的字段（如环境变量） |

------

## 五、典型用法示例

| 场景                  | 命令示例                                                     |
| --------------------- | ------------------------------------------------------------ |
| 修改镜像版本          | `kubectl set image deployment/web web=nginx:1.25`            |
| 添加环境变量          | `kubectl set env deployment/web LOG_LEVEL=debug`             |
| 删除环境变量          | `kubectl set env deployment/web LOG_LEVEL-`                  |
| 从 ConfigMap 导入变量 | `kubectl set env deployment/web --from=configmap/app-config` |
| 调整资源限制          | `kubectl set resources deployment/web --limits=cpu=500m,memory=256Mi` |
| 指定 ServiceAccount   | `kubectl set serviceaccount deployment/web web-sa`           |
| 添加卷挂载            | `kubectl set volume deployment/web --add --name=app-config --configmap-name=config` |
| 设置健康检查          | `kubectl set probe deployment/web --liveness --get-url=http://:8080/health` |

------

## 六、查看修改结果

修改后可立即查看效果：

```bash
kubectl get deployment web -o yaml
```

或查看 Pod 实际状态：

```bash
kubectl describe pod <pod-name>
```

------

## 七、与其他命令的对比

| 命令            | 功能                          | 特点                 |
| --------------- | ----------------------------- | -------------------- |
| `kubectl edit`  | 打开资源配置文件进行编辑      | 手动修改 YAML        |
| `kubectl patch` | 通过 JSON/YAML Patch 更新字段 | 适合精确修改         |
| `kubectl set`   | 通过命令式方式修改属性        | 简单高效             |
| `kubectl apply` | 声明式更新资源                | 推荐用于生产配置管理 |

------

## 八、常见错误与解决方案

| 错误                         | 原因                            | 解决方法                     |
| ---------------------------- | ------------------------------- | ---------------------------- |
| `error: no resources found`  | 资源不存在或命名空间错误        | 检查名称与命名空间           |
| `field is immutable`         | 修改了不可变字段（如 selector） | 删除后重新创建资源           |
| `invalid key=value`          | 环境变量语法错误                | 确保使用正确格式 `KEY=VALUE` |
| `error: container not found` | Pod 中无指定容器                | 使用 `-c` 指定容器名         |

------

## 九、最佳实践

✅ 使用 `--dry-run=client -o yaml` 预览修改效果
 ✅ 避免直接在生产环境使用 `set` 修改关键参数（建议使用 `apply`）
 ✅ 修改镜像后立即执行 `kubectl rollout status` 监控部署进度
 ✅ 使用标签选择器配合 `--all` 批量更新
 ✅ 对资源限制、环境变量变更建议纳入版本控制

------

## 十、命令速查表

| 功能                  | 命令                                                         |
| --------------------- | ------------------------------------------------------------ |
| 修改镜像              | `kubectl set image deployment/web web=nginx:1.25`            |
| 添加环境变量          | `kubectl set env deployment/web ENV=prod`                    |
| 从 ConfigMap 导入变量 | `kubectl set env deployment/web --from=configmap/app-config` |
| 调整资源限制          | `kubectl set resources deployment/web --limits=cpu=1,memory=512Mi` |
| 修改 ServiceAccount   | `kubectl set serviceaccount deployment/web web-sa`           |
| 修改卷挂载            | `kubectl set volume deployment/web --add --name=config --configmap-name=cfg` |
| 修改探针              | `kubectl set probe deployment/web --liveness --get-url=http://:80/health` |

------

## 十一、总结

`kubectl set` 是一个强大、灵活的命令集，
 可在不修改 YAML 文件的情况下快速调整资源配置。
 它适合临时修正配置、调试、或批量变更操作。

> 📖 一句话总结：
>  **“kubectl apply 管理配置文件，kubectl set 快速修改运行中对象。”**



# 🧭 kubectl exec 

------

## 一、命令概述

`kubectl exec` 是 Kubernetes 中非常常用的命令之一，
 用于在 **Pod 的容器内执行命令**，类似于通过 `docker exec` 进入容器。

它常被用于：

- 进入容器排查问题；
- 查看运行时日志或配置；
- 执行临时调试命令；
- 启动交互式 Shell。

------

## 二、命令语法

```bash
kubectl exec [options] POD [-c CONTAINER] -- COMMAND [args...]
```

或交互式执行：

```bash
kubectl exec -it POD [-c CONTAINER] -- /bin/sh
```

------

## 三、参数说明

| 参数              | 说明                                                  |
| ----------------- | ----------------------------------------------------- |
| `POD`             | 目标 Pod 的名称                                       |
| `-c, --container` | 指定要进入的容器名称（一个 Pod 有多个容器时必须指定） |
| `-i`              | 保持标准输入（stdin）打开，用于交互式命令             |
| `-t`              | 分配一个伪终端（tty）                                 |
| `--namespace`     | 指定命名空间（默认为 default）                        |
| `--`              | 分隔符，表示后面为要执行的命令                        |
| `--stdin`         | 与 `-i` 类似，保持标准输入打开                        |
| `--tty`           | 与 `-t` 类似，分配 TTY 终端                           |

------

## 四、常用示例

### 🧩 1️⃣ 在容器中执行单条命令

```bash
kubectl exec my-pod -- ls /usr/share/nginx/html
```

------

### 🧩 2️⃣ 进入容器的交互式 Shell

```bash
kubectl exec -it my-pod -- /bin/bash
```

或容器使用 `sh`：

```bash
kubectl exec -it my-pod -- /bin/sh
```

> 💡 **提示**：部分镜像（如 `nginx`、`busybox`）不包含 bash，需要改用 `/bin/sh`。

------

### 🧩 3️⃣ 指定容器名称执行命令

如果一个 Pod 有多个容器，例如：

```bash
kubectl get pod my-pod -o jsonpath='{.spec.containers[*].name}'
```

输出：

```
nginx sidecar
```

则执行命令时要加 `-c` 参数：

```bash
kubectl exec -it my-pod -c sidecar -- /bin/sh
```

------

### 🧩 4️⃣ 查看容器环境变量

```bash
kubectl exec my-pod -- printenv
```

------

### 🧩 5️⃣ 执行脚本或多条命令

执行多条命令：

```bash
kubectl exec my-pod -- sh -c "cd /tmp && ls && cat file.txt"
```

执行脚本文件：

```bash
kubectl exec my-pod -- sh -c "/scripts/run.sh"
```

------

### 🧩 6️⃣ 跨命名空间执行命令

```bash
kubectl exec -n kube-system -it coredns-xxxxx -- sh
```

------

### 🧩 7️⃣ 与 `kubectl cp` 配合使用

先将脚本复制进 Pod：

```bash
kubectl cp ./test.sh my-pod:/tmp/test.sh
```

再执行：

```bash
kubectl exec my-pod -- sh /tmp/test.sh
```

------

## 五、进阶用法

### ✅ 在多个 Pod 中批量执行命令（配合 `xargs`）

```bash
kubectl get pods -l app=nginx -o name | xargs -I {} kubectl exec {} -- hostname
```

------

### ✅ 查看容器日志文件

```bash
kubectl exec my-pod -- cat /var/log/nginx/access.log
```

------

### ✅ 检查容器网络连接

```bash
kubectl exec -it my-pod -- curl -I http://localhost:8080
```

------

### ✅ 检查容器文件系统

```bash
kubectl exec my-pod -- df -h
kubectl exec my-pod -- du -sh /var/log
```

------

## 六、常见错误与解决方法

| 错误信息                                       | 可能原因                     | 解决方案                               |
| ---------------------------------------------- | ---------------------------- | -------------------------------------- |
| `error: unable to upgrade connection`          | 网络问题或 Pod 未运行        | 检查 Pod 状态是否为 `Running`          |
| `error: container not found`                   | Pod 有多个容器，未指定容器名 | 使用 `-c` 指定容器名称                 |
| `rpc error: code = 2 desc = oci runtime error` | 容器崩溃或镜像无 Shell       | 确认容器是否可进入；镜像可用 `/bin/sh` |
| `no TTY present`                               | 忘记加 `-t` 参数             | 使用 `-it` 分配交互式终端              |
| `exec failed: container not found`             | Pod 重建导致名称变化         | 重新获取 Pod 名称再执行                |

------

## 七、调试技巧 💡

| 场景           | 命令                                       |
| -------------- | ------------------------------------------ |
| 查看容器网络   | `kubectl exec -it pod -- netstat -tunlp`   |
| 查看进程       | `kubectl exec -it pod -- ps aux`           |
| 查看环境变量   | `kubectl exec pod -- env`                  |
| 测试网络连通性 | `kubectl exec pod -- ping -c 3 10.244.0.1` |
| 查看文件内容   | `kubectl exec pod -- cat /etc/resolv.conf` |

------

## 八、与其他命令的对比

| 命令                   | 功能              | 特点                 |
| ---------------------- | ----------------- | -------------------- |
| `kubectl exec`         | 在容器中执行命令  | 实时执行，适合调试   |
| `kubectl logs`         | 查看容器日志      | 只读输出，不进入容器 |
| `kubectl cp`           | 复制文件至/从容器 | 文件传输操作         |
| `kubectl port-forward` | 转发本地端口      | 调试网络连接         |

------

## 九、常用快捷命令汇总

| 功能         | 命令                                                       |
| ------------ | ---------------------------------------------------------- |
| 进入容器     | `kubectl exec -it pod -- sh`                               |
| 查看目录     | `kubectl exec pod -- ls /app`                              |
| 查看环境变量 | `kubectl exec pod -- env`                                  |
| 查看日志文件 | `kubectl exec pod -- cat /var/log/app.log`                 |
| 测试网络连接 | `kubectl exec pod -- curl http://localhost:8080`           |
| 多条命令     | `kubectl exec pod -- sh -c "cd /app && ls && cat log.txt"` |
| 指定容器     | `kubectl exec -it pod -c container-name -- bash`           |

------

## 十、最佳实践 ✅

1. **使用 `-it` 进入交互式终端**（例如 `/bin/sh`）。

2. **避免在生产环境容器中频繁修改文件**，推荐用 ConfigMap/Secret 管理配置。

3. **排查 Pod 状态时，先执行**：

   ```bash
   kubectl get pod <pod-name> -o wide
   ```

   确认 Pod 处于 `Running` 状态。

4. **调试短命容器**时，可以使用：

   ```bash
   kubectl run debug --image=busybox -it --rm --restart=Never -- sh
   ```

5. **与 `kubectl cp` 搭配**使用，用于上传脚本进行测试。

------

## 十一、命令速查表

| 场景                          | 命令示例                                           |
| ----------------------------- | -------------------------------------------------- |
| 进入容器 Shell                | `kubectl exec -it my-pod -- /bin/sh`               |
| 查看文件内容                  | `kubectl exec my-pod -- cat /etc/hosts`            |
| 执行多条命令                  | `kubectl exec my-pod -- sh -c "ls / && echo done"` |
| 查看环境变量                  | `kubectl exec my-pod -- env`                       |
| 指定容器执行命令              | `kubectl exec -it my-pod -c sidecar -- /bin/bash`  |
| 执行脚本文件                  | `kubectl exec my-pod -- sh /tmp/test.sh`           |
| 在所有 nginx Pod 中查看主机名 | `kubectl get pods -l app=nginx -o name             |

------

## 十二、总结

- `kubectl exec` 是 Kubernetes 调试中最常用的命令之一；
- 通过它可以直接在容器内执行命令，快速定位问题；
- 常与 `kubectl logs`、`kubectl cp`、`kubectl port-forward` 等命令结合使用；
- 在生产环境使用时，应注意安全与权限控制（RBAC）。

> 📖 一句话总结：
>  **`kubectl exec` 是你进入 Kubernetes 容器世界的“后门”。**

------

# 🧭 kubectl describe 

------

## 一、命令概述

`kubectl describe` 是 Kubernetes 中用于**查看资源详细信息**的命令。
 与 `kubectl get` 不同，`describe` 会输出**更详细的人类可读信息**，
 包括状态、事件、容器、卷、策略、调度信息等。

> 📖 一句话总结：
>  `kubectl get` 是“概览”，
>  `kubectl describe` 是“深度解剖”。

------

## 二、命令语法

```bash
kubectl describe (TYPE [NAME | -l label] | TYPE/NAME) [options]
```

------

## 三、参数说明

| 参数               | 说明                                           |
| ------------------ | ---------------------------------------------- |
| `TYPE`             | 资源类型，如 pod、node、service、deployment 等 |
| `NAME`             | 资源名称                                       |
| `-n, --namespace`  | 指定命名空间                                   |
| `-l, --selector`   | 使用标签选择器过滤资源                         |
| `--show-events`    | 是否显示事件（默认为 true）                    |
| `--all-namespaces` | 显示所有命名空间的资源                         |
| `--chunk-size`     | 分块请求资源数量，防止过大结果超时             |

------

## 四、常见用法示例

------

### 🧩 1️⃣ 查看 Pod 详情

```bash
kubectl describe pod nginx-7c8f9b9f5b-xyz12
```

**输出示例（部分）：**

```
Name:         nginx-7c8f9b9f5b-xyz12
Namespace:    default
Node:         node1/192.168.1.10
Start Time:   Thu, 13 Nov 2025 09:00:00 +0800
Labels:       app=nginx
Status:       Running
Containers:
  nginx:
    Image:          nginx:1.25
    Port:           80/TCP
    State:          Running
    Ready:          True
    Restart Count:  0
Events:
  Type     Reason     Age   From               Message
  ----     ------     ----  ----               -------
  Normal   Scheduled  1m    default-scheduler  Successfully assigned nginx-7c8f9b9f5b-xyz12 to node1
  Normal   Pulled     1m    kubelet            Container image "nginx:1.25" already present on machine
```

📌 **重点内容：**

- **状态（Status）**：Running / Pending / CrashLoopBackOff
- **容器（Containers）**：镜像、端口、重启次数
- **事件（Events）**：调度、拉取镜像、启动、错误信息

------

### 🧩 2️⃣ 查看 Deployment 详情

```bash
kubectl describe deployment nginx-deploy
```

**可查看内容：**

- 副本数（Desired / Current / Updated）
- 滚动更新策略
- 镜像版本
- Pod 模板
- 最近事件（如滚动更新）

------

### 🧩 3️⃣ 查看 Node（节点）详情

```bash
kubectl describe node node1
```

**可查看内容：**

- 节点信息（IP、OS、Kubelet 版本）
- 资源分配（CPU、内存）
- Pod 分布情况
- 节点条件（Ready / NotReady）
- Taints（污点）
- 分配的 Pod 资源汇总

------

### 🧩 4️⃣ 查看 Service 详情

```bash
kubectl describe svc my-service
```

**可查看内容：**

- 类型（ClusterIP / NodePort / LoadBalancer）
- Cluster IP / External IP
- Selector
- 端口映射
- 关联的 Endpoints

------

### 🧩 5️⃣ 查看 Namespace 下所有 Pod 的详细信息

```bash
kubectl describe pods -n default
```

或使用标签过滤：

```bash
kubectl describe pods -l app=nginx
```

------

### 🧩 6️⃣ 查看 ReplicaSet、DaemonSet、StatefulSet 信息

```bash
kubectl describe rs my-rs
kubectl describe ds kube-proxy
kubectl describe sts mysql
```

------

### 🧩 7️⃣ 查看 Secret / ConfigMap 信息

```bash
kubectl describe configmap app-config
kubectl describe secret db-secret
```

> 注意：`describe secret` 不会直接显示 Secret 的明文内容。

------

### 🧩 8️⃣ 查看 Event 事件资源

```bash
kubectl describe event <event-name>
```

或列出所有事件：

```bash
kubectl get events --sort-by=.metadata.creationTimestamp
```

------

## 五、配合选项使用

| 命令示例                                    | 功能                   |
| ------------------------------------------- | ---------------------- |
| `kubectl describe pod nginx -n kube-system` | 指定命名空间           |
| `kubectl describe pod -l app=nginx`         | 按标签选择器过滤       |
| `kubectl describe pods --show-events=false` | 不显示事件             |
| `kubectl describe node --all-namespaces`    | 查看所有命名空间的节点 |

------

## 六、常见资源类型

| 类型                     | 说明         |
| ------------------------ | ------------ |
| `pods`                   | Pod 详情     |
| `nodes`                  | 节点详情     |
| `deployments`            | 部署详情     |
| `services`               | 服务详情     |
| `replicasets`            | 副本集详情   |
| `daemonsets`             | 守护进程集   |
| `statefulsets`           | 有状态副本集 |
| `configmaps`             | 配置详情     |
| `secrets`                | 密钥详情     |
| `events`                 | 事件详情     |
| `namespaces`             | 命名空间详情 |
| `persistentvolumeclaims` | PVC 详情     |
| `persistentvolumes`      | PV 详情      |

------

## 七、典型使用场景

| 场景                | 命令                                 | 说明                                |
| ------------------- | ------------------------------------ | ----------------------------------- |
| Pod 一直 Pending    | `kubectl describe pod <pod>`         | 查看事件，排查调度或资源问题        |
| Deployment 无法更新 | `kubectl describe deployment <name>` | 检查滚动更新策略、镜像拉取错误      |
| Node 不可用         | `kubectl describe node <name>`       | 查看节点条件（Ready、DiskPressure） |
| Service 无法访问    | `kubectl describe svc <name>`        | 检查端口、Selector 与 Endpoint      |
| PVC 绑定失败        | `kubectl describe pvc <name>`        | 查看绑定状态与事件                  |
| 查看 ConfigMap 内容 | `kubectl describe configmap <name>`  | 了解配置键值信息                    |

------

## 八、事件（Events）分析

在 `kubectl describe` 的输出中，**Events 区块**非常关键，
 可帮助你定位 Pod 启动失败或调度异常原因。

常见事件类型包括：

| 类型                 | 说明                                   |
| -------------------- | -------------------------------------- |
| **Normal**           | 正常事件（如已调度、镜像已拉取）       |
| **Warning**          | 异常事件（如拉取镜像失败、挂载卷失败） |
| **FailedScheduling** | 调度失败，可能是资源不足或节点污点     |
| **BackOff**          | 容器反复重启                           |
| **CrashLoopBackOff** | 容器启动后立即崩溃                     |
| **ImagePullBackOff** | 镜像拉取失败                           |

------

## 九、常见问题与解决思路

| 问题                 | 原因                     | 排查命令                       |
| -------------------- | ------------------------ | ------------------------------ |
| Pod Pending          | 节点资源不足、无匹配节点 | `kubectl describe pod <name>`  |
| Pod CrashLoopBackOff | 应用启动失败             | 查看 `Events` 与容器日志       |
| Service 无后端       | Selector 标签不匹配      | `kubectl describe svc <name>`  |
| PVC Pending          | 无可绑定 PV              | `kubectl describe pvc <name>`  |
| 节点 NotReady        | 节点网络或 kubelet 故障  | `kubectl describe node <name>` |

------

## 十、`describe` 与 `get` 的区别

| 对比项   | `kubectl get`  | `kubectl describe`     |
| -------- | -------------- | ---------------------- |
| 输出形式 | 简洁表格       | 详细文本               |
| 信息量   | 少             | 丰富（包括事件）       |
| 适用场景 | 快速查看状态   | 深入排查问题           |
| 支持选项 | `-o yaml/json` | 无格式化输出（仅文本） |

------

## 十一、常用命令速查表

| 场景                 | 命令                                    |
| -------------------- | --------------------------------------- |
| 查看 Pod 详情        | `kubectl describe pod my-pod`           |
| 查看 Deployment 详情 | `kubectl describe deployment my-deploy` |
| 查看 Service 详情    | `kubectl describe svc my-service`       |
| 查看 Node 详情       | `kubectl describe node node1`           |
| 查看 ConfigMap       | `kubectl describe configmap my-config`  |
| 查看 PVC 详情        | `kubectl describe pvc data-pvc`         |
| 查看某标签 Pod       | `kubectl describe pods -l app=nginx`    |
| 查看所有事件         | `kubectl get events -A`                 |

------

## 十二、最佳实践 ✅

1. **优先查看 Events**：这是诊断问题的关键。

2. **结合 `kubectl logs`** 使用，可快速定位 Pod 启动错误。

3. **结合 `kubectl get`** 查看概览，再用 `describe` 深入分析。

4. **调试 Pod 启动失败时**，先：

   ```bash
   kubectl describe pod <pod-name>
   ```

   然后再：

   ```bash
   kubectl logs <pod-name>
   ```

5. **可搭配 grep 提取关键信息**：

   ```bash
   kubectl describe pod my-pod | grep -A5 Events
   ```

------

## 十三、总结

- `kubectl describe` 提供 Kubernetes 资源的**详细运行状态**；
- 可查看容器配置、事件、调度信息，是排障第一工具；
- 建议与 `kubectl logs`、`kubectl get`、`kubectl exec` 联合使用；
- 重点关注输出末尾的 **Events 区块**。

> 📖 一句话总结：
>  **`kubectl get` 看整体，`kubectl describe` 找问题。**

------



# 🧭 kubectl events

------

## 一、命令概述

`kubectl events` 是 Kubernetes 中用于**查看集群中事件（Event）\**的命令。
 它显示了 Kubernetes 系统中对象（如 Pod、Node、Deployment、PVC 等）在运行过程中发生的各种\**状态变化、错误、调度、重启**等事件。

> 📖 一句话总结：
>  **`kubectl events` 是排查 Kubernetes 问题的“日志窗口”。**

它是对旧命令

```bash
kubectl get events
```

的增强替代，输出更清晰、格式更现代化，支持实时监控和高级过滤。

------

## 二、命令语法

```bash
kubectl events [flags]
```

或查看某个命名空间下的事件：

```bash
kubectl events -n <namespace>
```

------

## 三、参数说明

| 参数                   | 说明                                               |
| ---------------------- | -------------------------------------------------- |
| `-A, --all-namespaces` | 显示所有命名空间的事件                             |
| `-n, --namespace`      | 指定命名空间                                       |
| `--for <resource>`     | 仅显示与某资源相关的事件（例如 Pod 或 Deployment） |
| `--field-selector`     | 通过字段过滤（如 `involvedObject.kind=Pod`）       |
| `--watch`              | 实时持续输出事件                                   |
| `--sort-by`            | 按字段排序（如时间戳）                             |
| `--limit`              | 限制输出事件的数量                                 |
| `-o, --output`         | 输出格式：`wide`, `json`, `yaml`, `name` 等        |

------

## 四、事件类型与结构

每条事件记录包含以下关键信息：

| 字段                     | 含义                                                   |
| ------------------------ | ------------------------------------------------------ |
| **Type**                 | 事件类型：Normal / Warning                             |
| **Reason**               | 事件原因（如 `Pulled`、`Created`、`FailedScheduling`） |
| **Object**               | 触发事件的对象（如 Pod、Node）                         |
| **Source**               | 事件来源（kubelet、scheduler 等）                      |
| **Message**              | 详细描述                                               |
| **FirstSeen / LastSeen** | 首次与最近发生时间                                     |
| **Count**                | 事件重复发生次数                                       |

------

## 五、常见用法

------

### 🧩 1️⃣ 查看当前命名空间下的事件

```bash
kubectl events
```

等价于旧写法：

```bash
kubectl get events
```

输出示例：

```
LAST SEEN   TYPE      REASON              OBJECT                    MESSAGE
1m          Normal    Scheduled           pod/nginx-7c8f9b9f5b-xyz  Successfully assigned default/nginx to node1
1m          Normal    Pulled              pod/nginx-7c8f9b9f5b-xyz  Container image "nginx:1.25" already present on machine
1m          Normal    Created             pod/nginx-7c8f9b9f5b-xyz  Created container nginx
1m          Normal    Started             pod/nginx-7c8f9b9f5b-xyz  Started container nginx
```

------

### 🧩 2️⃣ 查看所有命名空间的事件

```bash
kubectl events -A
```

------

### 🧩 3️⃣ 查看特定资源的事件

查看某个 Pod 的事件：

```bash
kubectl events --for pod/nginx-7c8f9b9f5b-xyz
```

查看 Deployment 的事件：

```bash
kubectl events --for deployment/nginx-deploy
```

查看 Node 的事件：

```bash
kubectl events --for node/node1
```

------

### 🧩 4️⃣ 实时查看事件（类似 tail）

```bash
kubectl events --watch
```

或带命名空间：

```bash
kubectl events -n kube-system --watch
```

> 💡 类似日志“实时刷新”，非常适合部署或调试过程中使用。

------

### 🧩 5️⃣ 查看警告类事件

只查看 Warning 级别事件：

```bash
kubectl events --field-selector type=Warning
```

------

### 🧩 6️⃣ 查看特定对象类型事件

仅查看 Pod 相关事件：

```bash
kubectl events --field-selector involvedObject.kind=Pod
```

仅查看某个节点事件：

```bash
kubectl events --field-selector involvedObject.kind=Node
```

------

### 🧩 7️⃣ 按时间排序

```bash
kubectl events --sort-by=.metadata.creationTimestamp
```

------

### 🧩 8️⃣ 输出为 YAML 或 JSON 格式

```bash
kubectl events -o yaml
kubectl events -o json
```

可配合 `jq` 等工具进行分析：

```bash
kubectl events -o json | jq '.items[] | {type, reason, message}'
```

------

## 六、事件类型（Type）

| 类型        | 说明     | 示例                                                         |
| ----------- | -------- | ------------------------------------------------------------ |
| **Normal**  | 正常事件 | `Scheduled`, `Created`, `Started`                            |
| **Warning** | 警告事件 | `FailedScheduling`, `BackOff`, `CrashLoopBackOff`, `FailedMount` |

------

## 七、常见事件与含义

| 事件类型 | REASON                | 说明                             |
| -------- | --------------------- | -------------------------------- |
| Normal   | **Scheduled**         | Pod 被成功调度到某节点           |
| Normal   | **Pulled**            | 容器镜像已拉取                   |
| Normal   | **Started**           | 容器已启动                       |
| Warning  | **FailedScheduling**  | 调度失败（资源不足、节点污点）   |
| Warning  | **BackOff**           | 容器反复重启（CrashLoopBackOff） |
| Warning  | **FailedMount**       | 挂载卷失败（PVC 未绑定）         |
| Warning  | **ImagePullBackOff**  | 镜像拉取失败                     |
| Warning  | **Unhealthy**         | 健康检查失败                     |
| Warning  | **FailedCreate**      | 无法创建 Pod 或 ReplicaSet       |
| Normal   | **Killing**           | 容器被终止                       |
| Normal   | **ScalingReplicaSet** | Deployment 调整副本数            |

------

## 八、实际场景分析

| 场景                 | 命令                               | 说明                                 |
| -------------------- | ---------------------------------- | ------------------------------------ |
| Pod 一直 Pending     | `kubectl events --for pod/<name>`  | 检查是否因节点资源不足或污点调度失败 |
| Pod CrashLoopBackOff | `kubectl events --for pod/<name>`  | 检查是否镜像错误或启动命令错误       |
| PVC Pending          | `kubectl events --for pvc/<name>`  | 检查是否缺少 PV                      |
| Service 无响应       | `kubectl events --for svc/<name>`  | 检查端口或 Endpoints 状态            |
| 节点异常             | `kubectl events --for node/<name>` | 查看节点健康与 taint 状态            |

------

## 九、结合其他命令使用

| 组合命令                                                   | 功能                    |
| ---------------------------------------------------------- | ----------------------- |
| `kubectl describe pod <name>`                              | 查看事件详情与状态      |
| `kubectl logs <pod>`                                       | 分析容器内部日志        |
| `kubectl get events --sort-by=.metadata.creationTimestamp` | 旧格式事件列表          |
| `kubectl get pods --watch` + `kubectl events --watch`      | 实时监控 Pod 状态与事件 |

------

## 十、常见问题与解决方案

| 问题           | 原因                                 | 解决方法                                      |
| -------------- | ------------------------------------ | --------------------------------------------- |
| 没有事件输出   | 命名空间不对或事件已过期             | 使用 `-A` 或 `--for` 指定对象                 |
| Event 记录过多 | 历史事件未清理                       | 可重启 `kube-controller-manager` 或事件存储器 |
| Event 消失太快 | Kubernetes 默认事件保留 1 小时       | 修改 `--event-ttl` 参数延长时间               |
| 排查不到异常   | 某些错误不触发事件（如容器内部错误） | 配合 `kubectl logs` 使用                      |

------

## 十一、进阶：事件过滤技巧

| 过滤目标                     | 命令示例                                                     |
| ---------------------------- | ------------------------------------------------------------ |
| 查看 Pod 警告                | `kubectl events --field-selector type=Warning,involvedObject.kind=Pod` |
| 查看 Node 事件               | `kubectl events --field-selector involvedObject.kind=Node`   |
| 查看某 Deployment 的所有事件 | `kubectl events --for deployment/my-deploy`                  |
| 实时监控所有事件             | `kubectl events -A --watch`                                  |

------

## 十二、与 `kubectl get events` 的区别

| 对比项          | `kubectl get events`   | `kubectl events`                   |
| --------------- | ---------------------- | ---------------------------------- |
| 是否新版推荐    | ❌ 旧命令（将逐步弃用） | ✅ 新推荐命令                       |
| 输出格式        | 表格（不对齐）         | 清晰分栏                           |
| 实时监控        | 不支持                 | ✅ 支持 `--watch`                   |
| 过滤能力        | 较弱                   | ✅ 支持 `--for`、`--field-selector` |
| 输出格式        | 支持 `yaml/json`       | ✅ 更完整                           |
| Kubernetes 版本 | 1.27+ 推荐使用         | 官方新标准                         |

------

## 十三、命令速查表

| 任务                   | 命令                                                   |
| ---------------------- | ------------------------------------------------------ |
| 查看当前命名空间事件   | `kubectl events`                                       |
| 查看所有命名空间事件   | `kubectl events -A`                                    |
| 查看某 Pod 事件        | `kubectl events --for pod/<name>`                      |
| 查看某 Deployment 事件 | `kubectl events --for deployment/<name>`               |
| 实时监控事件           | `kubectl events --watch`                               |
| 仅显示警告事件         | `kubectl events --field-selector type=Warning`         |
| 按时间排序             | `kubectl events --sort-by=.metadata.creationTimestamp` |
| 输出 YAML 格式         | `kubectl events -o yaml`                               |

------

## 十四、最佳实践 ✅

1. **排查任何 Pod 问题的第一步**：

   ```bash
   kubectl events --for pod/<name>
   ```

2. **实时监控部署发布**：

   ```bash
   kubectl rollout restart deployment/web && kubectl events --watch
   ```

3. **配合日志分析**：
    先看事件，再看日志：

   ```bash
   kubectl events --for pod/<name>
   kubectl logs <name>
   ```

4. **关注 Warning 事件**：
    它们通常揭示了调度失败、镜像错误、卷挂载问题。

5. **定期导出事件记录用于审计**：

   ```bash
   kubectl events -A -o yaml > cluster-events.yaml
   ```

------

## 十五、总结

- `kubectl events` 是诊断 Kubernetes 问题的首选工具之一；
- 可以实时查看集群中所有对象的事件流；
- 对调度失败、镜像拉取失败、容器崩溃、卷挂载等问题尤为有用；
- 支持按类型、命名空间、资源进行精准过滤。

> 📖 一句话总结：
>  **`kubectl events` = Kubernetes 的“系统事件日志监控器”。**

------

