`rsync` 是运维里**必会**的一把瑞士军刀🛠️，用来做**文件/目录同步、备份、增量传输** scp 能用的地方，rsync 都能更好

------

## 一、最常用格式（记住这一个就够了）

```bash
rsync [选项] 源路径 目标路径
```

本地 ↔ 本地
本地 ↔ 远程
远程 ↔ 远程（通过 ssh）

------

## 二、最常用参数（重点）

| 参数         | 作用                               |
| ------------ | ---------------------------------- |
| `-a`         | 归档模式（= -rlptgoD，**最常用**） |
| `-v`         | 显示详细过程                       |
| `-z`         | 传输时压缩                         |
| `-P`         | 显示进度 + 支持断点续传            |
| `--delete`   | 删除目标端多余文件（⚠️危险）        |
| `--exclude`  | 排除文件/目录                      |
| `--dry-run`  | 演练，不真正执行（强烈推荐）       |
| `-e ssh`     | 指定使用 ssh 传输                  |
| `--progress` | 显示传输进度                       |

👉 **99% 场景：**

```bash
rsync -avzP 源 目标
```

------

## 三、本地同步示例

### 1️⃣ 同步目录（推荐）

```bash
rsync -avz /data/src/ /data/dest/
```

⚠️ **注意结尾的 `/`**

| 写法    | 含义              |
| ------- | ----------------- |
| `/src/` | 复制 src 里的内容 |
| `/src`  | 复制整个 src 目录 |

------

### 2️⃣ 只看效果，不执行

```bash
rsync -avz --dry-run /data/src/ /data/dest/
```

运维必备救命参数😅

------

## 四、本地 → 远程（最常用）

```bash
rsync -avzP /data/ user@192.168.1.10:/backup/data/
```

指定端口：

```bash
rsync -avzP -e "ssh -p 2222" /data/ user@ip:/backup/
```

------

## 五、远程 → 本地

```bash
rsync -avzP user@ip:/backup/data/ /data/
```

------

## 六、删除目标端多余文件（镜像同步）

⚠️ **非常危险，用前一定 dry-run**

```bash
rsync -avz --delete /data/src/ /data/dest/
```

建议流程：

```bash
rsync -avz --delete --dry-run /data/src/ /data/dest/
# 确认没问题再执行
rsync -avz --delete /data/src/ /data/dest/
```

------

## 七、排除文件 / 目录

### 排除单个目录

```bash
rsync -avz --exclude logs /data/ /backup/
```

### 排除多种

```bash
rsync -avz \
  --exclude logs \
  --exclude "*.tmp" \
  --exclude cache \
  /data/ /backup/
```

### 用文件统一管理（推荐）

```bash
rsync -avz --exclude-from=exclude.txt /data/ /backup/
```

`exclude.txt`：

```text
logs/
*.tmp
cache/
```

------

## 八、只同步文件列表（不传内容）

```bash
rsync -avzn /data/ /backup/
```

👉 常用于**核对目录差异**

------

## 九、限速（防止打爆带宽）

```bash
rsync -avz --bwlimit=2000 /data/ /backup/
```

单位：**KB/s**

------

## 十、rsync + sshpass（无人值守）

```bash
sshpass -p 'password' rsync -avz /data/ user@ip:/backup/
```

⚠️ 建议：
👉 **生产环境用 SSH key，不要明文密码**

------

## 十一、运维高频实战场景

### 🔹 Jenkins / CI 代码发布

```bash
rsync -avz --delete ./build/ app@server:/opt/app/
```

### 🔹 日志备份

```bash
rsync -avz /var/log/ backup@bak:/log/$(date +%F)/
```

### 🔹 多台服务器分发文件

```bash
for ip in 10.0.0.{1..10}; do
  rsync -avz file.txt root@$ip:/tmp/
done
```

------

## 十二、rsync vs scp

| 对比     | rsync | scp  |
| -------- | ----- | ---- |
| 断点续传 | ✅     | ❌    |
| 增量同步 | ✅     | ❌    |
| 速度     | 快    | 一般 |
| 运维常用 | ⭐⭐⭐⭐⭐ | ⭐⭐   |

👉 **scp 能用的地方，rsync 都能更好**

