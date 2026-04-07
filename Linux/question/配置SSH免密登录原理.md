# 一、SSH 免密登录原理

SSH 免密登录本质上是 **基于非对称加密（公钥/私钥）认证机制**。

非对称加密有两个关键文件：

| 类型               | 位置            | 作用           |
| ------------------ | --------------- | -------------- |
| 私钥 (Private Key) | 客户端（B主机） | 用于身份认证   |
| 公钥 (Public Key)  | 服务器（A主机） | 用于验证客户端 |

认证流程如下：

1️⃣ 在 **B 主机生成密钥对**

```
id_rsa      私钥
id_rsa.pub  公钥
```

2️⃣ 将 **公钥放入 A 主机**

```
~/.ssh/authorized_keys
```

3️⃣ 当 B 主机 SSH 登录 A 主机时：

流程如下：

```
B主机                A主机
 |----请求登录------->|
 |                    |
 |<--随机字符串-------|
 |                    |
 |--私钥加密返回------>|
 |                    |
 |--用公钥验证-------->|
 |                    |
 |<--验证成功---------|
```

如果验证成功：

✅ **无需输入密码即可登录**

------

# 二、环境示例

假设：

| 主机 | IP           | 作用         |
| ---- | ------------ | ------------ |
| A    | 192.168.1.10 | 被登录服务器 |
| B    | 192.168.1.20 | 登录客户端   |

目标：

```
B 主机 ssh 登录 A 主机免密
```

------

# 三、实现步骤

## 1 在 B 主机生成 SSH 密钥

在 **B 主机执行**

```bash
ssh-keygen -t rsa -b 4096
```

系统提示：

```
Enter file in which to save the key (/root/.ssh/id_rsa):
```

直接回车。

然后：

```
Enter passphrase (empty for no passphrase):
```

直接回车。

再次回车确认。

生成结果：

```
~/.ssh/id_rsa
~/.ssh/id_rsa.pub
```

查看：

```bash
ls ~/.ssh
```

输出示例：

```
id_rsa
id_rsa.pub
```

说明生成成功。

------

# 四、将公钥复制到 A 主机

推荐使用：

## 方法1（推荐）

在 **B 主机执行**

```bash
ssh-copy-id root@192.168.1.10
```

输入一次密码：

```
root@192.168.1.10 password:
```

成功后系统提示：

```
Number of key(s) added: 1
```

------

## 方法2（手动方式）

如果没有 `ssh-copy-id`。

### 1 查看公钥

在 **B 主机**

```bash
cat ~/.ssh/id_rsa.pub
```

复制输出的整行内容。

示例：

```
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC....
```

------

### 2 登录 A 主机

```bash
ssh root@192.168.1.10
```

输入密码。

------

### 3 创建 ssh 目录

```bash
mkdir -p ~/.ssh
chmod 700 ~/.ssh
```

------

### 4 写入公钥

```bash
vi ~/.ssh/authorized_keys
```

把刚刚复制的 **公钥粘贴进去**。

------

### 5 修改权限

```bash
chmod 600 ~/.ssh/authorized_keys
```

------

# 五、测试免密登录

回到 **B 主机**

执行：

```bash
ssh root@192.168.1.10
```

如果成功：

```
Last login: Tue Mar 10
[root@server ~]#
```

不再需要输入密码。

说明：

✅ **免密登录成功**

------

# 六、关键文件结构

SSH key 文件结构：

```
~/.ssh/
├── id_rsa            私钥（客户端）
├── id_rsa.pub        公钥
├── authorized_keys   服务器信任公钥
```

权限必须是：

```
~/.ssh                700
authorized_keys       600
```

否则 SSH 会拒绝登录。

------

# 七、SSH 服务配置检查

如果免密不生效，需要检查：

```bash
vi /etc/ssh/sshd_config
```

确认以下配置：

```
PubkeyAuthentication yes
AuthorizedKeysFile .ssh/authorized_keys
```

修改后重启 ssh：

```bash
systemctl restart sshd
```

------

# 八、运维常用一键配置

一条命令生成密钥：

```bash
ssh-keygen -t rsa -N "" -f ~/.ssh/id_rsa
```

复制公钥：

```bash
ssh-copy-id root@192.168.1.10
```

测试：

```bash
ssh root@192.168.1.10
```

------

# 九、运维扩展（企业常用）

企业环境常见：

| 场景            | 说明                 |
| --------------- | -------------------- |
| Ansible 自动化  | 批量免密管理         |
| Kubernetes 集群 | master 免密登录 node |
| 批量服务器运维  | 跳板机统一登录       |
| CI/CD           | 自动发布             |

例如批量免密：

```
for ip in 192.168.1.{10..20}
do
ssh-copy-id root@$ip
done
```

------

