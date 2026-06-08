# Kubernetes ConfigMap 与 Secret 详解（笔记版）

这是 Kubernetes 中最重要的配置管理内容之一。

很多初学者容易把它们理解成：

- ConfigMap = 配置文件
- Secret = 密码文件

实际上它们的作用远不止这些。

------

# 一、为什么需要 ConfigMap 和 Secret

假设有一个 Java 程序：

```yaml
mysql:
  host: 192.168.1.10
  port: 3306
  user: root
  password: 123456
```

最原始做法：

```dockerfile
COPY application.yml /app
```

把配置写死到镜像里。

问题：

### 问题1：环境变化

开发环境

```yaml
mysql.host=192.168.1.10
```

测试环境

```yaml
mysql.host=192.168.1.20
```

生产环境

```yaml
mysql.host=192.168.1.30
```

难道每个环境重新打包一次镜像？

显然不合理。

------

### 问题2：密码泄露

如果密码写进镜像：

```yaml
password=123456
```

那么：

```bash
docker save
```

导出镜像后别人就能看到。

------

### 问题3：修改配置需要重新构建

仅修改：

```yaml
server.port=8080
```

就要：

```bash
mvn package
docker build
docker push
```

非常麻烦。

------

因此 Kubernetes 提供：

| 资源      | 作用         |
| --------- | ------------ |
| ConfigMap | 保存普通配置 |
| Secret    | 保存敏感数据 |

实现：

```text
镜像
  ↓
配置分离
  ↓
运行时注入
```

即：

```text
Application
    ↓
ConfigMap
Secret
```

------

# 二、ConfigMap 详解

## 什么是 ConfigMap

官方定义：

```text
ConfigMap 用于保存非机密配置数据。
```

例如：

- mysql地址
- redis地址
- kafka地址
- 服务端口
- nginx配置
- application.yml

都属于 ConfigMap。

------

# 三、ConfigMap 数据结构

ConfigMap 本质：

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-config

data:
  key1: value1
  key2: value2
```

结构非常简单：

```text
ConfigMap
 ├── metadata
 └── data
      ├── key
      └── value
```

data中全部是：

```text
字符串
```

例如：

```yaml
data:
  mysql_host: 10.1.1.1
  mysql_port: "3306"
```

注意：

```yaml
3306
```

也会被存储成：

```text
"3306"
```

字符串。

------

# 四、创建 ConfigMap

## 方法1：YAML创建

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: mysql-config

data:
  host: 192.168.10.100
  port: "3306"
```

创建：

```bash
kubectl apply -f configmap.yaml
```

查看：

```bash
kubectl get configmap
```

------

## 方法2：命令创建

```bash
kubectl create configmap mysql-config \
--from-literal=host=192.168.10.100 \
--from-literal=port=3306
```

查看：

```bash
kubectl describe cm mysql-config
```

------

## 方法3：从文件创建

配置：

```ini
db.host=10.1.1.1
db.port=3306
```

创建：

```bash
kubectl create configmap app-config \
--from-file=app.properties
```

结果：

```yaml
data:
  app.properties: |
    db.host=10.1.1.1
    db.port=3306
```

------

## 方法4：从目录创建

```bash
conf/
├── nginx.conf
├── app.yml
└── logback.xml
```

创建：

```bash
kubectl create configmap my-config \
--from-file=conf/
```

结果：

```yaml
data:
  nginx.conf: ...
  app.yml: ...
  logback.xml: ...
```

------

# 五、Pod 使用 ConfigMap

有三种方式。

------

## 方式1：环境变量

ConfigMap

```yaml
data:
  MYSQL_HOST: 10.1.1.1
  MYSQL_PORT: "3306"
```

Pod

```yaml
env:
- name: MYSQL_HOST
  valueFrom:
    configMapKeyRef:
      name: mysql-config
      key: MYSQL_HOST
```

进入容器：

```bash
env
```

结果：

```bash
MYSQL_HOST=10.1.1.1
```

------

## 方式2：批量导入环境变量

ConfigMap

```yaml
data:
  MYSQL_HOST: 10.1.1.1
  MYSQL_PORT: "3306"
```

Pod

```yaml
envFrom:
- configMapRef:
    name: mysql-config
```

结果：

```bash
MYSQL_HOST=10.1.1.1
MYSQL_PORT=3306
```

全部导入。

------

## 方式3：挂载文件（最常用）

ConfigMap

```yaml
data:
  application.yml: |
    mysql:
      host: 10.1.1.1
```

Pod

```yaml
volumes:
- name: config
  configMap:
    name: app-config

containers:
- volumeMounts:
  - name: config
    mountPath: /config
```

结果：

```text
/config
 └── application.yml
```

容器直接读取文件。

------

# 六、ConfigMap 更新机制

修改：

```bash
kubectl edit cm app-config
```

修改：

```yaml
host: 10.1.1.100
```

------

