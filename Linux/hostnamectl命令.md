# **Linux `hostnamectl` 命令教程文档**

`hostnamectl` 是用于管理和查看系统主机名（hostname）及主机相关信息的命令，属于 `systemd` 工具集。

适用于常见 Linux 发行版：

- CentOS 7+
- RHEL 7+
- Ubuntu 16.04+
- Debian 8+

------

# **一、基本功能**

`hostnamectl` 主要用于：

- 查看主机信息
- 设置或修改主机名
- 修改静态、临时、或漂亮主机名（pretty hostname）
- 配置主机的 chassis、部署位置 等元数据

------

# **二、基本语法**

```
hostnamectl [选项] [命令]
```

你可以省略 `[命令]` 直接使用子命令，例如：

```
hostnamectl set-hostname myserver
```

------

# **三、查看主机信息**

## **1. 显示主机完整信息（最常用）**

```
hostnamectl
```

示例输出：

```
   Static hostname: server01
         Icon name: computer-server
           Chassis: server
        Machine ID: 23d30112f9d34c7db0b9bb...
           Boot ID: ...
  Operating System: CentOS Linux 7 (Core)
            Kernel: Linux 3.10.0-957.el7.x86_64
      Architecture: x86-64
```

------

# **四、设置主机名**

Linux 中有三种主机名类型：

| 类型                   | 说明                                       |
| ---------------------- | ------------------------------------------ |
| **Static hostname**    | 静态主机名（系统永久使用，最常用）         |
| **Pretty hostname**    | “漂亮的主机名”，可包含空格等字符           |
| **Transient hostname** | 动态主机名，当 DHCP 服务器提供时会自动改变 |

------

## **1. 设置静态主机名（最常用）**

```
hostnamectl set-hostname myserver
```

立即生效，无需重启。

------

## **2. 设置带域名的完整主机名（FQDN）**

```
hostnamectl set-hostname server01.example.com
```

------

## **3. 设置漂亮主机名（可含空格）**

```
hostnamectl set-hostname "My Office PC" --pretty
```

------

## **4. 设置临时主机名（重启后失效）**

```
hostnamectl set-hostname temp-host --transient
```

------

# **五、修改其他主机元信息**

## **1. 设置机箱类型（chassis）**

```
hostnamectl set-chassis server
```

常见类型：

| 值        | 说明   |
| --------- | ------ |
| desktop   | 桌面   |
| laptop    | 笔记本 |
| server    | 服务器 |
| vm        | 虚拟机 |
| container | 容器   |

------

## **2. 设置部署位置（location）**

```
hostnamectl set-location "IDC Room 3"
```

------

# **六、主机名修改后的注意事项**

### 1. `/etc/hostname` 会被自动更新

无需手动编辑。

### 2. `/etc/hosts` 可能需要人工更新

例如：

```
127.0.0.1   server01
```

如果不更新 `/etc/hosts`，某些软件可能无法解析主机名。

------

# **七、与旧命令的对比**

以前使用：

```
hostname
```

- 可查看主机名
- 也能设置，但不可修改 `/etc/hostname` 文件

而 `hostnamectl`：

- 系统化管理
- 兼容 systemd
- 可设置 pretty/transient 等不同主机名信息

------

# **八、常用命令速查表（推荐收藏）**

| 命令                                              | 说明           |
| ------------------------------------------------- | -------------- |
| `hostnamectl`                                     | 查看主机信息   |
| `hostnamectl set-hostname server01`               | 设置静态主机名 |
| `hostnamectl set-hostname server01.example.com`   | 设置 FQDN      |
| `hostnamectl set-hostname "My Linux PC" --pretty` | 设置漂亮主机名 |
| `hostnamectl set-hostname test --transient`       | 设置临时主机名 |
| `hostnamectl set-chassis server`                  | 设置机箱类型   |
| `hostnamectl set-location "Room-01"`              | 设置部署位置   |

