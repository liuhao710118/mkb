# 🧭 GNU grep 使用教程文档

## 一、命令简介

`grep`（Global Regular Expression Print）是 Linux / Unix 系统中最常用的文本搜索工具之一。
 它用于在文件或输入流中查找**匹配指定模式（PATTERN）**的行，并输出结果。

> grep 是系统管理员、开发者、测试工程师分析日志、排查配置、过滤输出的必备工具。

------

## 二、命令格式

```bash
grep [选项] PATTERN [文件...]
```

- **PATTERN**：搜索模式（可以是字符串或正则表达式）
- **FILE**：要搜索的一个或多个文件（可省略，从标准输入读取）

**示例：**

```bash
grep "error" /var/log/syslog
```

搜索文件中包含 “error” 的所有行。

------

## 三、正则表达式类型

`grep` 支持多种模式解释方式：

| 选项                    | 说明                                      |
| ----------------------- | ----------------------------------------- |
| `-E, --extended-regexp` | 使用扩展正则表达式（ERE），等价于 `egrep` |
| `-F, --fixed-strings`   | 使用固定字符串（不解析正则）              |
| `-G, --basic-regexp`    | 使用基本正则表达式（BRE），默认模式       |
| `-P, --perl-regexp`     | 使用 Perl 风格正则（功能最强）            |

**示例：**

```bash
grep -E "ERROR|WARN" app.log     # 匹配 ERROR 或 WARN
grep -F "a.b" file.txt           # 匹配字面字符串 a.b
```

------

## 四、常用匹配选项

| 选项                 | 功能说明                                   |
| -------------------- | ------------------------------------------ |
| `-e PATTERN`         | 指定搜索模式，可多次使用多个模式           |
| `-f FILE`            | 从文件中读取模式（每行一个）               |
| `-i, --ignore-case`  | 忽略大小写匹配                             |
| `-w, --word-regexp`  | 仅匹配整个单词                             |
| `-x, --line-regexp`  | 匹配整行                                   |
| `-v, --invert-match` | 反向匹配（显示不匹配的行）                 |
| `-z, --null-data`    | 用 `\0` 而非换行符分隔行（处理二进制数据） |

**示例：**

```bash
grep -i "error" log.txt          # 忽略大小写
grep -w "fail" result.txt        # 匹配单词 fail，不匹配 failed
grep -v "^#" config.conf         # 过滤注释行
```

------

## 五、输出控制选项

| 选项                        | 功能说明                                                |
| --------------------------- | ------------------------------------------------------- |
| `-n, --line-number`         | 显示行号                                                |
| `-b, --byte-offset`         | 显示字节偏移量                                          |
| `-H, --with-filename`       | 输出文件名（默认在多文件搜索时）                        |
| `-h, --no-filename`         | 不显示文件名（单文件时默认）                            |
| `-o, --only-matching`       | 仅显示匹配部分                                          |
| `-c, --count`               | 只输出匹配的行数                                        |
| `-l, --files-with-matches`  | 仅显示包含匹配内容的文件名                              |
| `-L, --files-without-match` | 仅显示不包含匹配内容的文件名                            |
| `-m NUM`                    | 匹配 NUM 条后停止                                       |
| `-q, --quiet`               | 静默模式，不输出结果，只返回状态码                      |
| `--color[=WHEN]`            | 高亮显示匹配内容（WHEN 可为 `always`, `never`, `auto`） |

**示例：**

```bash
grep -n "listen" nginx.conf      # 显示行号
grep -c "failed" auth.log        # 显示匹配行数
grep -l "timeout" /etc/nginx/*   # 显示含有 timeout 的文件
grep -o "[0-9]\+" data.txt       # 只输出匹配的数字
grep --color=auto "error" log.txt
```

------

## 六、目录与文件控制选项

| 选项                          | 功能说明                                     |
| ----------------------------- | -------------------------------------------- |
| `-r, --recursive`             | 递归搜索目录                                 |
| `-R, --dereference-recursive` | 递归并跟随符号链接                           |
| `-d ACTION`                   | 目录处理方式：`read` / `recurse` / `skip`    |
| `-D ACTION`                   | 设备、FIFO、socket 处理方式：`read` / `skip` |
| `--include=PATTERN`           | 只搜索匹配文件名的文件                       |
| `--exclude=PATTERN`           | 排除匹配的文件或目录                         |
| `--exclude-from=FILE`         | 从文件中读取要排除的文件模式                 |
| `--exclude-dir=PATTERN`       | 跳过匹配的目录                               |

