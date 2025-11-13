# 🗑️ Linux `rm` 命令教程文档

## 一、命令简介

`rm`（remove）命令用于 **删除文件或目录**。
 它是 Linux 文件系统管理中最常用但也最危险的命令之一——**执行后无法恢复**（除非使用专业数据恢复工具）。

------

## 二、命令格式

```bash
rm [选项] 文件或目录
```

常见形式：

- 删除文件 → `rm file.txt`
- 删除多个文件 → `rm file1 file2`
- 删除目录及内容 → `rm -r dirname`

------

## 三、常用选项说明

| 选项                 | 含义                                  |
| -------------------- | ------------------------------------- |
| `-f`                 | 强制删除（force），不提示确认         |
| `-i`                 | 删除前逐一提示确认                    |
| `-I`                 | 删除 3 个以上文件或递归删除前提示一次 |
| `-r`                 | 递归删除目录及其内容（recursive）     |
| `-R`                 | 同 `-r`，推荐使用大写版本以提高兼容性 |
| `-v`                 | 显示详细删除过程（verbose）           |
| `--preserve-root`    | 默认启用，防止删除 `/` 根目录         |
| `--no-preserve-root` | 允许删除 `/`（⚠️极其危险！）           |

------

## 四、基本用法示例

### 1. 删除单个文件

```bash
rm file.txt
```

👉 删除当前目录下的 `file.txt`。

------

### 2. 删除多个文件

```bash
rm file1.txt file2.txt file3.txt
```

👉 同时删除多个文件。

------

### 3. 删除时提示确认

```bash
rm -i important.txt
```

输出示例：

```
rm: remove regular file 'important.txt'? y
```

👉 输入 `y` 确认删除。

------

### 4. 递归删除目录及其内容

```bash
rm -r /tmp/testdir
```

👉 删除目录及所有子目录、文件。

------

### 5. 强制删除，不提示

```bash
rm -rf /tmp/testdir
```

👉 即使目录中有只读文件，也不提示直接删除。
 ⚠️ 常用于自动化脚本，但极易造成误删！

------

### 6. 显示删除过程

```bash
rm -rv /tmp/testdir
```

输出示例：

```
removed '/tmp/testdir/file1.txt'
removed '/tmp/testdir/file2.txt'
removed directory '/tmp/testdir'
```

------

### 7. 删除带通配符的文件

```bash
rm *.log
```

👉 删除当前目录下所有 `.log` 文件。

------

### 8. 仅删除特定前缀或后缀文件

```bash
rm data_*.csv
rm *.bak
```

------

### 9. 删除空目录（推荐使用 `rmdir`）

```bash
rmdir emptydir
```

👉 仅当目录为空时删除。

------

## 五、常见错误与原因

| 错误提示                                          | 原因                | 解决方法               |
| ------------------------------------------------- | ------------------- | ---------------------- |
| `rm: cannot remove 'file': Permission denied`     | 权限不足            | 使用 `sudo` 或修改权限 |
| `rm: cannot remove 'dir': Is a directory`         | 目标是目录未加 `-r` | 添加 `-r` 参数         |
| `rm: missing operand`                             | 未指定要删除的文件  | 确认命令格式           |
| `rm: cannot remove ‘file’: Read-only file system` | 文件系统只读        | 检查挂载参数或系统状态 |

------

## 六、安全删除技巧

### 1. 删除前确认 (`-i`)

```bash
rm -i *.conf
```

✅ 适合删除重要配置文件。

------

### 2. 批量删除时确认一次 (`-I`)

```bash
rm -I *.log
```

✅ 删除多个文件时仅提示一次。

------

### 3. 保护根目录

系统默认保护根路径：

```bash
rm -rf /
```

会报错：

```
rm: it is dangerous to operate recursively on '/'
rm: use --no-preserve-root to override this failsafe
```

⚠️ 切勿使用 `--no-preserve-root`！

------

### 4. 删除前查看目标

```bash
ls | grep keyword
```

确认要删除的文件列表后再执行 `rm`。

------

### 5. 删除指定目录下的特定文件

```bash
find /var/log -name "*.old" -exec rm -v {} \;
```

👉 结合 `find` 命令更灵活、安全。

------

## 七、实践示例

### 示例 1：清理日志文件

```bash
rm -f /var/log/*.log
```

### 示例 2：清空缓存目录

```bash
rm -rf /tmp/cache/
```

### 示例 3：删除旧备份文件

```bash
find /backup -name "*.bak" -mtime +30 -exec rm -v {} \;
```

👉 删除 30 天前的 `.bak` 文件。

### 示例 4：模拟删除（使用 `ls` 代替）

```bash
ls -d *.log
```

👉 先用 `ls` 查看匹配文件，确认无误后再删除。

------

## 八、危险操作警告 ⚠️

以下命令极度危险，**执行后系统可能无法恢复**：

| 命令               | 危险原因                 |
| ------------------ | ------------------------ |
| `rm -rf /`         | 删除整个系统根目录       |
| `rm -rf /*`        | 删除系统所有文件         |
| `rm -rf ./*`       | 当前目录所有内容被删除   |
| `rm -rf ~`         | 删除当前用户的所有文件   |
| `sudo rm -rf /etc` | 系统配置被破坏，无法启动 |

💡 建议在任何 `rm -rf` 前都执行：

```bash
echo "即将删除: /path/to/target"
```

确认路径后再执行。

------

## 九、推荐替代方案（更安全）

| 工具                    | 说明                                             |
| ----------------------- | ------------------------------------------------ |
| `trash-cli`             | 删除文件时移动到回收站（命令：`trash-put file`） |
| `gio trash`             | 桌面环境下移动到系统回收站                       |
| `mv file /tmp/recycle/` | 临时转移文件，再人工确认删除                     |
| `alias rm='rm -i'`      | 全局启用删除确认（见下方）                       |

### 设置删除保护别名

在 `~/.bashrc` 中加入：

```bash
alias rm='rm -i'
```

使删除命令默认提示确认。

------

## 十、命令总结表

| 操作             | 命令示例                          |
| ---------------- | --------------------------------- |
| 删除文件         | `rm file.txt`                     |
| 删除多个文件     | `rm file1 file2 file3`            |
| 删除目录         | `rm -r dirname`                   |
| 强制删除         | `rm -rf dirname`                  |
| 删除前确认       | `rm -i file`                      |
| 批量删除提示一次 | `rm -I *.log`                     |
| 显示删除过程     | `rm -v file`                      |
| 删除旧文件       | `find . -mtime +7 -exec rm {} \;` |

------

## 十一、总结与最佳实践 ✅

- **安全第一**：删除前请先确认路径；

- **推荐使用**：`rm -i` 或 `alias rm='rm -i'`；

- **批量删除前先用 `ls` 预览**；

- **自动化脚本中使用 `rm -rf` 时要小心拼接路径错误**；

- **重要文件删除前可先备份**：

  ```bash
  cp important.conf important.conf.bak
  ```

