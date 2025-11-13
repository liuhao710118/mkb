# 🧭 kubectl create 命令使用教程

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
 以下是一份格式规范、适合教学或培训使用的 **《kubectl explain 命令使用教程》** 文档。内容涵盖命令功能、语法结构、常见用法、进阶技巧与实际示例，可直接作为内部培训资料或学习笔记使用。

------

好的 ✅
 以下是一份完整、结构化、可直接作为学习或培训资料使用的 **《kubectl delete 命令使用教程》**。
 内容包含命令功能、语法、使用示例、参数说明、注意事项、错误排查与最佳实践。

------

# 🧭 kubectl delete 命令使用教程

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





# 🧭 kubectl explain 命令使用教程

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



好的 ✅
 以下是一份结构清晰、适合教学、内部培训或项目文档使用的
 **《kubectl expose 命令使用教程》**。
 内容包括命令说明、语法、示例、选项说明、常见错误及最佳实践。

------

# 🧭 kubectl expose 命令使用教程

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
