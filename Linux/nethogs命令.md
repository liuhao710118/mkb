`nethogs` 是个**按进程统计实时网络流量**的神器，排查「**到底是谁在偷网速**」特别好用 😄

------

## 一、nethogs 是干嘛的

- 实时显示 **每个进程** 的网络上传 / 下载速率
- 支持 TCP / UDP
- 比 `iftop`（按连接）更适合**定位具体程序**

典型场景：

- 服务器网速慢了，想知道是哪个进程在跑流量
- 排查异常进程 / 木马 / 自动更新
- 运维、排错、性能分析

------

## 二、安装 nethogs

### Ubuntu / Debian

```bash
sudo apt update
sudo apt install nethogs
```

### CentOS / Rocky / Alma

```bash
sudo yum install nethogs
# 或
sudo dnf install nethogs
```

> ⚠️ 必须 **root 权限** 才能运行

------

## 三、最常用命令（核心）

### 1️⃣ 直接运行（默认网卡）

```bash
sudo nethogs
```

界面示例：

```
PID   USER   PROGRAM                DEV   SENT     RECEIVED
1234  root   nginx                   eth0  12 KB/s  120 KB/s
4567  user   python app.py           eth0  1 KB/s   3 KB/s
```

------

### 2️⃣ 指定网卡

```bash
sudo nethogs eth0
```

多网卡：

```bash
sudo nethogs eth0 wlan0
```

------

### 3️⃣ 非交互模式（适合脚本 / 日志）

```bash
sudo nethogs -t
```

输出示例：

```
1234 root nginx eth0 12.34 120.56
```

👉 **推荐用于监控脚本**

------

### 4️⃣ 设置刷新间隔（秒）

```bash
sudo nethogs -d 2
```

每 2 秒刷新一次

------

## 四、交互界面快捷键（非常好用）

运行 `sudo nethogs` 后：

| 快捷键 | 作用                         |
| ------ | ---------------------------- |
| `m`    | 切换显示模式（KB/s、KB、MB） |
| `r`    | 按接收流量排序               |
| `s`    | 按发送流量排序               |
| `q`    | 退出                         |

------

## 五、常见组合用法

### 🔹 看哪个进程在疯狂下行

```bash
sudo nethogs -d 1
# 再按 r
```

------

### 🔹 只抓一段时间

```bash
sudo timeout 30 nethogs -t
```

------

### 🔹 配合 grep 找某进程

```bash
sudo nethogs -t | grep python
```

------

## 六、nethogs vs iftop

| 工具    | 统计维度  |
| ------- | --------- |
| nethogs | **进程**  |
| iftop   | IP / 连接 |
| ss      | 连接状态  |
| tcpdump | 抓包      |

👉 **想找“谁”用流量 → nethogs**
👉 **想看“跟谁”通信 → iftop**

------

## 七、常见坑

❗ **不显示进程名**

- 可能是权限不足（必须 root）
- 内核版本过老

❗ **容器里看不准**

- 宿主机跑更准确
- K8s 建议用 `cilium monitor` / `kubectl top`

------

## 八、适合你的场景（运维向建议）

如果你是在：

- **服务器排流量异常**
- **Kafka / Elasticsearch / Agent 程序流量排查**

👉 可以：

```bash
sudo nethogs -t -d 1 > net.log
```

然后结合进程名、PID 精准定位