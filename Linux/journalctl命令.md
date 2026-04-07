# 一、journalctl 是什么

`journalctl` 是 `systemd` 的日志查看工具，对应组件是：

👉 `systemd-journald`

它的特点：

- 集中管理所有日志（系统 + 服务）
- 替代传统 `/var/log/*`
- 支持按服务、时间、级别过滤

👉 一句话总结：

> **journalctl = Linux 日志查询神器**

------

# 二、日志来源（你要知道）

`journalctl` 能看到的日志包括：

- 内核日志（kernel）
- systemd 服务日志
- Docker / kubelet 日志
- 标准输出（stdout/stderr）

------

# 三、journalctl 核心命令（必须掌握）

------

## 1. 查看全部日志

```bash
journalctl
```

👉 不推荐直接用（太多）

------

## 2. 查看某个服务日志（最重要）

```bash
journalctl -u nginx
```

👉 实战中最常用

------

## 3. 实时查看日志（类似 tail -f）

```bash
journalctl -u nginx -f
```

👉 用于：

- 观察服务启动过程
- 实时排错

------

## 4. 查看最近 N 行

```bash
journalctl -n 100
```

------

## 5. 查看服务最近日志

```bash
journalctl -u nginx -n 50
```

------

## 6. 查看启动日志

```bash
journalctl -b
```

------

### 查看上一次启动

```bash
journalctl -b -1
```

👉 非常适合：

- 系统重启后排查问题

------

# 四、时间过滤（重点）

------

## 1. 指定时间范围

```bash
journalctl --since "2026-04-01 10:00:00"
journalctl --until "2026-04-01 12:00:00"
```

------

## 2. 相对时间

```bash
journalctl --since "1 hour ago"
journalctl --since "10 min ago"
```

------

## 3. 结合服务

```bash
journalctl -u kubelet --since "30 min ago"
```

------

# 五、日志级别过滤（排障神器）

------

## 查看错误日志

```bash
journalctl -p err
```

------

## 常见级别

| 级别    | 含义       |
| ------- | ---------- |
| emerg   | 系统不可用 |
| alert   | 紧急       |
| crit    | 严重       |
| err     | 错误       |
| warning | 警告       |
| info    | 信息       |
| debug   | 调试       |

------

## 示例

```bash
journalctl -u kubelet -p err
```

------

# 六、增强阅读体验（非常实用）

------

## 1. 显示完整日志（避免截断）

```bash
journalctl -u nginx -l
```

------

## 2. 不分页输出

```bash
journalctl --no-pager
```

------

## 3. 只显示最新日志

```bash
journalctl -e
```

------

## 4. 详细模式（调试常用）

```bash
journalctl -xe
```

👉 包含：

- 错误解释
- systemd 提示

------

# 七、按条件过滤（进阶）

------

## 1. 按进程过滤

```bash
journalctl _PID=1234
```

------

## 2. 按用户过滤

```bash
journalctl _UID=1000
```

------

## 3. 按命令过滤

```bash
journalctl /usr/bin/docker
```

------

## 4. 查看内核日志

```bash
journalctl -k
```

------

# 八、日志导出（你之前问过）

------

## 1. 输出到文件

```bash
journalctl -u kubelet > kubelet.log
```

------

## 2. 导出时间段日志

```bash
journalctl --since "1 hour ago" > log.txt
```

------

## 3. JSON格式（用于分析）

```bash
journalctl -u kubelet -o json
```

------

# 九、实战场景（重点）

------

## 场景 1：kubelet 启动失败

```bash
journalctl -u kubelet -xe
```

👉 看：

- 证书错误
- API Server 连接失败
- 权限问题

------

## 场景 2：Docker 拉镜像失败

```bash
journalctl -u docker -f
```

👉 关注：

- timeout
- proxy
- registry 错误

------

## 场景 3：系统启动异常

```bash
journalctl -b -p err
```

------

## 场景 4：网络问题

```bash
journalctl -u NetworkManager
```

------

## 场景 5：服务频繁重启

```bash
journalctl -u nginx
```

👉 查：

- crash
- 配置错误
- 端口冲突

------

# 十、排障标准流程（非常重要）

------

## 第一步：看服务

```bash
systemctl status kubelet
```

------

## 第二步：看日志

```bash
journalctl -u kubelet -xe
```

------

## 第三步：缩小范围

```bash
journalctl -u kubelet --since "10 min ago"
```

------

## 第四步：实时观察

```bash
journalctl -u kubelet -f
```

------

# 十一、常见坑

------

## 1. 日志不全

👉 原因：

```bash
/etc/systemd/journald.conf
```

关键参数：

```ini
SystemMaxUse=
```

------

## 2. 日志丢失（重启后）

👉 默认可能是内存日志

解决：

```ini
Storage=persistent
```

目录：

```bash
/var/log/journal/
```

------

## 3. journalctl 没有权限

```bash
sudo journalctl
```

或加入组：

```bash
systemd-journal
```

------

# 十二、核心记忆总结

------

## 必会命令

```bash
journalctl -u xxx
journalctl -f
journalctl -xe
journalctl -n 100
journalctl --since "1 hour ago"
journalctl -b
```

------

## 排障黄金组合

```bash
systemctl status xxx
journalctl -u xxx -xe
```

------

# 如果你想再进一层（强烈建议）

我可以给你一份：

👉 **“journalctl + Kubernetes 日志排障实战（生产级）”**

包括：

- kubelet 常见报错全集
- containerd / docker 日志分析
- API Server 连接问题定位

直接帮你把现在这些问题打通。