
# 🧩 Linux `split` 命令操作教程文档

------

## 一、文档目的

本操作文档用于指导 Linux 系统用户掌握 **`split` 命令** 的使用方法。
 `split` 是 Linux 中用于将大文件分割成多个小文件的工具，常应用于**分卷压缩、数据备份、日志切割、文件传输**等场景。

------

## 二、命令简介

`split` 的主要功能是：

> 将一个大文件按照指定的 **大小** 或 **行数** 拆分成多个小文件。

它不会修改文件内容，只是对原文件进行“物理分割”，每个分割后的文件可以独立存储、传输或合并。

------

## 三、基本语法

```bash
split [选项] [输入文件] [输出文件前缀]
```

当不指定输入文件时，`split` 会从 **标准输入（stdin）** 读取数据，常与管道 `|` 结合使用。

------

## 四、常用参数说明

| 参数                       | 说明                                             |
| -------------------------- | ------------------------------------------------ |
| `-b SIZE`                  | 按字节大小分割，例如 `-b 100M`（每个文件 100MB） |
| `-C SIZE`                  | 按块大小分割，但不拆分行（适用于文本文件）       |
| `-l NUM`                   | 按行数分割文件，例如 `-l 1000`                   |
| `-d`                       | 使用数字编号（00, 01, 02...）代替字母编号        |
| `--additional-suffix=.ext` | 为输出文件添加扩展名                             |
| `--verbose`                | 显示分割进度                                     |
| `--help`                   | 查看帮助信息                                     |

------

## 五、输出文件命名规则

默认输出文件命名格式：

```
xaa, xab, xac, ...
```

若指定输出前缀：

```bash
split -b 100M largefile.zip part-
```

输出结果：

```
part-aa
part-ab
part-ac
...
```

若使用数字编号：

```bash
split -b 100M -d largefile.zip part-
```

输出结果：

```
part-00
part-01
part-02
...
```

------

## 六、常见操作示例

------

### 1️⃣ 按大小分割文件

```bash
split -b 100M bigfile.iso part-
```

说明：

- 将 `bigfile.iso` 拆分为每个 100MB 的小文件；

- 输出文件前缀为 `part-`；

- 生成结果：

  ```
  part-aa
  part-ab
  part-ac
  ```

------

### 2️⃣ 按行数分割文件

```bash
split -l 5000 data.log log-part-
```

说明：

- 将 `data.log` 每 5000 行分割为一个文件；
- 输出文件命名为 `log-part-aa`、`log-part-ab`、`log-part-ac` 等。

------

### 3️⃣ 按块大小分割但不截断行

```bash
split -C 50M access.log access-
```

说明：

- 每个文件最大 50MB；
- 若遇到行尾，确保不截断一整行；
- 常用于分割大型文本或日志。

------

### 4️⃣ 分割结果带扩展名

```bash
split -b 200M -d --additional-suffix=.part largefile.bin chunk-
```

生成结果：

```
chunk-00.part
chunk-01.part
chunk-02.part
...
```

------

### 5️⃣ 与 tar 管道配合进行分卷压缩

```bash
tar -czf - /data/project | split -b 100M - project.tar.gz.part-
```

说明：

- `tar` 将目录打包并压缩后输出到标准输出；

- `split` 将压缩数据流按 100MB 一卷分割；

- 输出结果：

  ```
  project.tar.gz.part-aa
  project.tar.gz.part-ab
  project.tar.gz.part-ac
  ```

------

### 6️⃣ 合并分卷文件

```bash
cat part-* > largefile.iso
```

或对压缩包直接解压：

```bash
cat project.tar.gz.part-* | tar -xzvf -
```

> ⚠️ 注意：文件顺序必须正确，`ls -1v` 可查看自然顺序列表。

------

## 七、验证文件完整性

### 1️⃣ 使用 md5sum

```bash
md5sum part-* > checksum.md5
```

> 在目标主机合并后使用：

```bash
md5sum -c checksum.md5
```

### 2️⃣ 使用 sha256sum（更安全）

```bash
sha256sum part-* > checksum.sha256
```

------

## 八、应用场景

| 场景           | 示例命令                           | 说明                                 |
| -------------- | ---------------------------------- | ------------------------------------ |
| 📦 分卷压缩     | `tar -czf - /data                  | split -b 500M - backup.tar.gz.part-` |
| 🪵 分割日志     | `split -l 1000000 access.log log-` | 防止日志过大                         |
| 💾 超大文件传输 | `split -b 1G video.mp4 video-`     | 便于通过 U 盘/网盘传输               |
| 🧪 数据集切分   | `split -l 1000 dataset.csv part-`  | 模型训练/测试数据分离                |
| 🔁 异地备份     | `split -b 500M db_dump.sql dump-`  | 分块上传云存储                       |

------

## 九、合并与验证流程图（文字版）

```
       大文件（bigfile.tar.gz）
                 │
                 ▼
       split 分割为多块
        ├── part-aa
        ├── part-ab
        ├── part-ac
                 │
                 ▼
        传输/备份/存储
                 │
                 ▼
     cat part-* > bigfile.tar.gz
                 │
                 ▼
          验证完整性(md5sum)
```

------

## 十、注意事项与最佳实践

1. **分卷顺序非常重要**
    合并时文件必须按字母或数字顺序排列：

   ```bash
   ls -1v part-*  # 查看自然排序
   ```

2. **split 不会压缩文件**
    如果需要压缩，请结合 `tar` 或 `gzip` 使用。

3. **分卷大小单位**

   - `K`（千字节）
   - `M`（兆字节）
   - `G`（千兆字节）
      示例：`-b 512M`、`-b 2G`

4. **存储安全建议**

   - 生成校验文件 (`md5sum` / `sha256sum`)；
   - 分卷存储时避免文件重命名；
   - 对重要数据可配合加密传输（如 `gpg`）。

5. **合并前检查完整性**
    确认所有分卷文件都存在且大小一致。

------

## 十一、命令速查表

| 功能             | 命令                                                        |
| ---------------- | ----------------------------------------------------------- |
| 按大小分割文件   | `split -b 100M bigfile.zip part-`                           |
| 按行数分割文件   | `split -l 1000 data.log log-`                               |
| 按块分割不截断行 | `split -C 10M text.txt text-`                               |
| 数字编号输出     | `split -b 500M -d file.bin chunk-`                          |
| 添加扩展名       | `split -b 100M -d --additional-suffix=.part file.bin part-` |
| 合并文件         | `cat part-* > file.bin`                                     |
| 验证完整性       | `md5sum part-* > md5list.txt`                               |

------

## 十二、总结

✅ **split 是 Linux 中用于文件分割的核心命令。**
 它本身不压缩文件，但能灵活控制分卷大小，
 结合 `tar`、`gzip`、`zip` 等命令可以实现：

- 高效分卷备份
- 大文件分发
- 数据安全传输