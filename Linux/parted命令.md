`parted` 是 Linux 中 **磁盘分区管理** 的核心命令之一，主要用于
👉 **大于 2TB 磁盘**
👉 **GPT 分区表**
👉 **在线分区调整（比 fdisk 强）**

非常适合运维 / 云服务器 / 存储场景。

------

## 一、parted 是干什么的

- 创建 / 删除 / 查看 **磁盘分区**
- 支持 **MBR / GPT**
- 支持 **大磁盘（>2TB）**
- 可 **在线调整分区大小（部分场景）**

📌 常见替代关系：

- `fdisk`：传统，小盘
- **`parted`：推荐，大盘**

------

## 二、查看磁盘与分区

### 1️⃣ 查看所有磁盘

```bash
parted -l
```

示例：

```
Model: VMware Virtual disk (scsi)
Disk /dev/sdb: 2.2TB
Partition Table: gpt
Number  Start   End     Size    File system  Name  Flags
```

------

### 2️⃣ 查看单块磁盘

```bash
parted /dev/sdb print
```

------

## 三、交互式使用（最常见）

### 进入 parted

```bash
parted /dev/sdb
```

常用命令：

```
print        # 显示分区表
mklabel gpt # 创建 GPT 分区表（⚠会清盘）
mkpart      # 创建分区
rm 1        # 删除分区
quit        # 退出
```

------

## 四、创建 GPT 分区（实战）

### 示例：新磁盘 /dev/sdb，创建一个数据分区

```bash
parted /dev/sdb
mklabel gpt
mkpart primary 0% 100%
print
quit
```

然后：

```bash
mkfs.xfs /dev/sdb1
mount /dev/sdb1 /data
```

------

## 五、非交互式（脚本推荐）

```bash
parted -s /dev/sdb mklabel gpt
parted -s /dev/sdb mkpart primary 0% 100%
```

👉 自动化 / Ansible / Shell 非常常用

------

## 六、分区单位（避免对不齐）

```bash
parted /dev/sdb
unit GB
mkpart primary 0 500
```

常用单位：

- MB
- GB
- TB
- `%`（最推荐）

------

## 七、扩容磁盘（高频场景）

### 1️⃣ 云盘扩容后

```bash
lsblk
parted /dev/sdb print
```

### 2️⃣ 扩展分区到最大

```bash
parted /dev/sdb
resizepart 1 100%
quit
```

### 3️⃣ 扩展文件系统

- ext4

```bash
resize2fs /dev/sdb1
```

- xfs

```bash
xfs_growfs /data
```

------

## 八、parted vs fdisk

| 对比项   | parted | fdisk |
| -------- | ------ | ----- |
| GPT      | ✅      | ⚠️     |
| >2TB     | ✅      | ❌     |
| 在线调整 | ✅      | ❌     |
| 自动化   | ✅      | 一般  |

------

## 九、常见坑（一定要注意）

❗ **mklabel 会清空整个磁盘**
❗ xfs **不能缩容**
❗ resizepart 后一定要扩展文件系统
❗ 操作前确认磁盘名（lsblk）

------

## 十、最常用速查

```bash
parted -l
parted /dev/sdb print
parted -s /dev/sdb mklabel gpt
parted -s /dev/sdb mkpart primary 0% 100%
```

------

- 