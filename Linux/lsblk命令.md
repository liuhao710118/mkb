`lsblk` 是 Linux 中**查看块设备（磁盘、分区、挂载点）结构**的常用命令之一，直观、非常适合运维和排障场景。

------

## 一、lsblk 是做什么的

- 列出系统中的 **块设备**
- 显示 **磁盘 → 分区 → LVM / RAID → 挂载点** 的层级关系
- **不读取磁盘数据**，只读内核信息（安全、快速）

------

## 二、最常用的基本用法

```bash
lsblk
```

示例输出：

```
NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sda      8:0    0   500G  0 disk
├─sda1   8:1    0   200G  0 part /
├─sda2   8:2    0    50G  0 part [SWAP]
└─sda3   8:3    0   250G  0 part /data
```

字段含义：

- **NAME**：设备名
- **SIZE**：大小
- **TYPE**：类型（disk / part / lvm / rom）
- **MOUNTPOINT**：挂载点

------

## 三、常用参数（运维必会）

### 1️⃣ 查看文件系统类型

```bash
lsblk -f
```

输出示例：

```
NAME   FSTYPE LABEL UUID                                 MOUNTPOINT
sda
├─sda1 ext4         a1b2c3d4-...                         /
├─sda2 swap         e5f6g7h8-...                         [SWAP]
└─sda3 xfs          1234-5678                            /data
```

👉 **非常常用：判断 ext4 / xfs / swap**

------

### 2️⃣ 查看完整设备信息（推荐）

```bash
lsblk -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINT
```

------

### 3️⃣ 查看磁盘型号 / 序列号

```bash
lsblk -o NAME,SIZE,MODEL,SERIAL
```

👉 常用于 **云服务器 / 多盘识别**

------

### 4️⃣ 查看某一块磁盘

```bash
lsblk /dev/sda
```

------

### 5️⃣ 查看未挂载的磁盘或分区

```bash
lsblk -o NAME,SIZE,MOUNTPOINT | grep -v /
```

或：

```bash
lsblk -o NAME,SIZE,MOUNTPOINT | awk '$3==""'
```

👉 **新加磁盘排查必用**

------

## 四、lsblk + 其他命令配合

### 对比 df（已挂载）

```bash
df -h
lsblk
```

区别：

| 命令    | 作用           |
| ------- | -------------- |
| `lsblk` | 所有磁盘结构   |
| `df -h` | 已挂载文件系统 |

------

### 查看分区详细信息

```bash
lsblk -f
blkid
```

------

## 五、常见运维场景

### 🔹 新加一块磁盘，确认是否识别

```bash
lsblk
```

### 🔹 判断某目录挂在哪个盘

```bash
lsblk
df -h /data
```

### 🔹 LVM 场景

```bash
lsblk
sdb
└─vgdata-lvdata lvm 1T /data
```

------

## 六、常用速查表

| 需求           | 命令                       |
| -------------- | -------------------------- |
| 查看磁盘结构   | `lsblk`                    |
| 查看文件系统   | `lsblk -f`                 |
| 查看挂载点     | `lsblk -o NAME,MOUNTPOINT` |
| 查看磁盘型号   | `lsblk -o NAME,MODEL`      |
| 查看未挂载磁盘 | `lsblk                     |

------

