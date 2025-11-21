# **Linux `sftp` 命令教程**

## 一、`sftp` 是什么？

`SFTP`（SSH File Transfer Protocol）是一种 **通过 SSH 安全传输文件的协议**。
 它可以：

- 安全地上传/下载文件
- 浏览远程目录
- 删除、重命名文件
- 设置权限
- 支持批量操作

相比 `scp`，`sftp` 更适合交互式管理文件和目录。

------

# **二、基本语法**

```
sftp [选项] [user@]host
```

示例：

```
sftp root@192.168.1.100
```

------

# **三、连接远程服务器**

### 1. 交互式登录

```
sftp user@remote_host
```

登录后进入 `sftp>` 提示符：

```
sftp>
```

### 2. 通过私钥登录（免密码）

```
sftp -i ~/.ssh/id_rsa user@remote_host
```

------

# **四、常用交互命令**

| 命令                  | 说明             |
| --------------------- | ---------------- |
| `ls`                  | 列出远程目录内容 |
| `lls`                 | 列出本地目录内容 |
| `cd <dir>`            | 切换远程目录     |
| `lcd <dir>`           | 切换本地目录     |
| `pwd`                 | 显示远程当前目录 |
| `lpwd`                | 显示本地当前目录 |
| `mkdir <dir>`         | 创建远程目录     |
| `rmdir <dir>`         | 删除远程空目录   |
| `rm <file>`           | 删除远程文件     |
| `rename <old> <new>`  | 重命名远程文件   |
| `chmod <mode> <file>` | 修改远程文件权限 |

------

# **五、上传和下载文件**

### 1. 上传单个文件

```
put localfile.txt
```

### 2. 上传多个文件

```
mput *.txt
```

### 3. 下载单个文件

```
get remotefile.txt
```

### 4. 下载多个文件

```
mget *.log
```

### 5. 上传/下载目录（递归）

```
put -r localdir
get -r remotedir
```

------

# **六、批量操作和非交互模式**

### 1. 使用 `-b` 批处理文件

创建批处理文件 `commands.txt`：

```
lcd /home/user/local
cd /home/user/remote
put *.txt
get *.log
bye
```

执行：

```
sftp -b commands.txt user@remote_host
```

### 2. 非交互式单命令执行

```
sftp user@remote_host <<< $'put localfile.txt\nbye'
```

------

# **七、常用参数**

| 参数               | 说明                                                |
| ------------------ | --------------------------------------------------- |
| `-b file`          | 批处理模式，读取命令文件                            |
| `-i identity_file` | 指定私钥文件                                        |
| `-P port`          | 指定端口（注意大写 P）                              |
| `-o option`        | 传递 ssh 配置选项，如 `-o StrictHostKeyChecking=no` |
| `-v`               | 显示详细调试信息                                    |

------

# **八、实战示例**

------

### **1. 上传单个文件**

```
sftp user@192.168.1.100
sftp> put report.pdf
```

------

### **2. 上传整个目录**

```
sftp user@192.168.1.100
sftp> put -r project/
```

------

### **3. 下载日志文件**

```
sftp user@192.168.1.100
sftp> get /var/log/nginx/access.log
```

------

### **4. 批量上传本地 txt 文件**

```
sftp user@192.168.1.100
sftp> mput *.txt
```

------

### **5. 批量命令执行（非交互）**

创建 `batch.txt`：

```
lcd /home/user/files
cd /home/remote/files
put *.txt
bye
```

执行：

```
sftp -b batch.txt user@192.168.1.100
```

------

### **6. 指定端口和私钥登录**

```
sftp -i ~/.ssh/id_rsa -P 2222 user@remote_host
```

------

# **九、补充技巧**

- 使用 `lls` / `lcd` 在本地切换目录，配合 `put` / `mput` 上传
- 使用 `get -r` 下载整个目录非常方便
- 批处理模式 `-b` 可用于定时任务（cron）自动同步文件
- 配合 `ssh config` 可以简化命令：

```
Host server1
    HostName 192.168.1.100
    User user
    Port 2222
    IdentityFile ~/.ssh/id_rsa
```

然后只需：

```
sftp server1
```

------

# **十、Cheat Sheet（快速速查表）**

| 功能         | 命令                              |
| ------------ | --------------------------------- |
| 登录远程     | `sftp user@host`                  |
| 上传文件     | `put localfile`                   |
| 上传目录     | `put -r localdir`                 |
| 批量上传     | `mput *.txt`                      |
| 下载文件     | `get remotefile`                  |
| 下载目录     | `get -r remotedir`                |
| 切换远程目录 | `cd /path`                        |
| 切换本地目录 | `lcd /path`                       |
| 查看远程目录 | `ls`                              |
| 查看本地目录 | `lls`                             |
| 创建目录     | `mkdir dir`                       |
| 删除文件     | `rm file`                         |
| 修改权限     | `chmod 644 file`                  |
| 批处理模式   | `sftp -b batch.txt user@host`     |
| 指定端口     | `sftp -P 2222 user@host`          |
| 指定私钥     | `sftp -i ~/.ssh/id_rsa user@host` |