如果通过 Volume 挂载：

```text
Pod
 ↓
自动感知
 ↓
文件更新
```

通常几十秒内同步。

------

如果通过环境变量：

```yaml
env:
```

不会更新。

必须重建 Pod。

------

记住：

| 方式   | 自动更新 |
| ------ | -------- |
| Volume | 是       |
| Env    | 否       |

这是面试高频题。

------

# 七、Secret 详解

## 什么是 Secret

官方定义：

```text
Secret 用于保存敏感信息。
```

例如：

- mysql密码
- redis密码
- token
- jwt
- tls证书
- ssh密钥

------

# 八、Secret 与 ConfigMap 区别

| 对比项     | ConfigMap | Secret   |
| ---------- | --------- | -------- |
| 用途       | 普通配置  | 敏感配置 |
| 存储       | 明文      | Base64   |
| 访问方式   | 相同      | 相同     |
| Volume挂载 | 支持      | 支持     |
| Env注入    | 支持      | 支持     |

很多人误解：

```text
Secret = 加密
```

实际上：

```text
Secret ≠ 加密
```

默认只是：

```text
Base64编码
```

例如：

```bash
echo -n 123456 | base64
```

结果：

```text
MTIzNDU2
```

任何人都能解码。

------

# 九、创建 Secret

## 方法1：命令创建

```bash
kubectl create secret generic mysql-secret \
--from-literal=username=root \
--from-literal=password=123456
```

查看：

```bash
kubectl get secret
```

------

## 方法2：YAML创建

先编码：

```bash
echo -n root | base64
```

结果：

```text
cm9vdA==
```

------

```bash
echo -n 123456 | base64
```

结果：

```text
MTIzNDU2
```

------

YAML：

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysql-secret

type: Opaque

data:
  username: cm9vdA==
  password: MTIzNDU2
```

------

# 十、Secret Type

## Opaque

最常见。

```yaml
type: Opaque
```

保存任意键值。

------

## TLS

保存证书。

```yaml
type: kubernetes.io/tls
```

创建：

```bash
kubectl create secret tls nginx-tls \
--cert=tls.crt \
--key=tls.key
```

通常给 Ingress 使用。

------

## Docker Registry

保存镜像仓库登录信息。

```yaml
type: kubernetes.io/dockerconfigjson
```

例如：

```bash
harbor.company.com
```

拉取私有镜像。

------

# 十一、Pod 使用 Secret

## 环境变量

```yaml
env:
- name: MYSQL_PASSWORD
  valueFrom:
    secretKeyRef:
      name: mysql-secret
      key: password
```

------

进入：

```bash
echo $MYSQL_PASSWORD
```

结果：

```text
123456
```

------

# 十二、Secret 挂载文件

Secret：

```yaml
data:
  password: MTIzNDU2
```

Pod：

```yaml
volumes:
- name: secret-volume
  secret:
    secretName: mysql-secret
```

挂载：

```yaml
volumeMounts:
- mountPath: /secret
  name: secret-volume
```

结果：

```text
/secret
 └── password
```

内容：

```text
123456
```

K8s自动解码。

------

# 十三、ConfigMap 与 Secret 底层存储

默认存储：

```text
API Server
    ↓
etcd
```

即：

```text
ConfigMap
Secret
     ↓
etcd
```

查看 etcd 实际内容：

```bash
ETCDCTL_API=3 etcdctl get /registry/secrets/default/mysql-secret
```

可以看到 Secret 数据。

------

生产环境必须：

```text
Encryption at Rest
```

开启 etcd 加密。

否则 Secret 仍然能被读取。

------

# 十四、ConfigMap 与 Secret 实战架构

典型 SpringBoot：

```text
Deployment
     │
     ├── ConfigMap
     │      ├── application.yml
     │      ├── redis地址
     │      └── kafka地址
     │
     └── Secret
            ├── mysql密码
            ├── redis密码
            └── jwt密钥
```

挂载后：

```text
/app/config/application.yml

/app/secret/mysql-password
```

应用启动读取。

------

# 十五、生产环境最佳实践

## ConfigMap

只存：

```text
配置
```

例如：

```yaml
host
port
topic
namespace
```

------

## Secret

只存：

```text
密码
Token
证书
Key
```

------

## 不要把密码放 ConfigMap

错误：

```yaml
data:
  mysql_password: 123456
```

------

正确：

```yaml
Secret
```

存储。

------

## 使用 Volume 挂载配置

优先：

```yaml
volumeMounts
```

而不是：

```yaml
env
```

原因：

```text
支持热更新
支持大文件
更符合应用配置习惯
```

------

# sercret类型也支持多文件

支持，而且在生产环境中非常常见。

很多人误以为 Secret 只能存：

```yaml
data:
  username: xxx
  password: xxx
