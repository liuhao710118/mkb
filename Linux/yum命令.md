

# YUM 超完整整合教程（CentOS 7 / RHEL 7）


## 一、YUM 是什么 & 工作原理

### 1️⃣ YUM 的定位

- 基于 **RPM** 的包管理工具
- **自动解决依赖**
- 从 **仓库（repo）** 下载 RPM 和元数据

### 2️⃣ YUM 的工作流程（必懂）

```text
yum install nginx
↓
读取 /etc/yum.repos.d/*.repo
↓
下载 repodata/repomd.xml
↓
解析依赖
↓
下载 rpm 包
↓
调用 rpm 安装
↓
更新 RPM 数据库 (/var/lib/rpm)
```

------

## 二、YUM 配置体系（核心）

### 1️⃣ 主配置文件

路径：

```bash
/etc/yum.conf
```

常用配置：

```ini
[main]
cachedir=/var/cache/yum/$basearch/$releasever
keepcache=0
logfile=/var/log/yum.log
gpgcheck=1
plugins=1
installonly_limit=3
```

| 参数              | 说明         |
| ----------------- | ------------ |
| cachedir          | 缓存目录     |
| keepcache         | 是否保留 rpm |
| logfile           | yum 日志     |
| gpgcheck          | 签名校验     |
| installonly_limit | 保留内核数量 |

------

### 2️⃣ 仓库配置（repo）

路径：

```bash
/etc/yum.repos.d/*.repo
```

标准 repo 示例：

```ini
[base]
name=CentOS-7-Base
baseurl=http://mirror.centos.org/centos/7/os/x86_64/
enabled=1
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
```

#### 常用变量

| 变量        | 含义     |
| ----------- | -------- |
| $releasever | 系统版本 |
| $basearch   | CPU 架构 |

------

## 三、YUM 基础命令（100% 必会）

### 1️⃣ 仓库 & 查询

```bash
yum repolist
yum repolist all
yum search nginx
yum info nginx
yum list installed
```

### 2️⃣ 安装 / 删除

```bash
yum install nginx
yum install -y vim net-tools
yum remove nginx
yum autoremove
```

### 3️⃣ 升级（生产慎用）

```bash
yum check-update
yum update nginx
yum update
```

------

## 四、缓存 & 常见维护

```bash
yum makecache
yum clean all
yum clean metadata
```

> yum 出问题，第一步就是 `yum clean all`

------

## 五、依赖与高级查询（运维常用）

### 1️⃣ 安装工具

```bash
yum install -y yum-utils
```

### 2️⃣ 依赖分析

```bash
yum deplist nginx
repoquery --requires nginx
repoquery --whatprovides /usr/bin/ifconfig
```

------

## 六、YUM 历史记录 & 回滚

```bash
yum history
yum history info 10
yum history undo 10
```

> 适合误操作回退（有限制）

------

## 七、第三方仓库（必用）

### 1️⃣ EPEL

```bash
yum install -y epel-release
```

### 2️⃣ 国内镜像（推荐）

```bash
curl -o /etc/yum.repos.d/CentOS-Base.repo \
https://mirrors.aliyun.com/repo/Centos-7.repo

yum clean all
yum makecache
```

------

## 八、本地 YUM 仓库（企业级）

### 1️⃣ 创建仓库

```bash
mkdir -p /data/yum/centos7
cp *.rpm /data/yum/centos7
```

### 2️⃣ 生成元数据

```bash
yum install -y createrepo
createrepo /data/yum/centos7
```

### 3️⃣ 配置 repo

```ini
[local]
name=Local Repo
baseurl=file:///data/yum/centos7
enabled=1
gpgcheck=0
```

------

## 九、离线安装（生产常见）

### 1️⃣ 下载 RPM + 依赖

```bash
yum install --downloadonly \
--downloaddir=/tmp/nginx nginx
```

### 2️⃣ 离线安装

```bash
rpm -Uvh *.rpm
```

------

## 十、YUM 与 systemd 联动

```bash
yum install nginx
systemctl start nginx
systemctl enable nginx
```

------

## 十一、常见故障速查

### ❌ repomd.xml 错误

```bash
yum clean all
yum makecache
```

### ❌ 依赖冲突

```bash
yum install 包名 --skip-broken
```

### ❌ DNS 问题

```bash
cat /etc/resolv.conf
```

------

## 十二、生产环境最佳实践（重点）

### ✅ 推荐

- 使用 **私有 YUM 仓库**
- 固定版本安装
- 升级前快照
- 变更有记录

### ❌ 禁止

- 直接 `yum update`
- 混用多个第三方 repo
- 关闭 gpgcheck

------

## 十三、YUM vs DNF

- CentOS 7：`yum`（底层已是 dnf）
- CentOS 8+：`dnf`
- 命令基本一致

