# DNF 超完整整合教程（一篇就够）


## 一、DNF 是什么（先搞清定位）

### 1️⃣ DNF 的定义

**DNF（Dandified Yum）** 是 **YUM 的下一代实现**：

- 兼容 yum 命令
- 依赖解析更强
- 速度更快
- 内存占用更低

👉 **CentOS 8+：yum = dnf 的软链接**

------

## 二、DNF 工作原理（理解级）

```text
dnf install nginx
↓
读取 /etc/dnf/dnf.conf
↓
加载 /etc/yum.repos.d/*.repo
↓
下载 repodata
↓
libsolv 解析依赖
↓
下载 rpm
↓
rpm 安装
↓
更新 rpmdb
```

RPM 数据库：

```bash
/var/lib/rpm/
```

------

## 三、DNF 配置体系（核心）

### 1️⃣ 主配置文件

```bash
/etc/dnf/dnf.conf
```

示例：

```ini
[main]
gpgcheck=1
installonly_limit=3
clean_requirements_on_remove=True
fastestmirror=True
max_parallel_downloads=10
logdir=/var/log
```

#### 关键参数

| 参数                         | 说明           |
| ---------------------------- | -------------- |
| clean_requirements_on_remove | 删除自动依赖   |
| fastestmirror                | 自动选最快镜像 |
| max_parallel_downloads       | 并发下载       |
| installonly_limit            | 内核保留数量   |

------

### 2️⃣ 仓库配置（通用）

路径：

```bash
/etc/yum.repos.d/*.repo
```

repo 示例：

```ini
[baseos]
name=BaseOS
baseurl=https://mirrors.aliyun.com/rockylinux/8/BaseOS/x86_64/os/
enabled=1
gpgcheck=1
```

------

## 四、DNF 基础命令（必会）

### 1️⃣ 查询

```bash
dnf repolist
dnf search redis
dnf info redis
dnf list installed
```

### 2️⃣ 安装 / 删除

```bash
dnf install nginx
dnf install -y vim net-tools
dnf remove nginx
dnf autoremove
```

### 3️⃣ 升级

```bash
dnf check-update
dnf upgrade nginx
dnf upgrade
```

------

## 五、DNF 缓存管理

```bash
dnf makecache
dnf clean all
dnf clean metadata
```

------

## 六、模块化（DNF 独有重点）

> **这是 DNF 与 YUM 最大的区别**

### 1️⃣ 什么是 Module

- 一个软件的 **多个版本集合**
- 如：php、nodejs、mysql

### 2️⃣ 查看模块

```bash
dnf module list
dnf module list php
```

### 3️⃣ 启用模块流

```bash
dnf module enable php:8.1
```

### 4️⃣ 安装模块

```bash
dnf install php
```

### 5️⃣ 切换版本（危险）

```bash
dnf module reset php
dnf module enable php:7.4
```

⚠️ 生产环境慎用切换

------

## 七、依赖 & 高级查询

```bash
dnf repoquery --requires nginx
dnf repoquery --whatprovides /usr/bin/ifconfig
dnf deplist nginx
```

------

## 八、历史记录 & 回滚

```bash
dnf history
dnf history info 5
dnf history undo 5
```

------

## 九、版本锁定（生产必用）

### 1️⃣ 安装插件

```bash
dnf install -y dnf-plugins-core
```

### 2️⃣ 锁定版本

```bash
dnf versionlock add nginx
dnf versionlock list
dnf versionlock delete nginx
```

------

## 十、离线安装（生产场景）

### 1️⃣ 下载 rpm + 依赖

```bash
dnf download --resolve nginx
```

### 2️⃣ 离线安装

```bash
dnf install *.rpm
```

------

## 十一、本地 DNF 仓库（企业级）

```bash
dnf install -y createrepo
createrepo /data/dnf
```

repo：

```ini
[local]
name=Local Repo
baseurl=file:///data/dnf
enabled=1
gpgcheck=0
```

------

## 十二、DNF vs YUM（对照速查）

| 项目     | yum      | dnf       |
| -------- | -------- | --------- |
| 依赖解析 | 一般     | 强        |
| 模块     | ❌        | ✅         |
| 并发下载 | ❌        | ✅         |
| 默认工具 | CentOS 7 | CentOS 8+ |

------

## 十三、生产环境最佳实践（重点）

### ✅ 推荐

- 使用模块化管理版本
- 启用 versionlock
- 使用私有仓库
- 控制升级窗口

### ❌ 禁止

- 随意切模块版本
- 直接 dnf upgrade
- 混合 repo

------

## 十四、适合你的实战场景（结合你背景）

你平时会遇到：

- K8s 节点（containerd、kubelet）
- 中间件（nginx / redis / mysql）
- 安全升级（CVE）

👉 **DNF + versionlock + 私有仓库** 是最稳方案