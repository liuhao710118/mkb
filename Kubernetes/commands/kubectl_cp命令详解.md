# kubectl cp 命令详解

在 Kubernetes 中：

```bash
kubectl cp
```

用于：

```text
Pod 与本地之间复制文件
```

类似于：

```bash
docker cp
scp
```

------

# 一、kubectl cp 能做什么

常见用途：

- 下载日志
- 上传配置
- 导出 dump 文件
- 导入脚本
- 临时调试
- 拷贝 JVM dump
- 拷贝 core 文件

------

# 二、基本语法

```bash
kubectl cp <源路径> <目标路径>
```

------

# 三、从 Pod 拷贝到本地

------

## 示例

```bash
kubectl cp nginx:/tmp/test.log ./test.log
```

含义：

```text
Pod:
  nginx

文件:
  /tmp/test.log

复制到:
  当前目录
```

------

# 四、从本地拷贝到 Pod

------

## 示例

```bash
kubectl cp ./app.conf nginx:/tmp/app.conf
```

含义：

```text
本地:
  ./app.conf

复制到 Pod:
  /tmp/app.conf
```

------

# 五、指定 namespace

------

## 使用 -n

```bash
kubectl cp ./a.txt prod/nginx:/tmp/a.txt
```

或者：

```bash
kubectl cp ./a.txt nginx:/tmp/a.txt -n prod
```

------

# 六、多容器 Pod

因为：

```text
一个 Pod 可以多个容器
```

需要：

```bash
-c
```

------

## 示例

```bash
kubectl cp ./a.txt nginx-pod:/tmp/ -c nginx
```

------

# 七、复制目录

------

## Pod → 本地

```bash
kubectl cp nginx:/var/log ./logs
```

------

## 本地 → Pod

```bash
kubectl cp ./html nginx:/usr/share/nginx/
```

------

# 八、kubectl cp 工作原理

底层实际上：

```text
tar + exec
```

------

# Kubernetes 内部执行：

```bash
tar cf -
```

再通过：

```bash
kubectl exec
```

传输。

------

# 九、为什么 cp 经常失败

因为：

```text
容器里没有 tar
```

------

# 常见错误

```text
tar: command not found
```

------

# 十、解决 tar 不存在

------

# Alpine

```bash
apk add tar
```

------

# Debian/Ubuntu

```bash
apt install tar
```

------

# Distroless 镜像

很多：

```text
没有 shell
没有 tar
```

此时：

```text
kubectl cp 无法使用
```

------

# 十一、Java 场景高频操作

你之前做过：

```text
SkyWalking
Kafka
Java 微服务
```

这些场景：

```bash
kubectl cp
```

非常常用。

------

# 十二、导出 JVM Heap Dump

------

# 1. 生成 dump

```bash
kubectl exec java-app -- jmap -dump:live,format=b,file=/tmp/heap.hprof 1
```

------

# 2. 下载 dump

```bash
kubectl cp java-app:/tmp/heap.hprof ./heap.hprof
```

------

# 十三、导出线程 dump

------

# 生成

```bash
kubectl exec java-app -- jstack 1 > thread.txt
```

或者：

```bash
kubectl exec java-app -- sh -c 'jstack 1 > /tmp/thread.txt'
```

------

# 下载

```bash
kubectl cp java-app:/tmp/thread.txt ./thread.txt
```

------

# 十四、导出日志

------

# 下载日志目录

```bash
kubectl cp app:/logs ./logs
```

------

# 下载单个日志

```bash
kubectl cp app:/logs/error.log ./error.log
```

------

# 十五、上传脚本到 Pod

------

## 示例

```bash
kubectl cp fix.sh app:/tmp/fix.sh
```

然后：

```bash
kubectl exec app -- chmod +x /tmp/fix.sh
```

执行：

```bash
kubectl exec app -- /tmp/fix.sh
```

------

# 十六、数据库场景

------

# MySQL 导出

```bash
kubectl exec mysql -- mysqldump -uroot -p123 test > test.sql
```

------

# 上传 SQL

```bash
kubectl cp ./test.sql mysql:/tmp/test.sql
```

------

# 导入 SQL

```bash
kubectl exec mysql -- mysql -uroot -p123 test < /tmp/test.sql
```

