`gzip`是 Linux/Unix 系统中一个非常常用的**文件压缩和解压缩工具**。它使用 LZ77 算法，能够显著减小文件大小，节省存储空间和网络传输带宽。

## 🎯 基本语法

```
gzip [选项] [文件...]
gunzip [选项] [文件.gz...]  # 解压的另一种方式
```

## 📋 常用选项详解

| 选项        | 说明                                         | 示例                     |
| ----------- | -------------------------------------------- | ------------------------ |
| `-d`        | 解压缩文件                                   | `gzip -d file.gz`        |
| `-c`        | 将结果输出到标准输出，不改变原文件           | `gzip -c file > file.gz` |
| `-k`        | 压缩后保留原文件                             | `gzip -k file.txt`       |
| `-r`        | 递归压缩目录下的所有文件                     | `gzip -r directory/`     |
| `-v`        | 显示压缩详情                                 | `gzip -v file.txt`       |
| `-l`        | 列出压缩文件信息                             | `gzip -l file.gz`        |
| `-1`到 `-9` | 压缩级别（1最快压缩比最低，9最慢压缩比最高） | `gzip -9 file.txt`       |
| `-f`        | 强制压缩，覆盖已存在的文件                   | `gzip -f file.txt`       |

## 🚀 实际使用示例

### 1. **基本压缩和解压**

```
# 压缩文件（原文件会被删除，生成 .gz 文件）
gzip file.txt
# 生成：file.txt.gz

# 解压文件
gzip -d file.txt.gz
# 或者使用 gunzip
gunzip file.txt.gz
```

### 2. **压缩时保留原文件**

```
# 方法1：使用 -k 选项
gzip -k important.log

# 方法2：使用输出重定向
gzip -c file.txt > file.txt.gz

# 方法3：先复制再压缩
cp file.txt file_backup.txt
gzip file.txt
```

### 3. **查看压缩信息**

```
# 查看压缩文件的详细信息
gzip -l data.gz
# 输出示例：
# compressed        uncompressed  ratio uncompressed_name
# 1024              4096          75.0% data

# 查看压缩比和进度
gzip -v large_file.iso
```

### 4. **调整压缩级别**

```
# 最快压缩（压缩比低）
gzip -1 large_file.iso

# 默认压缩（平衡速度和压缩比）
gzip data.txt

# 最佳压缩（速度慢但压缩比高）
gzip -9 huge_database.sql

# 查看不同级别的压缩效果
time gzip -1 bigfile.img    # 查看压缩时间
time gzip -9 bigfile.img    # 对比压缩效果
```

### 5. **批量处理文件**

```
# 压缩多个文件（每个文件单独压缩）
gzip file1.txt file2.txt file3.txt

# 使用通配符
gzip *.log

# 递归压缩目录下所有文件
gzip -r project_directory/
```

### 6. **结合其他命令使用**

```
# 压缩 tar 归档文件
tar czf archive.tar.gz directory/    # 常用组合

# 查看压缩的文本文件内容
zcat file.txt.gz                    # 类似 cat
zless file.txt.gz                   # 类似 less
zgreap "error" *.gz                 # 在压缩文件中搜索

# 管道操作：压缩命令输出
ps aux | gzip > processes.gz
```

## 🔧 实用技巧和场景

### **日志文件管理**

```
# 压缩旧日志文件，节省空间
gzip -v /var/log/*.log.1

# 查找并压缩大文件
find /var/log -name "*.log" -size +100M -exec gzip {} \;

# 按日期压缩日志
gzip /var/log/nginx/access.log.$(date -d yesterday +%Y%m%d)
```

### **备份脚本示例**

```
#!/bin/bash
# 备份脚本：压缩重要数据
BACKUP_DIR="/home/backup"
DATE=$(date +%Y%m%d)

# 压缩数据库备份
mysqldump -u root -p database > db_backup.sql
gzip -9 db_backup.sql
mv db_backup.sql.gz $BACKUP_DIR/db_$DATE.sql.gz

# 压缩网站文件
tar czf $BACKUP_DIR/website_$DATE.tar.gz /var/www/html/
```

### **性能测试比较**

```
# 测试不同压缩级别的效果
echo "=== 压缩级别测试 ==="
for level in {1..9}; do
    echo -n "级别 $level: "
    cp large_file.txt test_file.txt
    time gzip -$level test_file.txt
    du -h test_file.txt.gz
    rm test_file.txt.gz
done
```

## ⚠️ 注意事项

### **重要提醒**

```
# 1. 原文件默认会被删除！
gzip important.txt    # important.txt 会被删除，生成 important.txt.gz

# 2. 使用 -k 保留原文件
gzip -k important.txt # 保留 important.txt，同时生成 important.txt.gz

# 3. 不能直接压缩目录
gzip directory/      # ❌ 错误！需要先用 tar
tar czf dir.tar.gz directory/  # ✅ 正确
```

### **压缩文件检查**

```
# 检查压缩文件完整性
gzip -t archive.gz && echo "文件完好" || echo "文件损坏"

# 测试解压但不实际解压
gzip -t -v *.gz
```

## 🔄 与其他压缩工具对比

| 工具      | 压缩比 | 速度 | 特点                   | 适用场景               |
| --------- | ------ | ---- | ---------------------- | ---------------------- |
| **gzip**  | 中等   | 快   | 标准工具，广泛支持     | 日常文件、日志压缩     |
| **bzip2** | 高     | 慢   | 压缩比高               | 需要高压缩比的场景     |
| **xz**    | 最高   | 最慢 | 极致压缩比             | 发行版软件包、长期归档 |
| **zip**   | 中等   | 中等 | 跨平台，可包含多个文件 | Windows/Linux 共享     |

## 💡 实用小贴士

```
# 快速创建测试文件
seq 1000000 > test.txt    # 生成100万行测试文件

# 比较压缩效果
ls -lh test.txt          # 查看原文件大小
gzip test.txt && ls -lh test.txt.gz    # 查看压缩后大小

# 一次性解压多个文件
gunzip *.gz

# 在脚本中使用
if gzip -t backup.gz; then
    echo "压缩文件完好，开始解压..."
    gunzip backup.gz
else
    echo "压缩文件损坏！"
fi
```

## 🎯 总结

**gzip 是最常用的压缩工具之一，特点是：**

- •

  ✅ **简单易用**：基本压缩解压一行命令搞定

- •

  ✅ **高效快速**：压缩解压速度都很理想

- •

  ✅ **广泛支持**：几乎所有 Linux 系统都预装

- •

  ✅ **管道友好**：可以轻松结合其他命令使用

**记住核心用法：**

- •

  **压缩**：`gzip 文件名`

- •

  **解压**：`gzip -d 文件名.gz`或 `gunzip 文件名.gz`

- •

  **保留原文件**：一定要加 `-k`选项

掌握了 gzip，你就拥有了处理文件压缩的基本能力！