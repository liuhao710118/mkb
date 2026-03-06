`nethogs` 是一个**实时查看“哪个进程在占用网络带宽”**的神器，运维/排障时特别好用 🚦
它盯的是 **进程级**，不是接口级（这点和 `iftop` 不一样）。

------

## 一、nethogs 是干嘛的？

👉 **回答一个核心问题：**

> **到底是哪个进程在疯狂跑流量？**

比如：

- 谁在偷偷 `wget / curl`
- Java / Python / Docker / kubelet 哪个在吃带宽
- 夜里流量异常，到底是备份还是有人在拉包

------

## 二、安装

### CentOS / Rocky / Alma

```bash
yum install -y epel-release
yum install -y nethogs
```

### Ubuntu / Debian

```bash
apt update
apt install -y nethogs
```

⚠️ 必须 root 权限运行

------

## 三、最常用（直接上）

```bash
nethogs
```

界面里你会看到：

```
PID  USER  PROGRAM           DEV   SENT   RECEIVED
1234 root  java               eth0  120KB/s  5MB/s
5678 www   python3            eth0  20KB/s   1MB/s
```

------

## 四、指定网卡（强烈推荐）

```bash
nethogs eth0
```

多网卡：

```bash
nethogs eth0 eth1
```

👉 不指定时，默认监听所有网卡，信息会有点乱

------

## 五、常用参数

| 参数   | 作用                            |
| ------ | ------------------------------- |
| `-d 2` | 每 2 秒刷新一次                 |
| `-t`   | 文本模式（适合写日志 / 重定向） |
| `-v 3` | 显示更详细（debug）             |
| `-p`   | 不解析端口和主机名（更快）      |

示例：

```bash
nethogs -d 1 eth0
```

------

## 六、文本模式（写脚本 / 记录用）

```bash
nethogs -t eth0
```

输出类似：

```text
Refreshing:
PID USER PROGRAM DEV SENT RECEIVED
1234 root java eth0 1048576 5242880
```

👉 很适合：

- cron 定时采样
- grep / awk 统计
- 结合监控系统

------

## 七、常用快捷键（界面内）

| 键   | 功能                |
| ---- | ------------------- |
| `q`  | 退出                |
| `s`  | 按发送流量排序      |
| `r`  | 按接收流量排序      |
| `m`  | 切换 KB/s ↔ KB ↔ MB |
| `d`  | 切换刷新时间        |

------

## 八、典型运维排障场景

### 🔥 场景 1：服务器突然卡、出口跑满

```bash
nethogs eth0
```

→ 一眼看到是 `java` / `rsync` / `docker pull`

------

### 🔥 场景 2：Docker / K8s 节点流量异常

```bash
nethogs -p eth0
```

重点关注：

- containerd
- dockerd
- kubelet
- pause

------

### 🔥 场景 3：谁在偷偷下大文件

```bash
nethogs | grep wget
```

------

## 九、nethogs vs iftop

| 对比       | nethogs | iftop     |
| ---------- | ------- | --------- |
| 视角       | 进程    | IP / 连接 |
| 排查谁占网 | ⭐⭐⭐⭐⭐   | ⭐⭐        |
| 排查流向   | ⭐⭐      | ⭐⭐⭐⭐⭐     |
| 运维定位   | 首选    | 辅助      |

👉 **结论**

> 查“谁” → nethogs
> 查“连哪” → iftop

------

## 十、常见坑

### ❌ 看不到进程？

- 必须 root
- 某些内核老版本不支持

### ❌ 全是 unknown？

```bash
nethogs -p
```

### ❌ Docker 看不清？

→ 用 `nsenter` 或在宿主机直接看 `dockerd`

------

## 十一、一条命令定位网络元凶（推荐）

```bash
nethogs -d 1 eth0
```

10 秒内基本就能锁定“罪魁祸首”😈