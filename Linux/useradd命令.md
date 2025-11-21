# # **Linux `useradd` 命令教程**

## 一、`useradd` 是什么？

`useradd` 是 Linux 中用于 **创建新用户** 的命令，可以同时创建：

- 用户账号
- 用户家目录
- 用户主组
- 用户附加组
- 用户使用的 shell
- 用户 UID/GID
- 初始化配置文件（取自 /etc/skel）

创建用户后，可以用 `passwd` 设置密码。

------

# **二、基本语法**

```
useradd [选项] 用户名
```

例子：

```
useradd testuser
```

------

# **三、常用选项**

| 参数            | 说明                          |
| --------------- | ----------------------------- |
| `-d <目录>`     | 指定家目录                    |
| `-m`            | 创建家目录（如果不存在）      |
| `-u <UID>`      | 指定用户 ID                   |
| `-g <组>`       | 指定主组                      |
| `-G <组列表>`   | 指定附加组（逗号分隔）        |
| `-s <shell>`    | 指定 shell（如 /bin/bash）    |
| `-c <注释>`     | 用户描述（写入 /etc/passwd）  |
| `-e YYYY-MM-DD` | 设置账户过期时间              |
| `-M`            | 不创建家目录                  |
| `-r`            | 创建系统用户（UID 小于 1000） |

------

# **四、最常用示例**

## 1. 创建一个普通用户（带家目录）

```
useradd -m testuser
```

默认家目录：

```
/home/testuser
```

------

## 2. 创建用户并设置密码

```
useradd -m bob
passwd bob
```

------

## 3. 指定家目录

```
useradd -m -d /data/user01 user01
```

------

## 4. 指定使用的登录 shell（常见：bash）

```
useradd -m -s /bin/bash jack
```

------

## 5. 指定主组

```
useradd -g developers tom
```

------

## 6. 指定多个附加组

```
useradd -G wheel,developers,qa alice
```

查看组：

```
id alice
```

------

## 7. 指定 UID

```
useradd -u 1500 newuser
```

------

## 8. 不创建家目录（系统服务账号常用）

```
useradd -M serviceuser
```

------

## 9. 创建系统用户（不用于登录）

```
useradd -r nginx
```

系统用户特性：

- UID < 1000
- 无登录 shell 或 shell 被设置为 /sbin/nologin

------

## 10. 设置账号过期时间

```
useradd -e 2025-12-31 tempuser
```

查看过期时间：

```
chage -l tempuser
```

------

# **五、查看用户信息**

## 1. 从 /etc/passwd 查看

```
grep testuser /etc/passwd
```

格式：

```
用户名:x:UID:GID:注释:家目录:Shell
```

------

## 2. 查看用户组信息

```
id testuser
groups testuser
```

------

# **六、删除用户（补充）**

删除用户但保留家目录：

```
userdel username
```

删除用户及其家目录：

```
userdel -r username
```

------

# **七、实战案例**

------

## **1. 创建一个开发用户（带家目录、指定group、指定shell）**

```
useradd -m -g dev -G sudo -s /bin/bash dev01
passwd dev01
```

------

## **2. 创建一个只用于运行程序的系统用户（无家目录、无登录能力）**

```
useradd -r -M -s /sbin/nologin appuser
```

------

## **3. 批量创建多个用户**

users.txt 内容：

```
user01
user02
user03
```

脚本：

```
for u in $(cat users.txt); do
    useradd -m $u
    echo "123456" | passwd --stdin $u
done
```

------

# **八、Cheat Sheet（速查表）**

| 功能         | 命令                                |
| ------------ | ----------------------------------- |
| 创建用户     | `useradd username`                  |
| 创建带家目录 | `useradd -m username`               |
| 设置密码     | `passwd username`                   |
| 指定家目录   | `useradd -m -d /data/user username` |
| 指定 shell   | `useradd -s /bin/bash username`     |
| 指定主组     | `useradd -g group username`         |
| 指定附加组   | `useradd -G g1,g2 username`         |
| 不创建家目录 | `useradd -M username`               |
| 创建系统用户 | `useradd -r username`               |
| 账户过期     | `useradd -e YYYY-MM-DD username`    |
| 删除用户     | `userdel -r username`               |