**示例：**

```bash
grep -r "192.168" /etc/
grep -R "server" /etc/nginx/
grep -r "error" /var/log --include="*.log"
grep -r "password" /etc --exclude-dir="backup"
```

------

## 七、上下文显示选项

| 选项                           | 功能说明                  |
| ------------------------------ | ------------------------- |
| `-A NUM, --after-context=NUM`  | 显示匹配行及其后 NUM 行   |
| `-B NUM, --before-context=NUM` | 显示匹配行及其前 NUM 行   |
| `-C NUM, --context=NUM`        | 显示匹配行及其前后 NUM 行 |
| `--group-separator=SEP`        | 自定义匹配组之间的分隔符  |
| `--no-group-separator`         | 不显示组分隔符            |

**示例：**

```bash
grep -A 2 "error" app.log        # 显示匹配行及后 2 行
grep -B 3 "warning" app.log      # 显示匹配行及前 3 行
grep -C 1 "failed" app.log       # 显示匹配行及上下 1 行
```

------

## 八、文件内容与编码控制

| 选项                      | 功能说明                                                  |
| ------------------------- | --------------------------------------------------------- |
| `--binary-files=TYPE`     | 指定二进制文件处理方式：`binary`、`text`、`without-match` |
| `-a, --text`              | 把二进制文件当作文本处理                                  |
| `-I`                      | 忽略二进制文件（等价于 `--binary-files=without-match`）   |
| `-U, --binary`            | 不移除行尾的 CR（Windows 文件）                           |
| `-u, --unix-byte-offsets` | 报告偏移时忽略 CR 字符                                    |

------

## 九、执行结果状态码

| 状态码 | 含义                       |
| ------ | -------------------------- |
| `0`    | 匹配成功                   |
| `1`    | 无匹配                     |
| `2`    | 出现错误（例如文件不存在） |

这在脚本中非常常用：

```bash
if grep -q "success" output.log; then
  echo "检测到成功"
else
  echo "未检测到"
fi
```

------

## 十、组合与实战示例

### 1️⃣ 实时过滤日志输出

```bash
tail -f app.log | grep --color=auto "ERROR"
```

### 2️⃣ 过滤注释和空行

```bash
grep -vE "^#|^$" /etc/nginx/nginx.conf
```

### 3️⃣ 多模式匹配

```bash
grep -E "ERROR|WARN|FATAL" server.log
```

### 4️⃣ 统计匹配次数

```bash
grep -o "timeout" app.log | wc -l
```

### 5️⃣ 搜索源码中函数定义

```bash
grep -R "def " . --include="*.py"
```

### 6️⃣ 搜索配置文件并显示上下文

```bash
grep -C 2 "listen" /etc/nginx/nginx.conf
```

------

## 十一、扩展命令

> GNU grep 中包含的别名命令（已弃用）：

| 命令    | 等价于    | 说明           |
| ------- | --------- | -------------- |
| `egrep` | `grep -E` | 扩展正则匹配   |
| `fgrep` | `grep -F` | 固定字符串匹配 |

建议直接使用 `grep -E`、`grep -F`。

------

## 十二、总结与最佳实践

| 任务                        | 推荐命令                                    |
| --------------------------- | ------------------------------------------- |
| 搜索日志关键字并高亮        | `grep --color=auto "error" app.log`         |
| 忽略大小写搜索              | `grep -i "keyword" file`                    |
| 搜索目录中所有 `.conf` 文件 | `grep -r "setting" /etc --include="*.conf"` |
| 过滤空行与注释              | `grep -vE "^#                               |
| 只看匹配行内容              | `grep -o "pattern" file`                    |
| 统计匹配次数                | `grep -o "pattern" file                     |
| 查看上下文                  | `grep -C 3 "pattern" file`                  |

------

是否希望我把这份教程 **整理成 PDF（带目录和示例对照）**，方便打印或收藏？
 我可以帮你生成一份排版精美的版本。