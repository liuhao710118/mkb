在 Kubernetes 中，可以通过为 Deployment 添加 **annotations（注解）** 来记录每次更新的备注信息，方便后续回滚时参考。以下是具体操作方法：

------

## **方法一：通过 `kubectl annotate`添加备注**

在更新 Deployment 时，可以通过 `kubectl annotate`命令添加自定义注解（如 `kubernetes.io/change-cause`），这是 Kubernetes 官方推荐用于记录更新原因的方式。

### **示例：**

```
# 更新 Deployment 并记录原因
kubectl annotate deployment/my-deployment kubernetes.io/change-cause="修复了内存泄漏问题 v1.2.3" --record=false

# 或者直接更新镜像时添加备注
kubectl set image deployment/my-deployment app=my-image:v2.0 --record=false
kubectl annotate deployment/my-deployment kubernetes.io/change-cause="升级到v2.0版本，新增用户管理功能"
```

> **注意**：`--record`参数在较新版本的 Kubernetes 中已废弃，推荐使用 `annotate`手动添加注解。

------

## **方法二：在 YAML 中定义注解**

修改 Deployment 的 YAML 文件，在 `metadata.annotations`中添加备注：

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-deployment
  annotations:
    kubernetes.io/change-cause: "初始部署：基础版本 v1.0"
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-container
        image: my-image:v1.0
```

更新时修改注解内容：

```
annotations:
  kubernetes.io/change-cause: "更新：优化数据库查询性能 v1.1"
```

然后通过 `kubectl apply -f deployment.yaml`应用更新。

------

## **查看更新历史与备注**

使用以下命令查看 Deployment 的更新历史及备注：

```
kubectl rollout history deployment/my-deployment
```

**输出示例：**

```
deployment.apps/my-deployment 
REVISION  CHANGE-CAUSE
1         Initial deployment
2         修复了登录超时问题 v1.1
3         升级到v2.0版本，新增用户管理功能
```

------

## **回滚时参照备注**

回滚到指定版本时，可以通过历史记录中的备注选择目标版本：

```
# 查看历史并确定要回滚的版本（如 REVISION 2）
kubectl rollout undo deployment/my-deployment --to-revision=2
```

------

## **最佳实践建议**

1. **统一注解键名**：建议使用标准键 `kubernetes.io/change-cause`，或使用自定义键（如 `team/change-cause`）。

2. **CI/CD 集成**：在自动化流水线中自动添加 Git 提交信息或版本号作为备注，例如：

   ```
   kubectl annotate deployment/my-deployment kubernetes.io/change-cause="CI构建: $(git log -1 --pretty=%B)"
   ```

3. **保留历史版本**：确保 `spec.revisionHistoryLimit`设置足够大（默认10），以保留更多历史记录。

------

通过以上方法，你可以在每次更新 Deployment 时添加清晰的备注，并在回滚时快速识别对应的变更内容。