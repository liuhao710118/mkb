# **Linux `lscpu` 命令教程文档**

`lscpu` 用于显示 **CPU 的架构、核心数、线程数、型号、缓存等详细信息**。
 是排查性能、规划服务器、分析硬件配置的重要命令之一。

------

# **一、基本语法**

```
lscpu [选项]
```

常用选项：

| 选项       | 含义                                    |
| ---------- | --------------------------------------- |
| **-e**     | 以表格形式列出 CPU 的每个逻辑处理器信息 |
| **-J**     | 以 JSON 格式输出数据                    |
| **-p**     | 以 CSV 格式输出（适合脚本处理）         |
| **--help** | 查看帮助                                |

------

# **二、基本输出说明**

执行最常用命令：

```
lscpu
```

输出示例（部分）：

```
Architecture:        x86_64
CPU op-mode(s):      32-bit, 64-bit
CPU(s):              8
Thread(s) per core:  2
Core(s) per socket:  4
Socket(s):           1
Vendor ID:           GenuineIntel
Model name:          Intel(R) Xeon(R) CPU E5-2678 v3 @ 2.50GHz
CPU MHz:             2500.000
L1d cache:           32K
L1i cache:           32K
L2 cache:            256K
L3 cache:            20M
```

下面是关键字段说明：

| 字段                   | 含义                                  |
| ---------------------- | ------------------------------------- |
| **Architecture**       | CPU 架构（x86_64/arm 等）             |
| **CPU(s)**             | 逻辑 CPU 数量（顶层核心数 * 线程数）  |
| **Thread(s) per core** | 每个物理核心的线程数（HT 超线程技术） |
| **Core(s) per socket** | 每个 CPU 插槽的物理核心数             |
| **Socket(s)**          | 实际安装的 CPU 颗数                   |
| **Vendor ID**          | 厂商（Intel/AMD）                     |
| **Model name**         | CPU 型号                              |
| **CPU MHz**            | 当前动态频率                          |
| **L1/L2/L3 cache**     | 各级 CPU 缓存大小                     |

------

# **三、常用命令示例**

------

## **1. 显示 CPU 全部信息（最常用）**

```
lscpu
```

------

## **2. 以表格方式展示所有逻辑 CPU（含 NUMA、核心 ID）**

```
lscpu -e
```

示例输出（部分）：

```
CPU NODE SOCKET CORE L1d L1i L2 L3
0     0      0    0   0   0  0  0
1     0      0    1   1   1  1  0
...
```

常用于：

- 查看核绑定情况（NUMA）
- 分析 CPU 分布
- 进行 CPU 调度优化

------

## **3. 以 JSON 输出（适合 API/脚本分析）**

```
lscpu -J
```

------

## **4. 查看 CPU 拓扑以 CSV 格式（脚本处理常用）**

```
lscpu -p
```

示例：

```
# CPU,Core,Socket,Node
0,0,0,0
1,1,0,0
...
```

------

# **四、实战运维场景**

------

## **1. 判断服务器是否是多核 + 超线程（HT）**

```
lscpu | grep -E "CPU\(s\)|Thread|Core"
```

示例结果：

```
CPU(s):                16
Thread(s) per core:    2
Core(s) per socket:    8
```

说明：8 核心 × 超线程（2）= 16 逻辑 CPU

------

## **2. 查看 CPU 型号信息**

```
lscpu | grep "Model name"
```

------

## **3. 判断是否为虚拟机（识别 KVM/Xen）**

```
lscpu | grep Hypervisor
```

输出示例：

```
Hypervisor vendor:     KVM
```

------

## **4. 查看每个 NUMA 节点的 CPU 分布**

```
lscpu -e | column -t
```

------

## **5. 统计 CPU 缓存

```
lscpu | grep cache
```

------

# **五、和其他 CPU 相关命令对比**

| 命令                  | 作用                                  |
| --------------------- | ------------------------------------- |
| **lscpu**             | 显示 CPU 详细拓扑和缓存信息           |
| **nproc**             | 显示逻辑 CPU 数                       |
| **cat /proc/cpuinfo** | 显示每个逻辑 CPU 的详细信息（最底层） |
| **top/htop**          | 查看实时 CPU 使用情况                 |

------

# **六、总结**

`lscpu` 是快速查看 CPU 结构与性能信息的核心命令，可获取：

- 架构
- 核心数/线程数
- CPU 型号
- NUMA 拓扑
- 多级缓存
- 多 CPU 插槽信息

最常用命令：

```
lscpu        # 显示所有信息
lscpu -e     # 以表格形式查看 CPU 拓扑
lscpu -J     # JSON 格式输出
lscpu -p     # CSV 输出，适合脚本
```

