# 📘 Linux `ln` 命令使用文档

## 一、命令简介

`ln` 用于在文件之间创建链接。  
它可以创建两种类型的链接：

- **硬链接（Hard Link）**：多个文件名指向同一个物理数据块。
- **符号链接（Symbolic Link，软链接）**：类似 Windows 的快捷方式，指向另一个文件路径。

默认情况下，`ln` 创建的是 **硬链接**。  
若要创建 **符号链接**，需要使用选项 `-s` 或 `--symbolic`。

---

## 二、命令语法

```bash
ln [OPTION]... [-T] TARGET LINK_NAME
ln [OPTION]... TARGET
ln [OPTION]... TARGET... DIRECTORY
ln [OPTION]... -t DIRECTORY TARGET...
```

### 各语法形式说明

| 形式                        | 说明                                           |
| --------------------------- | ---------------------------------------------- |
| `ln TARGET LINK_NAME`       | 创建一个名为 `LINK_NAME` 的链接，指向 `TARGET` |
| `ln TARGET`                 | 在当前目录下创建一个与 `TARGET` 同名的链接     |
| `ln TARGET... DIRECTORY`    | 在指定的目录中为多个目标文件创建链接           |
| `ln -t DIRECTORY TARGET...` | 指定目标目录并批量创建链接                     |

---

## 三、常用选项说明

| 选项                               | 说明                                         |
| ---------------------------------- | -------------------------------------------- |
| `-s, --symbolic`                   | 创建**符号链接（软链接）**                   |
| `-f, --force`                      | 若目标已存在，**强制覆盖**                   |
| `-i, --interactive`                | 若目标已存在，**提示是否覆盖**               |
| `-b, --backup`                     | 若目标已存在，先**备份**旧文件               |
| `--backup=CONTROL`                 | 控制备份行为，见下方说明                     |
| `-r, --relative`                   | 创建**相对路径**的符号链接                   |
| `-v, --verbose`                    | 显示创建的链接信息                           |
| `-L, --logical`                    | 若目标是符号链接，则链接指向**实际文件**     |
| `-P, --physical`                   | 若目标是符号链接，则链接指向**符号链接本身** |
| `-n, --no-dereference`             | 若链接名为指向目录的符号链接，则视为普通文件 |
| `-t, --target-directory=DIRECTORY` | 指定目标目录创建链接                         |
| `-T, --no-target-directory`        | 总是将 `LINK_NAME` 视为普通文件，而非目录    |
| `--help`                           | 显示帮助信息并退出                           |
| `--version`                        | 显示版本信息并退出                           |

---

## 四、备份选项说明

若启用 `--backup` 或 `-b`，`ln` 会在覆盖文件前自动备份。  
备份文件的默认后缀为 `~`，可使用 `--suffix=SUFFIX` 修改。

| 控制值            | 含义                                       |
| ----------------- | ------------------------------------------ |
| `none`, `off`     | 不创建备份                                 |
| `numbered`, `t`   | 使用编号备份（如 `file.~1~`, `file.~2~`）  |
| `existing`, `nil` | 若已有编号备份则继续编号，否则使用简单备份 |
| `simple`, `never` | 始终使用简单备份（如 `file~`）             |

---

## 五、使用示例

### 1️⃣ 创建硬链接

```bash
ln source.txt link.txt
```

说明：  
`link.txt` 是 `source.txt` 的硬链接，删除任意一个文件不会影响另一个。

---

### 2️⃣ 创建符号（软）链接

```bash
ln -s /usr/local/nginx/nginx.conf /etc/nginx.conf
```

说明：  
创建一个指向 `nginx.conf` 的软链接，常用于简化配置文件路径。

---

### 3️⃣ 批量在目录中创建链接

```bash
ln -s *.log /backup/logs/
```

说明：  
为当前目录下所有 `.log` 文件在 `/backup/logs/` 中创建软链接。

---

### 4️⃣ 使用 `-t` 指定目标目录

```bash
ln -s -t /usr/local/bin myscript.sh
```

说明：  
将脚本 `myscript.sh` 的软链接放入 `/usr/local/bin`，方便全局执行。

---

### 5️⃣ 使用 `-r` 创建相对路径链接

```bash
ln -s -r /var/www/html/index.html /home/user/index.html
```

说明：  
创建相对路径符号链接，便于移动整个目录时保持链接有效。

---

### 6️⃣ 覆盖已存在文件

```bash
ln -sf /data/new.conf /etc/myapp.conf
```

说明：  
若 `/etc/myapp.conf` 已存在，则强制覆盖。

---

### 7️⃣ 创建并显示操作详情

```bash
ln -sv /opt/tool /usr/local/bin/tool
```

输出示例：

```
'/usr/local/bin/tool' -> '/opt/tool'
```

---

## 六、注意事项

1. **硬链接限制：**
   - 无法跨越不同的文件系统（分区）。
   - 无法链接目录（除非使用 `-d` 且具有超级权限，但通常会失败）。

2. **符号链接优势：**
   - 可跨文件系统。
   - 可链接目录。
   - 即使目标文件不存在，链接仍可创建。

3. **删除链接文件不会影响原文件**，但删除目标文件后，符号链接将失效（称为“悬挂链接”）。

---

## 七、更多信息

官方文档：

- 在线帮助：<http://www.gnu.org/software/coreutils/>

- 终端查看：  

  ```bash
  info coreutils 'ln invocation'
  man ln
  ```