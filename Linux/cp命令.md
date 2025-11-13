# 🧭 Linux `cp` 命令教程文档

## 一、命令简介

`cp`（copy）命令用于 **复制文件或目录**。
 它是 Linux 中最常用的文件管理命令之一，可将文件从一个位置复制到另一个位置，并支持递归、覆盖、权限保持等功能。

------

## 二、命令格式

```bash
cp [选项] 源文件/目录 目标文件/目录
```

常见格式：

- 复制文件 → `cp source.txt /home/user/`
- 复制并重命名 → `cp source.txt target.txt`
- 复制目录（递归）→ `cp -r /dir1 /dir2`

------

## 三、常用选项说明

| 选项                   | 含义                                                         |
| ---------------------- | ------------------------------------------------------------ |
| `-a`                   | 归档模式（相当于 `-dR --preserve=all`），保持文件属性与符号链接 |
| `-r`                   | 递归复制目录及其内容                                         |
| `-R`                   | 同 `-r`，推荐使用大写版本以兼容性更好                        |
| `-i`                   | 覆盖前提示确认（interactive）                                |
| `-f`                   | 强制覆盖，不提示                                             |
| `-u`                   | 仅在源文件比目标文件新或目标文件不存在时复制                 |
| `-v`                   | 显示详细复制过程（verbose）                                  |
| `-p`                   | 保留文件的权限、时间戳、所有者等信息                         |
| `--parents`            | 保留原有路径层次                                             |
| `--remove-destination` | 删除目标再复制（确保完全替换）                               |
| `--backup`             | 复制时自动为目标文件生成备份（例如 `file.txt~`）             |

------

## 四、基本用法示例

### 1. 复制单个文件

```bash
cp file.txt /tmp/
```

👉 将当前目录下的 `file.txt` 复制到 `/tmp/`。

------

### 2. 复制并重命名

```bash
cp file.txt /tmp/newfile.txt
```

👉 将文件复制到 `/tmp/` 并重命名为 `newfile.txt`。

------

### 3. 覆盖前提示

```bash
cp -i file.txt /tmp/
```

👉 若目标存在同名文件，会提示是否覆盖。

------

### 4. 递归复制整个目录

```bash
cp -r /home/user/docs /backup/
```

👉 将整个 `docs` 目录（包括子目录与文件）复制到 `/backup/`。

------

### 5. 保留权限与时间戳

```bash
cp -p source.txt /backup/
```

👉 复制时保留原文件的权限、时间、所有者等信息。

------

### 6. 保留目录层次结构

```bash
cp --parents ./src/config/app.conf /backup/
```

👉 将文件复制到 `/backup/src/config/app.conf`，保留原路径结构。

------

### 7. 显示复制进度

```bash
cp -v file1.txt file2.txt /tmp/
```

👉 输出类似：

```
'file1.txt' -> '/tmp/file1.txt'
'file2.txt' -> '/tmp/file2.txt'
```

------

### 8. 仅复制更新文件

```bash
cp -u *.log /var/logs/backup/
```

👉 仅当源日志文件比目标目录中更新时才复制。

------

### 9. 使用通配符批量复制

```bash
cp *.jpg /home/user/pictures/
```

👉 复制当前目录下所有 `.jpg` 文件。

------

### 10. 强制覆盖而不提示

```bash
cp -f config.yaml /etc/app/
```

👉 不经确认，直接覆盖。

------

## 五、高级用法示例

### 1. 同时复制多个文件

```bash
cp file1.txt file2.txt /tmp/
```

👉 将多个文件复制到同一个目录中。

------

### 2. 复制到当前目录并重命名

```bash
cp /etc/hosts ./hosts_backup
```

------

### 3. 带备份功能的复制

```bash
cp --backup important.conf /etc/
```

👉 若目标已存在，则生成备份文件，如：`important.conf~`。

------

### 4. 结合通配符与归档模式

```bash
cp -av /var/www/html /backup/
```

👉 复制网站目录，保留文件属性、链接及时间信息。

------

### 5. 覆盖时删除原目标（防止硬链接或权限问题）

```bash
cp --remove-destination config.json /etc/myapp/
```

------

## 六、常见错误与解决方法

| 错误提示                                            | 原因                         | 解决方法                              |
| --------------------------------------------------- | ---------------------------- | ------------------------------------- |
| `cp: cannot stat 'file': No such file or directory` | 源文件不存在                 | 检查路径是否正确                      |
| `cp: omitting directory 'dir'`                      | 未加 `-r` 选项复制目录       | 使用 `cp -r`                          |
| `Permission denied`                                 | 权限不足                     | 使用 `sudo` 或调整文件权限            |
| `cp: cannot overwrite directory`                    | 复制目录时目标已存在同名文件 | 删除目标或使用 `--remove-destination` |

------

## 七、实践建议

1. **重要目录复制时建议加 `-a` 归档模式**
    例如备份配置文件：

   ```bash
   cp -a /etc/nginx /backup/nginx_$(date +%F)/
   ```

2. **不确定是否覆盖时使用 `-i`**
    防止误操作。

3. **批量复制前可先用 `ls` 验证匹配结果**
    避免通配符误选文件。

4. **大规模数据迁移建议使用 `rsync`**
    `rsync` 比 `cp` 更高效，可断点续传。

------

## 八、命令总结

| 操作       | 示例命令                   |
| ---------- | -------------------------- |
| 复制文件   | `cp file.txt /tmp/`        |
| 复制目录   | `cp -r dir/ /backup/`      |
| 覆盖前提示 | `cp -i file /etc/`         |
| 保留属性   | `cp -p file /backup/`      |
| 显示过程   | `cp -v file /tmp/`         |
| 自动备份   | `cp --backup config /etc/` |
| 仅复制更新 | `cp -u *.log /backup/`     |

