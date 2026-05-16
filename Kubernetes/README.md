# Kubernetes 表格

| 核心模块           | 关键知识点/技能                                              | 掌握目标                            | 建议学习时间  |
| ------------------ | ------------------------------------------------------------ | ----------------------------------- | ------------- |
| **1. 核心概念**    | - Pod、Deployment、Service、Namespace - Node、Cluster、Label、Annotation - 声明式API与控制器模式 | 理解K8s核心抽象，能够描述各组件关系 | 1-2周         |
| **2. 架构理解**    | - Master节点组件（API Server、Scheduler等） - Worker节点组件（Kubelet、kube-proxy等） - etcd的作用与重要性 | 能画出K8s架构图并解释组件协作       | 1周           |
| **3. 环境搭建**    | - Minikube / Kind本地环境 - kubeadm集群部署 - 公有云托管服务（EKS/AKS/GKE） | 能够搭建开发和测试环境              | 1周           |
| **4. 工作负载**    | - Deployment（无状态应用） - StatefulSet（有状态应用） - DaemonSet、Job/CronJob | 熟练部署和管理各类应用              | 2-3周         |
| **5. 服务与网络**  | - Service类型（ClusterIP/NodePort/LB） - Ingress控制器（Nginx/Traefik） - 网络策略（NetworkPolicy） - DNS与服务发现 | 掌握服务暴露和网络配置              | 2周           |
| **6. 配置与存储**  | - ConfigMap与Secret管理 - Volume与PersistentVolume - StorageClass动态供应 - 存储卷类型（NFS/CSI等） | 掌握配置管理和持久化存储            | 2周           |
| **7. 安全**        | - RBAC权限管理 - ServiceAccount - Pod安全上下文 - 网络安全策略 | 能配置安全的K8s环境                 | 2周           |
| **8. 调度策略**    | - 资源请求与限制 - 节点选择与亲和性 - 污点与容忍 - 资源配额与限制范围 | 优化应用调度和资源利用              | 1-2周         |
| **9. 可观测性**    | - 日志收集方案（EFK） - 监控（Prometheus + Grafana） - 应用性能监控（APM） - 健康检查机制 | 建立完整的监控体系                  | 2-3周         |
| **10. 包管理**     | - Helm Chart开发与使用 - 私有Chart仓库搭建 - 应用生命周期管理 | 能使用Helm管理复杂应用              | 1-2周         |
| **11. CI/CD集成**  | - GitLab CI/K8s集成 - Jenkins流水线 - Argo CD GitOps实践 - Tekton流水线 | 建立自动化部署流水线                | 2-3周         |
| **12. 服务网格**   | - Istio核心功能（流量/安全/观测） - 金丝雀发布实现 - 分布式追踪 | 掌握服务网格基本使用                | 2-3周（可选） |
| **13. 集群运维**   | - 高可用集群部署 - 集群备份与恢复（Velero） - 证书管理与更新 - 集群升级策略 | 能够运维生产集群                    | 3-4周         |
| **14. 故障排查**   | - 系统化排错流程 - 常用诊断命令和工具 - 日志分析与调试技巧 - 网络问题诊断 | 能独立解决常见故障                  | 持续学习      |
| **15. 多集群管理** | - Karmada/Cluster API - 联邦集群管理 - 应用跨集群部署        | 掌握多集群管理方案                  | 1-2周（可选） |



# 概念集合

[kubernetes中的yaml是什么](./Kubernetes中YAML详解笔记.md)

[kubernetes中什么是POD？](./Kubernetes中的Pod详解笔记.md) [POD探针详解](./Kubernetes中Pod探针（Probe）详解.md)  [Kubernetes中Pod生命周期详解](Kubernetes中Pod生命周期详解.md) [initContainers、postStart和 preStop详情](initContainers、postStart和 preStop详情.md)

[kubernetes中的deployment](Kubernetes中Deployment与ReplicaSet详解.md) [滚动更新中的暂停pause与resume恢复](滚动更新中的pause与resume.md)

[Kubernetes中的StatefulSet详解](Kubernetes中的StatefulSet详解.md) [headlessService 返回固定的DNS 直接返回podIP](Kubernetes中的HeadlessService详解.md) [statefulset的灰度发布portation](StatefulSet 中的 Partition（分区更新）详解.md)

[kubernetes中的service](Kubernetes_Service详解.md)：流量分东西流量，和南北流量 东西流量就是集群内部横向通信，南北流量就是外部访问集群内部通信

[kubernetes中的ingress](Kubernetes_Ingress详解.md)：它是 Kubernetes 对外暴露 Web 服务最核心组件之一。

# 常用命令集

[kubeclt get命令](./commands/kubectl_get命令.md)： Kubernetes 中最基础、最常用的命令之一，用于**查询和列出集群中的资源**

[kubectl logs命令](commands/kubectl_logs命令详解.md)：查看容器标准输出日志

[kubectl exec命令](commands/kubectl_exec命令详解.md)：进入容器执行命令

[kubectl cp命令](commands/kubectl_cp命令详解.md)：Pod 与本地之间复制文件

[kubectl events命令](commands/kubectl_events详解.md)：查看 Kubernetes 事件（Event）

[kubectl debug命令](./commands/kubectl_debug命令详解.md)：排障调试命令 可以在不侵入的形式加入到已有的pod/node中进行调试

[kubectl label命令](commands/kubectl_label命令详解.md)：用于给 Kubernetes 资源（Node\Namespace\Deployment\Service\PVC\PV\ConfigMap\Secret）添加、修改、删除 **Label（标签）** 



# 帮助

[netshoot网络排障工具](./netshoot网络排障工具.md)：netshoot 是 Kubernetes / Docker 场景下非常经典的网络排障工具镜像 配合 **`kubectl debug`**最好
