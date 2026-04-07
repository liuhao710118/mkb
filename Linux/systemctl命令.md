

# 一、systemctl 是什么

`systemctl` 是 `systemd` 的管理工具，用来：

👉 **控制系统中的各种服务（service）和资源**

你可以把它理解为：

> 👉 Linux 的“服务管理总开关”

------

# 二、Unit（核心概念）

在 `systemd` 中，一切都叫 **Unit**

常见类型：

| 类型   | 后缀       | 示例              |
| ------ | ---------- | ----------------- |
| 服务   | `.service` | nginx.service     |
| 套接字 | `.socket`  | docker.socket     |
| 目标   | `.target`  | multi-user.target |
| 定时器 | `.timer`   | backup.timer      |

👉 实际使用中：

```bash
systemctl start nginx
```

等价于：

```bash
systemctl start nginx.service
```

------

# 三、systemctl 核心命令（必须掌握）

------

## 1. 查看服务状态（最常用）

```bash
systemctl status nginx
```

输出重点：

- `active (running)` → 正常运行
- `inactive` → 未运行
- `failed` → 启动失败 ❗

👉 示例判断：

| 状态             | 含义       |
| ---------------- | ---------- |
| active (running) | 正常       |
| active (exited)  | 执行完退出 |
| failed           | 出问题     |

------

## 2. 启动 / 停止 / 重启

```bash
systemctl start nginx
systemctl stop nginx
systemctl restart nginx
systemctl reload nginx
```

### 区别

| 命令    | 说明               |
| ------- | ------------------ |
| start   | 启动               |
| stop    | 停止               |
| restart | 停止+启动          |
| reload  | 重载配置（不中断） |

👉 举例：

```bash
systemctl reload nginx   # 热加载配置
```

------

## 3. 开机自启（重点）

```bash
systemctl enable nginx
systemctl disable nginx
```

### 原理

👉 实际是创建软链接：

```
/etc/systemd/system/multi-user.target.wants/
```

------

### 查看是否开机启动

```bash
systemctl is-enabled nginx
```

输出：

- enabled
- disabled

------

## 4. 查看所有服务

### 正在运行的服务

```bash
systemctl list-units --type=service
```

------

### 所有服务（包括未运行）

```bash
systemctl list-unit-files
```

------

## 5. 查看失败的服务（排障必备）

```bash
systemctl --failed
```

👉 这个命令非常关键！

------

## 6. 强制终止服务

```bash
systemctl kill nginx
```

------

## 7. 重载 systemd（非常重要）

```bash
systemctl daemon-reload
```

👉 场景：

- 修改了 `.service` 文件
- 新增服务

------

## 8. 重新执行 systemd（极少用）

```bash
systemctl daemon-reexec
```

------

# 四、进阶命令（建议掌握）

------

## 1. 查看服务依赖

```bash
systemctl list-dependencies nginx
```

------

## 2. 查看服务配置

```bash
systemctl cat nginx
```

------

## 3. 编辑服务（推荐方式）

```bash
systemctl edit nginx
```

👉 会创建：

```
/etc/systemd/system/nginx.service.d/override.conf
```

------

## 4. 查看服务属性

```bash
systemctl show nginx
```

------

## 5. 检查服务是否运行

```bash
systemctl is-active nginx
```

返回：

- active
- inactive

------

# 五、服务文件（.service）详解

路径：

```
/usr/lib/systemd/system/
/etc/systemd/system/
```

------

## 示例

```ini
[Unit]
Description=My Service
After=network.target

[Service]
ExecStart=/usr/bin/python3 /app/app.py
Restart=always
User=root

[Install]
WantedBy=multi-user.target
```

------

## 关键字段解释

### [Unit]

| 参数        | 说明     |
| ----------- | -------- |
| Description | 描述     |
| After       | 启动顺序 |

------

### [Service]

| 参数      | 说明     |
| --------- | -------- |
| ExecStart | 启动命令 |
| Restart   | 重启策略 |
| User      | 运行用户 |

------

### [Install]

| 参数     | 说明     |
| -------- | -------- |
| WantedBy | 绑定目标 |

------

# 六、实战案例（重点）

------

## 案例 1：启动失败排查

```bash
systemctl status kubelet
```

👉 如果失败：

```bash
journalctl -u kubelet -xe
```

------

## 案例 2：创建一个自定义服务

### 1. 创建文件

```bash
vi /etc/systemd/system/myapp.service
```

内容：

```ini
[Unit]
Description=My App
After=network.target

[Service]
ExecStart=/usr/bin/python3 /app/app.py
Restart=always

[Install]
WantedBy=multi-user.target
```

------

### 2. 生效

```bash
systemctl daemon-reload
systemctl enable myapp
systemctl start myapp
```

------

## 案例 3：Docker 开机自启

```bash
systemctl enable docker
systemctl start docker
```

------

## 案例 4：批量重启（K8s 常见）

```bash
systemctl restart kubelet
systemctl restart containerd
```

------

# 七、常见错误 & 解决方法

------

## 1. 修改 service 不生效

```bash
systemctl daemon-reload
```

------

## 2. 服务启动失败

```bash
systemctl status xxx
```

常见原因：

- 路径错误
- 权限问题
- 端口占用

------

## 3. enable 了但没启动

👉 enable ≠ start

```bash
systemctl start xxx
```

------

## 4. 服务不停重启

看：

```bash
systemctl status xxx
```

关注：

```
Restart=always
```

------

# 八、生产环境常用组合（非常实用）

------

## 1. 一键排查服务

```bash
systemctl status nginx
journalctl -u nginx -xe
```

------

## 2. 查看最近日志 + 状态

```bash
systemctl status nginx -l
```

------

## 3. 实时观察服务

```bash
watch -n 1 systemctl status nginx
```

------

## 4. 查端口占用导致启动失败

```bash
ss -lntp | grep 80
```

------

# 九、核心记忆总结

------

## 必会命令

```bash
systemctl start/stop/restart
systemctl enable/disable
systemctl status
systemctl --failed
systemctl daemon-reload
```

------

## 排障黄金组合

```bash
systemctl status xxx
journalctl -u xxx -xe
```

------

# 十、给你的进阶建议（结合你当前环境）

你现在在搞：

- Kubernetes
- Docker
- 内网环境

👉 强烈建议你下一步学：

1. systemd 管理 kubelet
2. systemd 管理 containerd
3. systemd 限制资源（CPU/内存）
4. systemd timer（替代 crontab）

------

如果你愿意，我可以给你一份：

👉 **“systemctl + Kubernetes 故障排查实战手册（生产级）”**

直接覆盖你现在遇到的那些问题（kubelet 启动失败、镜像拉取失败等）。