### 📚 核心概念速览

在开始操作前，先了解几个基本概念会更有帮助：

| 概念       | 是什么                                                       | 简单比喻                                   |
| ---------- | ------------------------------------------------------------ | ------------------------------------------ |
| **分区**   | 将一块物理硬盘划分成多个独立的逻辑部分。                     | 给一个大柜子安装几个不同的抽屉。           |
| **格式化** | 在分区上创建文件系统（如XFS、EXT4），操作系统才能识别和存储数据。 | 给抽屉装上隔板、贴上标签，规定东西怎么放。 |
| **挂载**   | 将分区关联到Linux目录树中的一个具体目录（挂载点），从而通过目录访问磁盘空间。 | 把这个装好的抽屉放到你房间的某个固定位置。 |

### 🔍 第一步：识别新磁盘

首先，你需要确认系统已经识别了新添加的磁盘。

1. **打开终端**：使用 `Ctrl + Alt + T`快捷键或右键点击桌面选择“打开终端”。

2. **查看磁盘列表**：输入以下命令，它会列出所有磁盘及其分区信息，以树状图显示，非常直观。

   ```
   lsblk
   ```

3. **解读结果**：你会看到类似下面的输出。假设你有一块系统盘（通常是 `sda`或 `vda`），新加的磁盘可能就是 `sdb`（第二块SCSI硬盘）。关键是看它的名字（NAME）和大小（SIZE），并且没有子分区（没有`└─`开头的行）。

   ```
   NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
   sda           8:0    0   20G  0 disk 
   ├─sda1        8:1    0    1G  0 part /boot
   └─sda2        8:2    0   19G  0 part /
   sdb           8:16   0   50G  0 disk  <-- 这就是新磁盘，没有分区
   ```

### ⚙️ 第二步：对磁盘进行分区

确认磁盘后，我们来为它创建分区。这里推荐使用 `parted`工具，因为它对新手更友好，且直接支持GPT分区表（更现代的分区方案）。

1. **启动parted**：指定你要操作的新磁盘，例如 `/dev/sdb`。

   ```
   sudo parted /dev/sdb
   ```

2. **设置分区表类型**：在 `(parted)`提示符下，输入以下命令创建GPT分区表。

   ```
   mklabel gpt
   ```

   如果系统提示确认，输入 `Yes`。

3. **创建分区**：接下来，创建一个占用整个磁盘空间的主分区。

   ```
   mkpart primary 1MiB 100%
   ```

   - `primary`表示主分区。
   - `1MiB 100%`表示分区从磁盘开头1MB后开始（为了对齐，性能更好），一直用到磁盘末尾。

4. **检查与退出**：输入 `print`可以查看刚创建的分区信息。确认无误后，输入 `quit`退出parted。

   ```
   (parted) print
   Model: ATA VBOX HARDDISK (scsi)
   Disk /dev/sdb: 53.7GB
   Sector size (logical/physical): 512B/512B
   Partition Table: gpt
   Disk Flags: 
   Number  Start   End     Size    File system  Name     Flags
    1      1049kB  53.7GB  53.7GB               primary
   ```

### 📝 第三步：格式化分区

现在分区已经创建好（比如 `/dev/sdb1`），需要把它格式化成Linux能使用的文件系统。Rocky Linux默认使用XFS文件系统，性能很好。

使用以下命令进行格式化（请务必将 `/dev/sdb1`替换成你的实际分区名）：

```
sudo mkfs.xfs /dev/sdb1
```

如果系统提示确认，输入 `y`。看到类似下面的输出，说明格式化成功了。

```
meta-data=/dev/sdb1              isize=512    agcount=4, agsize=3276800 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=1, rmapbt=0
         =                       reflink=1
data     =                       bsize=4096   blocks=13107200, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0, ftype=1
log      =internal log           bsize=4096   blocks=6400, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
```

### 📌 第四步：挂载分区

格式化后，就可以将分区“挂载”到一个目录上了。

1. **创建挂载点**：挂载点就是一个空目录。习惯上，在 `/mnt`或根目录 `/`下创建，例如创建一个 `/data`目录。

   ```
   sudo mkdir /data
   ```

2. **临时挂载**：使用 `mount`命令进行挂载。这样挂载后，重启系统就会失效，适合测试。

   ```
   sudo mount /dev/sdb1 /data
   ```

3. **验证挂载**：再次使用 `lsblk`或 `df -h`命令查看。现在你应该能看到 `/dev/sdb1`的挂载点（MOUNTPOINT）是 `/data`。

   ```
   df -h /data
   ```

### 🔄 第五步：配置开机自动挂载

为了让分区每次开机都自动挂载，需要修改 `/etc/fstab`文件。**这是非常关键的一步，请小心操作。**

1. **获取分区的唯一标识（UUID）**：使用设备名（如 `/dev/sdb1`）可能会变，而UUID是唯一的，更安全。用以下命令查看分区的UUID。

   ```
   sudo blkid /dev/sdb1
   ```

   记下输出的 `UUID`值，看起来像一长串数字和字母的组合。

2. **备份并编辑fstab文件**：先备份总是个好习惯。

   ```
   sudo cp /etc/fstab /etc/fstab.bak
   ```

   然后使用文本编辑器（如 `nano`或 `vi`）打开文件：

   ```
   sudo nano /etc/fstab
   ```

3. **添加挂载信息**：在文件末尾新起一行，添加如下内容（将 `你的UUID`替换为上一步命令查到的实际值）。

   ```
   UUID=你的UUID /data xfs defaults 0 0
   ```

   - 第一项：分区的UUID。
   - 第二项：挂载点（`/data`）。
   - 第三项：文件系统类型（`xfs`）。
   - 第四项：挂载参数（`defaults`）。
   - 第五、六项：备份和自检设置（`0 0`表示不需要）。

4. **测试并生效**：保存并关闭文件后（在nano中按 `Ctrl+X`，然后按 `Y`确认，再按回车），**务必**执行以下命令测试配置是否正确。如果没有报错，说明配置成功。

   ```
   sudo mount -a
   ```

   最后，你可以重启系统验证一下是否成功自动挂载。

### 💎 总结与提醒

整个流程可以概括为：**识别磁盘 → 创建分区表并分区 → 格式化文件系统 → 挂载并使用 → 设置自动挂载**。

作为新手，请特别注意：

- **谨慎操作**：尤其在修改 `fstab`文件时，错误的编辑可能导致系统无法启动。务必先备份并仔细核对UUID。
- **使用UUID**：在 `fstab`中使用UUID而非设备名（如 `/dev/sdb1`）更可靠，因为设备名可能在添加或移除硬盘后发生变化。

希望这份详细的指南能帮助你顺利完成磁盘分区与挂载！如果在这个过程中遇到任何问题，或者想了解更高级的LVM（逻辑卷管理）知识，随时可以再问我。