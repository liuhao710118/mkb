# **Linux `scp` 命令教程**

## 一、`scp` 是什么？

`scp`（Secure Copy）是基于 **SSH** 的 **安全文件拷贝工具**，用于在本地与远程主机之间安全传输文件或目录。

特点：

- 支持加密传输（SSH）
- 上传/下载文件和目录
- 可指定端口
- 可批量操作
- 常用作自动化脚本传输文件

> 注意：`scp` 适合 **单次传输**，如果需要递归下载整个站点或进行交互式操作，推荐 `sftp`。

------

## 二、基本语法

```
scp [选项] 来源目标 目标路径
```

来源和目标可以是：

- 本地文件：`/path/to/file`
- 远程文件：`user@host:/path/to/file`

------

## 三、常用操作示例

### 1. 从本地上传文件到远程主机

```
scp /home/user/file.txt user@192.168.1.100:/home/user/
```

说明：

- `/home/user/file.txt`：本地文件
- `user@192.168.1.100`：远程用户名和 IP
- `/home/user/`：远程路径

------

### 2. 从远程下载文件到本地

```
scp user@192.168.1.100:/home/user/file.txt /home/localuser/
```

------

### 3. 上传整个目录（递归）

```
scp -r /home/user/project user@192.168.1.100:/home/user/
```

`-r`：递归复制目录及文件

------

### 4. 指定 SSH 端口

```
scp -P 2222 file.txt user@192.168.1.100:/home/user/
```

> 注意：`-P` 大写，表示端口；小写 `-p` 表示保留文件时间和权限。

------

### 5. 保留文件权限和时间

```
scp -p file.txt user@host:/path
```

------

### 6. 批量上传多个文件

```
scp file1.txt file2.txt user@host:/path/
```

------

### 7. 使用私钥登录（免密码）

```
scp -i ~/.ssh/id_rsa file.txt user@host:/path
```

------

### 8. 显示传输进度

```
scp -v file.txt user@host:/path
```

------

## 四、常用参数速查表

| 参数               | 说明                       |
| ------------------ | -------------------------- |
| `-r`               | 递归复制目录               |
| `-P port`          | 指定远程端口（注意大写 P） |
| `-p`               | 保留文件权限、时间等       |
| `-i identity_file` | 使用私钥文件               |
| `-v`               | 显示详细传输信息           |
| `-q`               | 静默模式，不显示进度       |
| `-C`               | 压缩传输，提高速度         |

------

## 五、实战场景示例

### 1. 上传日志文件到远程服务器

```
scp /var/log/nginx/access.log root@192.168.1.100:/backup/logs/
```

------

### 2. 下载远程服务器的备份文件

```
scp root@192.168.1.100:/backup/db.sql /home/user/backups/
```

------

### 3. 递归备份整个目录

```
scp -r /home/user/project root@192.168.1.100:/backup/project
```

------

### 4. 使用非默认端口传输文件

```
scp -P 2222 file.tar.gz root@192.168.1.100:/backup/
```

------

### 5. 使用私钥免密码传输

```
scp -i ~/.ssh/id_rsa -P 2222 /home/user/file.zip root@192.168.1.100:/backup/
```

------

## 六、`scp` 与 `sftp` 对比

| 功能          | scp          | sftp                   |
| ------------- | ------------ | ---------------------- |
| 文件上传/下载 | ✔            | ✔                      |
| 目录递归      | ✔（-r）      | ✔（-r）                |
| 交互式浏览    | ✖            | ✔                      |
| 批处理        | ✖            | ✔（-b）                |
| 断点续传      | ✖            | ✔（sftp -C）           |
| 推荐场景      | 单次安全传输 | 交互式或自动化批量管理 |

------

## 七、Cheat Sheet（速查表）

| 功能     | 命令                                        |
| -------- | ------------------------------------------- |
| 上传文件 | `scp file.txt user@host:/path`              |
| 下载文件 | `scp user@host:/path/file.txt ./`           |
| 上传目录 | `scp -r dir/ user@host:/path`               |
| 保留权限 | `scp -p file user@host:/path`               |
| 指定端口 | `scp -P 2222 file user@host:/path`          |
| 使用私钥 | `scp -i ~/.ssh/id_rsa file user@host:/path` |
| 显示进度 | `scp -v file user@host:/path`               |
| 批量上传 | `scp file1 file2 user@host:/path`           |
| 压缩传输 | `scp -C file user@host:/path`               |

