crictl是Kubernetes社区官方推荐的CRI（容器运行时接口）调试工具，用于管理和调试Kubernetes节点上的容器运行时。以下是crictl常用命令大全：

## 基本配置

使用前需配置CRI运行时端点，配置文件通常为`/etc/crictl.yaml`：

```
runtime-endpoint: unix:///run/containerd/containerd.sock
image-endpoint: unix:///run/containerd/containerd.sock
timeout: 10
debug: false
```

或通过命令行指定：`crictl --runtime-endpoint unix:///run/containerd/containerd.sock ps`

## Pod管理命令

| 命令                              | 说明            | 示例                            |
| --------------------------------- | --------------- | ------------------------------- |
| `crictl pods`                     | 列出所有Pod沙箱 | `crictl pods`                   |
| `crictl pods --name <pod-name>`   | 按名称过滤Pod   | `crictl pods --name nginx-xxx`  |
| `crictl pods --label <key=value>` | 按标签过滤Pod   | `crictl pods --label run=nginx` |
| `crictl inspectp <pod-id>`        | 查看Pod详细信息 | `crictl inspectp 4dccb216c4adb` |
| `crictl runp <pod-config.json>`   | 运行新Pod       | `crictl runp pod.json`          |
| `crictl stopp <pod-id>`           | 停止Pod         | `crictl stopp 4dccb216c4adb`    |
| `crictl rmp <pod-id>`             | 删除Pod         | `crictl rmp 4dccb216c4adb`      |

## 容器管理命令

| 命令                                            | 说明                       | 示例                                    |
| ----------------------------------------------- | -------------------------- | --------------------------------------- |
| `crictl ps`                                     | 列出运行中的容器           | `crictl ps`                             |
| `crictl ps -a`                                  | 列出所有容器（包括已停止） | `crictl ps -a`                          |
| `crictl ps -s <status>`                         | 按状态过滤容器             | `crictl ps -s Running`                  |
| `crictl inspect <container-id>`                 | 查看容器详细信息           | `crictl inspect 1f73f2d81bf98`          |
| `crictl create <container-config> <pod-config>` | 创建新容器                 | `crictl create container.json pod.json` |
| `crictl start <container-id>`                   | 启动容器                   | `crictl start 3e025dd50a72d`            |
| `crictl stop <container-id>`                    | 停止容器                   | `crictl stop 1f73f2d81bf98`             |
| `crictl restart <container-id>`                 | 重启容器                   | `crictl restart 1f73f2d81bf98`          |
| `crictl rm <container-id>`                      | 删除容器                   | `crictl rm 1f73f2d81bf98`               |
| `crictl exec -it <container-id> <command>`      | 在容器中执行命令           | `crictl exec -it c8474738e4587 sh`      |
| `crictl attach <container-id>`                  | 连接到运行中的容器         | `crictl attach 1f73f2d81bf98`           |
| `crictl update <container-id>`                  | 更新容器配置               | `crictl update 1f73f2d81bf98`           |

## 镜像管理命令

| 命令                                   | 说明                 | 示例                         |
| -------------------------------------- | -------------------- | ---------------------------- |
| `crictl images`                        | 列出本地镜像         | `crictl images`              |
| `crictl images -q`                     | 仅显示镜像ID         | `crictl images -q`           |
| `crictl images <image-name>`           | 查看特定镜像         | `crictl images nginx`        |
| `crictl pull <image>`                  | 拉取镜像             | `crictl pull nginx:latest`   |
| `crictl rmi <image-id>`/<name:version> | 删除镜像             | `crictl rmi sha256:xxx`      |
| `crictl inspecti <image-id>`           | 查看镜像详细信息     | `crictl inspecti sha256:xxx` |
| `crictl imagefsinfo`                   | 查看镜像文件系统信息 | `crictl imagefsinfo`         |

## 日志与监控命令

| 命令                                  | 说明                 | 示例                                   |
| ------------------------------------- | -------------------- | -------------------------------------- |
| `crictl logs <container-id>`          | 查看容器日志         | `crictl logs 1f73f2d81bf98`            |
| `crictl logs -f <container-id>`       | 实时跟踪日志         | `crictl logs -f 1f73f2d81bf98`         |
| `crictl logs -t <container-id>`       | 显示时间戳           | `crictl logs -t 1f73f2d81bf98`         |
| `crictl logs --tail N <container-id>` | 显示最近N行日志      | `crictl logs --tail 100 1f73f2d81bf98` |
| `crictl stats`                        | 查看容器资源使用情况 | `crictl stats`                         |
| `crictl stats --id <container-id>`    | 查看单个容器资源占用 | `crictl stats --id 1f73f2d81bf98`      |
| `crictl statsp`                       | 查看Pod资源使用统计  | `crictl statsp`                        |

## 系统与连接命令

| 命令                | 说明                   | 示例                             |
| ------------------- | ---------------------- | -------------------------------- |
| `crictl version`    | 显示crictl和运行时版本 | `crictl version`                 |
| `crictl info`       | 获取运行时基本信息     | `crictl info`                    |
| `crictl config`     | 获取和设置crictl配置   | `crictl config get`              |
| `crictl completion` | 生成shell自动补全脚本  | `crictl completion bash`         |
| `crictl help`       | 显示帮助信息           | `crictl help`或 `crictl help ps` |

## 其他实用命令

- `crictl port-forward <pod-id> <local-port>:<pod-port>`：端口转发到Pod

- `crictl run <container-config> <pod-config>`：创建并运行容器（调试用）

- `crictl --help`：查看所有可用命令和选项

## 常用参数说明

- `-a`：显示所有（包括已停止的容器）

- `-q`：仅显示ID

- `-v`：显示详细信息

- `-it`：交互式终端（用于exec命令）

- `-f`：跟随模式（实时输出）

- `--name`：按名称过滤

- `--label`：按标签过滤

- `--runtime-endpoint`：指定运行时端点

这些命令覆盖了Kubernetes节点调试和容器管理的核心场景，功能与传统docker命令高度相似但专门针对CRI接口设计。