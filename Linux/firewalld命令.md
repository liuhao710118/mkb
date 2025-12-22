## 一、firewalld 是什么？

**firewalld** 是 Rocky Linux 默认的防火墙管理工具，基于 **zone（区域）** 和 **service（服务）** 的动态防火墙系统。

### firewalld 的特点

- 🔥 **动态生效**：大部分规则无需重启防火墙
- 📦 **基于区域（zone）管理**
- 🧠 **服务抽象（service）**，不用记端口号
- 🧩 支持 rich rule（复杂规则）

------

## 二、firewalld 的基本概念

### 1️⃣ Zone（区域）

Zone 表示网络信任级别，不同 zone 有不同的默认策略。

常见 zone：

| Zone     | 说明                 |
| -------- | -------------------- |
| public   | 默认区域，适合服务器 |
| trusted  | 全放行               |
| internal | 内部网络             |
| dmz      | 对外服务器           |
| drop     | 丢弃所有包           |
| block    | 拒绝并返回 ICMP      |

👉 **默认推荐用 `public`**

------

### 2️⃣ Service（服务）

Service 是一组端口 + 协议的集合，例如：

| Service | 实际端口 |
| ------- | -------- |
| ssh     | 22/tcp   |
| http    | 80/tcp   |
| https   | 443/tcp  |
| mysql   | 3306/tcp |

Service 文件位置：

```bash
/usr/lib/firewalld/services/
```

------

### 3️⃣ Runtime / Permanent

| 模式      | 说明                  |
| --------- | --------------------- |
| Runtime   | 临时生效，重启失效    |
| Permanent | 永久生效，需要 reload |

> 🔑 **生产环境一定要加 `--permanent`**

------

## 三、firewalld 安装与启动

### 1️⃣ 安装

```bash
dnf install -y firewalld
```

### 2️⃣ 启动并设置开机自启

```bash
systemctl enable --now firewalld
```

### 3️⃣ 查看状态

```bash
systemctl status firewalld
```

------

## 四、基础操作命令

### 1️⃣ 查看当前默认 zone

```bash
firewall-cmd --get-default-zone
```

### 2️⃣ 查看所有 zone

```bash
firewall-cmd --get-zones
```

### 3️⃣ 查看当前 zone 详细规则

```bash
firewall-cmd --zone=public --list-all
```

------

## 五、端口与服务管理（最常用）

### ✅ 放行 SSH（推荐用 service）

```bash
firewall-cmd --zone=public --add-service=ssh --permanent
firewall-cmd --reload
```

### ✅ 放行 HTTP / HTTPS

```bash
firewall-cmd --add-service=http --permanent
firewall-cmd --add-service=https --permanent
firewall-cmd --reload
```

------

### ✅ 放行指定端口

#### 单端口

```bash
firewall-cmd --add-port=8080/tcp --permanent
```

#### 端口范围

```bash
firewall-cmd --add-port=30000-31000/tcp --permanent
```

#### 生效

```bash
firewall-cmd --reload
```

------

### ❌ 删除端口 / 服务

```bash
firewall-cmd --remove-port=8080/tcp --permanent
firewall-cmd --remove-service=http --permanent
firewall-cmd --reload
```

------

## 六、绑定网卡到 Zone（很重要）

### 1️⃣ 查看网卡

```bash
nmcli device status
```

### 2️⃣ 查看当前网卡所属 zone

```bash
firewall-cmd --get-active-zones
```

### 3️⃣ 绑定网卡到 public

```bash
firewall-cmd --zone=public --change-interface=ens33 --permanent
firewall-cmd --reload
```

------

## 七、Rich Rule（高级规则）

### 示例 1：只允许某个 IP 访问 3306

```bash
firewall-cmd --permanent \
--add-rich-rule='rule family="ipv4" source address="192.168.1.100" port port="3306" protocol="tcp" accept'
firewall-cmd --reload
```

------

### 示例 2：禁止某个 IP 访问服务器

```bash
firewall-cmd --permanent \
--add-rich-rule='rule family="ipv4" source address="1.2.3.4" reject'
firewall-cmd --reload
```

------

### 示例 3：限制 SSH 访问来源

```bash
firewall-cmd --permanent \
--add-rich-rule='rule family="ipv4" source address="192.168.10.0/24" service name="ssh" accept'
firewall-cmd --reload
```

------

## 八、NAT / 端口转发

### 开启 masquerade（常用于网关）

```bash
firewall-cmd --zone=public --add-masquerade --permanent
firewall-cmd --reload
```

### 端口转发示例

```bash
firewall-cmd --permanent \
--add-forward-port=port=80:proto=tcp:toport=8080
firewall-cmd --reload
```

------

## 九、查看 & 调试

### 查看所有规则

```bash
firewall-cmd --list-all
```

### 查看 rich rules

```bash
firewall-cmd --list-rich-rules
```

### 查看服务定义

```bash
firewall-cmd --get-services
```

------

## 十、常见错误 & 注意事项

### ❗ 忘记 reload

```bash
firewall-cmd --reload
```

### ❗ SSH 被封导致无法登录

- 使用 **控制台 / VNC**
- 或临时关闭 firewalld：

```bash
systemctl stop firewalld
```

### ❗ firewalld vs iptables

- Rocky Linux 8+ 默认用 firewalld
- firewalld 底层仍是 nftables / iptables

------

## 十一、推荐的服务器最小规则示例

```bash
# SSH
firewall-cmd --add-service=ssh --permanent

# Web
firewall-cmd --add-service=http --permanent
firewall-cmd --add-service=https --permanent

# 应用端口
firewall-cmd --add-port=8080/tcp --permanent

firewall-cmd --reload
```

