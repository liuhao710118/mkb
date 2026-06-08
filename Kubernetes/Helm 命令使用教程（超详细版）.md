# Helm 命令使用教程（超详细版）

Helm 是 Kubernetes 的包管理工具，相当于 Linux 中的 yum、apt，或者 Python 中的 pip。

作用：

- 安装应用
- 升级应用
- 回滚应用
- 卸载应用
- 管理 Kubernetes YAML 模板
- 管理应用版本

------

# 一、Helm 基本概念

## 1. Chart

Chart 就是 Helm 包。

类似：

```text
nginx-chart
├── Chart.yaml
├── values.yaml
├── templates
│   ├── deployment.yaml
│   ├── service.yaml
│   └── ingress.yaml
```

一个 Chart 就是一套 Kubernetes 资源模板。

------

## 2. Release

Release 是 Chart 安装后的实例。

例如：

```bash
helm install nginx nginx-chart
```

结果：

```text
Chart：nginx-chart

Release：nginx
```

查看：

```bash
helm list
```

输出：

```text
NAME    NAMESPACE
nginx   default
```

------

## 3. Repository

仓库类似：

```text
Docker Hub
Harbor
Maven
Nexus
```

Helm 也有仓库：

```text
https://charts.bitnami.com/bitnami
```

里面存放各种 Chart。

------

# 二、安装 Helm

查看版本：

```bash
helm version
```

输出：

```bash
version.BuildInfo{
Version:"v3.16.0"
}
```

------

# 三、仓库管理

------

## 查看仓库

```bash
helm repo list
```

输出：

```text
NAME      URL
bitnami   https://charts.bitnami.com/bitnami
```

------

## 添加仓库

### 添加 Bitnami

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
```

------

### 添加阿里云仓库

```bash
helm repo add aliyun https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts
```

------

## 更新仓库

类似：

```bash
apt update
```

执行：

```bash
helm repo update
```

输出：

```text
Update Complete.
```

------

## 删除仓库

```bash
helm repo remove bitnami
```

------

# 四、搜索 Chart

------

## 搜索本地仓库

```bash
helm search repo nginx
```

输出：

```text
NAME                    CHART VERSION
bitnami/nginx           18.1.0
```

------

## 搜索所有版本

```bash
helm search repo nginx --versions
```

------

## 搜索 Hub

```bash
helm search hub redis
```

例如：

```text
URL
https://artifacthub.io/packages/helm/bitnami/redis
```

------

# 五、下载 Chart

------

## 下载

```bash
helm pull bitnami/nginx
```

结果：

```text
nginx-18.1.0.tgz
```

------

## 下载并解压

```bash
helm pull bitnami/nginx --untar
```

结果：

```text
nginx/
```

------

## 指定版本下载

```bash
helm pull bitnami/nginx --version 18.1.0
```

------

# 六、安装应用

------

## 最简单安装

```bash
helm install nginx bitnami/nginx
```

结构：

```bash
helm install \
<release名称> \
<chart名称>
```

------

## 指定 Namespace

```bash
helm install nginx bitnami/nginx \
-n test \
--create-namespace
```

------

## 指定版本安装

```bash
helm install nginx bitnami/nginx \
--version 18.1.0
```

------

## 指定配置文件

```bash
helm install nginx bitnami/nginx \
-f values.yaml
```

------

## 设置参数

```bash
helm install nginx bitnami/nginx \
--set replicaCount=3
```

等价：

```yaml
replicaCount: 3
```

------

## 多参数

```bash
helm install nginx bitnami/nginx \
--set replicaCount=3 \
--set service.type=NodePort
```

------

## 安装前验证

```bash
helm install nginx bitnami/nginx \
--dry-run
```

不会真正创建资源。

------

# 七、查看 Release

------

## 查看所有 Release

```bash
helm list
```

------

## 查看所有 Namespace

```bash
helm list -A
```

------

## 查看指定 Namespace

```bash
helm list -n kube-system
```

------

## 查看详细状态

```bash
helm status nginx
```

输出：

```text
STATUS: deployed
```

------

# 八、查看资源

------

## 查看 Values

```bash
helm get values nginx
```

------

## 查看所有配置

```bash
helm get values nginx -a
```

------

## 查看 Manifest

```bash
helm get manifest nginx
```

相当于：

```bash
kubectl get yaml
```

------

## 查看 Notes

```bash
helm get notes nginx
```

------

## 查看全部信息

```bash
helm get all nginx
```

------

# 九、升级 Release

------

## 修改 values

```bash
helm upgrade nginx bitnami/nginx \
-f values.yaml
```

------

## 修改副本数

```bash
helm upgrade nginx bitnami/nginx \
--set replicaCount=5
```

------

## 安装或升级

CI/CD 经常使用：

```bash
helm upgrade --install nginx bitnami/nginx
```

作用：

```text
存在：
    升级