------

# 十七、kubectl cp 与 exec 配合

通常：

```text
cp + exec
```

一起使用。

------

# 示例流程

------

## 上传文件

```bash
kubectl cp app.conf app:/tmp/
```

------

## 进入容器

```bash
kubectl exec -it app -- sh
```

------

## 移动文件

```bash
mv /tmp/app.conf /app/config/
```

------

# 十八、权限问题

------

# 1. Permission denied

容器用户权限不足。

例如：

```text
runAsNonRoot
```

------

# 解决

上传到：

```text
/tmp
```

再移动。

------

# 2. Read-only file system

如果：

```yaml
readOnlyRootFilesystem: true
```

不能写入。

------

# 十九、大文件问题

------

# Heap dump

可能：

```text
几个 GB
```

------

# kubectl cp 缺点

- 慢
- 容易断
- API Server 压力大

------

# 更推荐

使用：

- PVC
- S3
- NFS
- 对象存储

------

# 二十、生产最佳实践

------

# 1. 少用 kubectl cp

原因：

```text
破坏容器不可变
```

------

# 2. 配置使用 ConfigMap

而不是：

```text
手动 cp
```

------

# 3. 日志使用集中式平台

不要：

```text
cp 日志文件
```

------

# 4. Dump 文件上传对象存储

避免：

```text
占满节点磁盘
```

------

# 二十一、cp 与 volume 区别

| 方式       | 用途     |
| ---------- | -------- |
| kubectl cp | 临时拷贝 |
| Volume     | 持久共享 |

------

# 二十二、cp 与 rsync

很多人用：

```bash
rsync
```

替代。

------

# 示例

```bash
rsync -av
```

优点：

- 增量同步
- 更快
- 断点续传

------

# Kubernetes 常见方案

```bash
kubectl exec + tar
```

或者：

```bash
kubectl port-forward
```

结合 rsync。

------

# 二十三、kubectl cp 常见错误

------

# 1. tar not found

最常见。

------

# 2. unexpected EOF

通常：

- 网络断开
- 文件过大
- API Server 超时

------

# 3. file exists

目标目录已存在。

------

# 4. container not found

多容器 Pod。

需要：

```bash
-c
```

------

# 二十四、kubectl cp 与 docker cp 区别

| kubectl cp     | docker cp           |
| -------------- | ------------------- |
| Pod            | Container           |
| 经过 apiserver | 直接 Docker         |
| 依赖 tar       | 依赖 Docker runtime |

------

# 二十五、完整实战案例

------

# Pod OOM 排查

------

## 1. 生成 heap dump

```bash
kubectl exec java-app -- jmap -dump:live,format=b,file=/tmp/heap.hprof 1
```

------

## 2. 下载文件

```bash
kubectl cp java-app:/tmp/heap.hprof ./heap.hprof
```

------

## 3. MAT 分析

使用：

```text
Eclipse MAT
```

查看：

- 大对象
- 内存泄漏
- ThreadLocal

------

# 二十六、生产高频命令

------

# 下载日志

```bash
kubectl cp app:/logs/error.log .
```

------

# 上传配置

```bash
kubectl cp app.conf app:/tmp/
```

------

# 下载 dump

```bash
kubectl cp java:/tmp/heap.hprof .
```

------

# 多 namespace

```bash
kubectl cp prod/app:/tmp/a.txt .
```

------

# 多容器

```bash
kubectl cp app:/tmp/a.txt . -c nginx
```

------

# 二十七、面试高频问题

------

# 1. kubectl cp 原理

底层：

```text
tar + exec
```

------

# 2. 为什么 cp 失败

通常：

```text
容器没有 tar
```

------

# 3. 为什么生产少用 cp

因为：

- 非声明式
- 不可审计
- 破坏镜像一致性

------

# 4. 配置为什么不用 cp

应该：

```text
ConfigMap
Secret
```

------

# 5. 大文件为什么不推荐 cp

因为：

```text
经过 apiserver
```

效率低。

------

# 二十八、建议继续学习

推荐继续：

1. kubectl describe
2. kubectl debug
3. kubectl top
4. ConfigMap
5. Secret
6. Volume/PVC
7. StatefulSet
8. Service
9. Ingress
10. Helm