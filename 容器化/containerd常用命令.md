containerd 的主要命令行客户端是 **`ctr`** 和 **`nerdctl`**。以下是常用命令分类整理。

### 一、ctr 命令 (containerd 原生客户端)

`ctr`是 containerd 自带的客户端，功能完整但命令格式与 Docker 差异较大。

**1. 镜像管理**

- `ctr image pull docker.io/library/nginx:latest`- 拉取镜像

- `ctr image ls`- 列出本地镜像

- `ctr image tag <源镜像> <新标签>`- 给镜像打标签

- `ctr image push <镜像>`- 推送镜像到仓库

- `ctr image rm <镜像>`- 删除本地镜像

**2. 容器生命周期**

- `ctr run -d --runtime io.containerd.runc.v2 docker.io/library/nginx:latest my-nginx`- 创建并运行一个容器

- `ctr task ls`- 列出运行中的任务（容器）

- `ctr task pause <容器ID>`- 暂停容器

- `ctr task resume <容器ID>`- 恢复容器

- `ctr task kill <容器ID>`- 停止容器

- `ctr task rm <容器ID>`- 删除容器任务

- `ctr container ls`- 列出所有容器（包括未运行的）

- `ctr container rm <容器ID>`- 删除容器

**3. 命名空间管理**

- `ctr ns ls`- 列出所有命名空间

- `ctr -n <命名空间> image ls`- 在指定命名空间中操作（如 `moby`对应 Docker）

**4. 其他常用命令**

- `ctr snapshot ls`- 查看快照

- `ctr version`- 查看 containerd 版本

### 二、nerdctl 命令 (兼容 Docker CLI)

`nerdctl`是社区开发的工具，命令与 `docker`高度相似，使用更友好。

**1. 镜像管理**

- `nerdctl pull nginx:latest`- 拉取镜像

- `nerdctl images`- 列出镜像

- `nerdctl tag <源镜像> <新标签>`- 打标签

- `nerdctl push <镜像>`- 推送镜像

- `nerdctl rmi <镜像>`- 删除镜像

**2. 容器生命周期**

- `nerdctl run -d --name my-nginx -p 80:80 nginx:latest`- 创建并运行容器

- `nerdctl ps`- 查看运行中的容器

- `nerdctl ps -a`- 查看所有容器

- `nerdctl start/stop/restart <容器>`- 启动/停止/重启容器

- `nerdctl rm <容器>`- 删除容器

- `nerdctl exec -it <容器> sh`- 进入容器

**3. 构建与组合**

- `nerdctl build -t my-app .`- 构建镜像

- `nerdctl compose up -d`- 使用 Compose 启动应用栈

**4. 网络与存储**

- `nerdctl network ls`- 列出网络

- `nerdctl volume ls`- 列出卷

### 核心工具对比与建议

| 特性           | **ctr**                        | **nerdctl**                          |
| -------------- | ------------------------------ | ------------------------------------ |
| **来源**       | containerd 原生自带            | 社区项目 (CNCF)                      |
| **命令风格**   | 自有格式，与 Docker 不同       | **高度兼容 Docker CLI**              |
| **易用性**     | 较低，参数复杂                 | **高，熟悉 Docker 即可上手**         |
| **功能完整性** | 完整，但需熟悉其概念           | 完整，覆盖大部分 Docker 命令         |
| **推荐场景**   | 调试 containerd 本身，底层操作 | **日常容器管理，替代 Docker 命令行** |

**总结建议**：对于从 Docker 迁移过来的用户，**强烈推荐使用 `nerdctl`**，学习成本最低。`ctr`更适合进行底层调试或了解 containerd 核心概念。

你可以通过 `ctr --help`或 `nerdctl --help`查看更详细的子命令帮助。