不存在：
    安装
```

------

# 十、回滚 Release

------

## 查看历史

```bash
helm history nginx
```

输出：

```text
REVISION
1
2
3
```

------

## 回滚上一版本

```bash
helm rollback nginx 2
```

------

## 查看状态

```bash
helm status nginx
```

------

# 十一、卸载 Release

------

## 删除

```bash
helm uninstall nginx
```

------

## 删除指定 Namespace

```bash
helm uninstall nginx -n test
```

------

# 十二、Chart 开发

------

## 创建模板

```bash
helm create mychart
```

生成：

```text
mychart/
├── Chart.yaml
├── values.yaml
├── templates/
├── charts/
└── .helmignore
```

------

## Chart.yaml

Chart 元数据：

```yaml
apiVersion: v2
name: mychart
description: test chart
type: application
version: 1.0.0
appVersion: "1.0"
```

------

字段说明：

| 字段        | 作用      |
| ----------- | --------- |
| apiVersion  | Chart版本 |
| name        | Chart名称 |
| description | 描述      |
| version     | Chart版本 |
| appVersion  | 应用版本  |

------

# 十三、Values 使用

values.yaml

```yaml
replicaCount: 2

image:
  repository: nginx
  tag: latest
```

------

Deployment

```yaml
replicas: {{ .Values.replicaCount }}

image:
  image: {{ .Values.image.repository }}
  tag: {{ .Values.image.tag }}
```

------

安装：

```bash
helm install nginx . \
-f values.yaml
```

------

# 十四、模板语法

------

## 变量

```yaml
{{ .Values.replicaCount }}
```

------

## if

```yaml
{{ if .Values.ingress.enabled }}

......

{{ end }}
```

------

## else

```yaml
{{ if .Values.test }}

A

{{ else }}

B

{{ end }}
```

------

## range

```yaml
{{ range .Values.list }}

{{ . }}

{{ end }}
```

------

values：

```yaml
list:
  - a
  - b
  - c
```

输出：

```text
a
b
c
```

------

# 十五、调试命令

------

## 渲染模板

不连接 Kubernetes。

```bash
helm template nginx .
```

生成：

```yaml
Deployment
Service
Ingress
```

------

## 检查 Chart

```bash
helm lint .
```

检查：

```text
Chart.yaml
模板语法
values.yaml
```

------

## Dry Run

```bash
helm install nginx . \
--dry-run
```

------

## 显示 Debug

```bash
helm install nginx . \
--dry-run \
--debug
```

------

# 十六、生产环境最常用命令

查看：

```bash
helm list -A
```

安装：

```bash
helm install app ./chart
```

升级：

```bash
helm upgrade app ./chart
```

升级或安装：

```bash
helm upgrade --install app ./chart
```

查看状态：

```bash
helm status app
```

查看配置：

```bash
helm get values app
```

查看资源：

```bash
helm get manifest app
```

历史版本：

```bash
helm history app
```

回滚：

```bash
helm rollback app 1
```

删除：

```bash
helm uninstall app
```

------

# 十七、企业级 Helm 发布标准流程

实际生产环境一般遵循：

```text
开发编写Chart
       ↓
helm lint
       ↓
helm template
       ↓
GitLab
       ↓
Jenkins
       ↓
helm upgrade --install
       ↓
Kubernetes
```

完整发布命令通常是：

```bash
helm upgrade --install myapp ./chart \
-n prod \
--create-namespace \
-f values-prod.yaml \
--atomic \
--timeout 10m
```

参数说明：

| 参数      | 作用         |
| --------- | ------------ |
| --install | 不存在则安装 |
| --atomic  | 失败自动回滚 |
| --timeout | 超时时间     |
| -f        | 指定配置     |
| -n        | 指定命名空间 |

这条命令基本覆盖了生产环境 Helm 发布的核心场景。