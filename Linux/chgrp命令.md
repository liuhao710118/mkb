`chgrp` 是 Linux/Unix 中用于**修改文件或目录所属组（group）**的命令，功能相当于 `chown :组 文件`，但更专注、更直观。

------

## 一、基本语法

```bash
chgrp [选项] 组名 文件或目录
```

------

## 二、常见用法示例

### 1️⃣ 修改文件所属组

```bash
chgrp dev file.txt
```

------

### 2️⃣ 递归修改目录及其所有内容（常用）

```bash
chgrp -R dev /data/app
```

------

### 3️⃣ 使用 GID 修改

```bash
chgrp 1001 file.txt
```

------

### 4️⃣ 仅当发生改变时显示

```bash
chgrp -c dev file.txt
```

------

### 5️⃣ 显示修改过程

```bash
chgrp -v dev file.txt
```

------

### 6️⃣ 不跟随符号链接

```bash
chgrp -h dev symlink
```

------

## 三、常用选项速查

| 选项               | 说明               |
| ------------------ | ------------------ |
| `-R`               | 递归修改           |
| `-v`               | 显示修改过程       |
| `-c`               | 仅显示发生变更的项 |
| `-h`               | 修改符号链接本身   |
| `--reference=FILE` | 参考其他文件的属组 |

示例：

```bash
chgrp --reference=a.txt b.txt
```

------

## 四、与 `chown` 的区别

| 命令    | 能做什么        |
| ------- | --------------- |
| `chgrp` | 只修改属组      |
| `chown` | 修改属主 + 属组 |

等价写法：

```bash
chgrp dev file.txt
chown :dev file.txt
```

------

## 五、权限与注意事项 ⚠️

- 普通用户 **只能把文件改为自己所属的组**
- 修改为其他组通常需要 `root` 权限
- `-R` 对系统目录慎用，容易破坏权限

------

## 六、常见实际场景

### Web 服务目录

```bash
chgrp -R www /var/www/html
chmod -R g+w /var/www/html
```

### 多用户共享目录

```bash
chgrp dev /shared
chmod 2775 /shared   # 设置 SGID，继承组
```