```

实际上 Secret 和 ConfigMap 一样，都可以存储多个 Key，每个 Key 都可以映射成一个文件。

---

## 1 一个 Secret 存多个文件

例如：

```bash
tls.crt
tls.key
ca.crt
```

创建：

```bash
kubectl create secret generic nginx-secret \
  --from-file=tls.crt \
  --from-file=tls.key \
  --from-file=ca.crt
```

查看：

```bash
kubectl get secret nginx-secret -o yaml
```

结果：

```yaml
data:
  tls.crt: LS0tLS1CRUdJTi...
  tls.key: LS0tLS1CRUdJTi...
  ca.crt: LS0tLS1CRUdJTi...
```

实际上：

```text
Secret
 ├── tls.crt
 ├── tls.key
 └── ca.crt
```

---

## 2 挂载后会变成多个文件

Pod：

```yaml
volumes:
- name: certs
  secret:
    secretName: nginx-secret

containers:
- name: nginx
  volumeMounts:
  - name: certs
    mountPath: /etc/certs
```

容器内：

```text
/etc/certs
├── tls.crt
├── tls.key
└── ca.crt
```

每个 Key 对应一个文件。

---

## 3 YAML 方式定义多个文件

例如：

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
type: Opaque

stringData:
  db.conf: |
    host=10.0.0.1
    user=root

  redis.conf: |
    host=10.0.0.2
    password=123456
```

创建后：

```text
/app/secret
├── db.conf
└── redis.conf
```

---

## 4 Secret 与 ConfigMap 的文件挂载机制完全一样

ConfigMap：

```text
Key
 ↓
文件名
 ↓
文件内容
```

Secret：

```text
Key
 ↓
文件名
 ↓
文件内容
```

例如：

```yaml
data:
  application.yml: xxx
  logback.xml: xxx
```

挂载后：

```text
/application.yml
/logback.xml
```

Secret 也是同样逻辑。

---

## 5 还可以自定义文件名

假设 Secret：

```yaml
data:
  password: MTIzNDU2
```

默认挂载：

```text
/password
```

可以改成：

```yaml
volumes:
- name: secret-vol
  secret:
    secretName: mysql-secret
    items:
    - key: password
      path: mysql/password.txt
```

容器中：

```text
/mysql/password.txt
```

---

## 6 Secret 大文件限制

需要注意一个生产环境限制：

单个 Secret 最大：

```text
1 MiB
```

（约 1024 KB）

这是 Kubernetes API Server 的限制。

因此：

❌ 不适合

```text
大型证书库
大配置文件
几十MB文件
```

适合：

```text
密码
Token
TLS证书
SSH密钥
License
小型配置
```

---

## 7 最常见的生产场景

### 场景1：TLS证书

```text
Secret
 ├── tls.crt
 └── tls.key
```

挂载：

```text
/etc/nginx/certs
├── tls.crt
└── tls.key
```

Nginx直接读取。

---

### 场景2：Docker仓库认证

Secret：

```text
.dockerconfigjson
```

用于：

```yaml
imagePullSecrets:
- name: harbor-secret
```

拉取私有镜像。

---

### 场景3：SSH密钥

```text
Secret
 ├── id_rsa
 ├── id_rsa.pub
 └── known_hosts
```

挂载：

```text
/root/.ssh
```

GitLab Runner、Jenkins 经常这么干。

---

## 面试题

### Secret 能存多个文件吗？

能。

本质上 Secret 是：

```text
map[string]string
```

每个 Key 都可以映射成一个文件。

例如：

```yaml
data:
  tls.crt: xxx
  tls.key: xxx
  ca.crt: xxx
```

挂载后：

```text
/tls.crt
/tls.key
/ca.crt
```

因此从文件挂载角度来说：

```text
ConfigMap 能做的
Secret 基本都能做
```

二者最大的区别不是文件数量，而是：

```text
ConfigMap -> 普通配置
Secret -> 敏感数据
```

并且 Secret 挂载到容器时，Kubernetes 默认会把文件权限设置得更严格（通常为 0400/0444 一类权限），这是它与 ConfigMap 的另一个重要区别。

# 面试高频题

### ConfigMap 和 Secret 有什么区别？

答：

- ConfigMap存普通配置
- Secret存敏感数据
- 使用方式完全一致
- Secret默认Base64编码
- Secret建议开启etcd加密

------

### ConfigMap 更新后 Pod 会自动更新吗？

答：

- Volume挂载会同步更新
- Env方式不会更新
- Env需要重建Pod

------

### Secret 是加密的吗？

答：

默认不是加密。

只是：

```text
Base64 Encode
```

真正安全依赖：

```text
RBAC
ETCD Encryption
Network Policy
```

------

### Secret 为什么比 ConfigMap 安全？

答：

因为 Kubernetes 可以：

- RBAC控制访问
- etcd加密
- 审计日志
- Secret专门管理敏感数据

但默认情况下：

```text
Secret ≠ 加密存储
```

这是很多面试官最喜欢问的陷阱题。

