# # **Linux `wget` 命令教程**

## 一、`wget` 是什么？

`wget` 是 Linux 中最常用的 **文件下载工具**，支持：

- HTTP / HTTPS / FTP 下载
- 断点续传
- 后台下载
- 限速下载
- 递归下载网站
- 代理下载

非常稳定，适合批量、自动化脚本使用。

------

# **二、基本语法**

```
wget [选项] URL
```

例子：

```
wget https://example.com/file.zip
```

------

# **三、最常用的基本下载**

### 1. 下载文件

```
wget https://example.com/file.zip
```

### 2. 指定输出文件名

```
wget -O newname.zip https://example.com/file.zip
```

### 3. 后台下载

```
wget -b https://example.com/bigfile.iso
```

后台日志生成于：

```
wget-log
```

------

# **四、断点续传（非常常用）**

如果下载中断，可以续传：

```
wget -c https://example.com/bigfile.iso
```

说明：

- `-c`：continue（续传）
- 如果服务器支持 Range，续传成功

------

# **五、限速下载（避免占满带宽）**

限制下载速度，例如 200KB/s：

```
wget --limit-rate=200k https://example.com/file
```

单位支持：k、m、g

------

# **六、指定保存路径**

下载到指定目录：

```
wget -P /opt/downloads https://example.com/file
```

------

# **七、使用代理下载**

### 1. HTTP 代理：

```
wget -e "http_proxy=http://127.0.0.1:8080" https://example.com/file
```

### 2. SOCKS5 代理：

```
wget -e "https_proxy=socks5://127.0.0.1:1080" https://example.com/file
```

------

# **八、递归下载整个网站（镜像）**

非常强大！

```
wget -r -p -np -k https://example.com/
```

参数说明：

- `-r`：递归下载
- `-p`：下载页面所需资源（CSS/JS/图片等）
- `-np`：不下载父目录
- `-k`：转换链接为本地可用格式

下载后可离线浏览。

------

# **九、使用 cookie 下载**

```
wget --load-cookies cookies.txt https://example.com/file
```

导出浏览器 cookie 后，可下载需登录的文件。

------

# **十、使用用户名和密码（HTTP/FTP）**

```
wget --user=admin --password=123456 ftp://example.com/file
```

------

# **十一、模拟浏览器 UA（某些网站需要）**

```
wget --user-agent="Mozilla/5.0" https://example.com/file
```

------

# **十二、跳过证书验证（HTTPS 报错时使用）**

```
wget --no-check-certificate https://example.com/file
```

------

# **十三、下载多个 URL（批量下载）**

创建文件 `urls.txt`：

```
https://a.com/1.zip
https://a.com/2.zip
```

批量下载：

```
wget -i urls.txt
```

------

# **十四、常用参数总结表（Cheat Sheet）**

| 功能         | 命令                              |
| ------------ | --------------------------------- |
| 普通下载     | `wget URL`                        |
| 自定义文件名 | `wget -O filename URL`            |
| 断点续传     | `wget -c URL`                     |
| 后台下载     | `wget -b URL`                     |
| 限速下载     | `wget --limit-rate=200k URL`      |
| 指定目录     | `wget -P /path URL`               |
| 使用代理     | `wget -e "http_proxy=..." URL`    |
| 递归下载网站 | `wget -r -p -np -k URL`           |
| 批量下载     | `wget -i file.txt`                |
| 跳过证书检查 | `wget --no-check-certificate URL` |
| 自定义 UA    | `wget --user-agent="UA" URL`      |

------

# **十五、实战案例**

------

## **1. 下载最新版 Nginx**

```
wget http://nginx.org/download/nginx-1.25.3.tar.gz
```

## **2. 对大文件进行断点续传**

```
wget -c https://xxx.com/large.iso
```

## **3. 镜像整个站点**

```
wget -r -p -np -k https://docs.example.com/
```

