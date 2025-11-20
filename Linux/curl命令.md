# # **Linux `curl` 命令教程（超详细版）**

## 一、`curl` 是什么？

`curl` 是一款功能非常强大的 **命令行网络请求工具**，支持：

- HTTP / HTTPS
- FTP / SFTP
- Web API 调用
- 下载/上传文件
- JSON 交互
- 模拟浏览器请求
- 携带 Cookie / Token
- 代理
- 调试网络请求

相比 `wget`，`curl` 更适合用于 **API 调试与 Web 开发**。

------

# **二、基本语法**

```
curl [选项] [URL]
```

示例：

```
curl https://example.com
```

------

# **三、最常用的基本命令**

## **1. 请求 URL 内容（默认 GET）**

```
curl https://example.com
```

------

## **2. 将内容保存到文件**

```
curl -o index.html https://example.com
```

使用自动文件名：

```
curl -O https://example.com/file.zip
```

------

# **四、发送 HTTP 请求**

------

## **1. 自定义请求方法**

### GET（默认）

```
curl -X GET http://api.example.com
```

### POST

```
curl -X POST http://api.example.com/login
```

### PUT

```
curl -X PUT http://api.example.com/user/1
```

### DELETE

```
curl -X DELETE http://api.example.com/user/1
```

------

# **五、提交表单（POST）**

## **1. application/x-www-form-urlencoded**

```
curl -d "username=admin&password=123" https://example.com/login
```

等价于：

```
curl -X POST -d ...
```

------

## **2. JSON 格式提交（非常常用）**

```
curl -H "Content-Type: application/json" \
     -d '{"name":"test","age":18}' \
     https://api.example.com/user
```

------

## **3. 上传文件（multipart/form-data）**

```
curl -F "file=@/path/to/a.png" https://example.com/upload
```

也可带多个字段：

```
curl -F "file=@a.png" -F "user=admin" https://example.com/upload
```

------

# **六、设置 Header（非常常用）**

```
curl -H "Authorization: Bearer TOKEN" \
     -H "User-Agent: Mozilla/5.0" \
     https://api.example.com/data
```

------

# **七、携带 Cookie**

```
curl -b "name=value" https://example.com
```

或从文件加载：

```
curl -b cookies.txt https://example.com
curl -c cookies.txt https://example.com
```

------

# **八、使用代理**

## HTTP 代理

```
curl -x http://127.0.0.1:8080 https://example.com
```

## SOCKS5 代理

```
curl -x socks5://127.0.0.1:1080 https://example.com
```

------

# **九、HTTPS 相关参数**

## 跳过 SSL 验证（测试常用）

```
curl -k https://example.com
```

## 使用证书访问（如双向认证）

```
curl --cert client.crt --key client.key https://secure.example.com
```

------

# **十、调试网络请求（非常常用）**

## 显示请求头 + 响应头（详细）

```
curl -v https://example.com
```

## 只显示响应头

```
curl -I https://example.com
```

## 显示请求 + 响应的详细 debug

```
curl -vvv https://example.com
```

------

# **十一、限速、超时、重试**

## 限速

```
curl --limit-rate 200k https://example.com/file
```

## 设置超时

```
curl --max-time 10 https://example.com
```

## 设置重试

```
curl --retry 5 https://example.com
```

------

# **十二、REST API 常用示例（超实用）**

------

## **GET 请求带参数**

```
curl "https://api.example.com/user?id=1"
```

------

## **POST JSON**

```
curl -H "Content-Type: application/json" \
     -d '{"id":1,"name":"alex"}' \
     https://api.example.com/user
```

------

## **PUT 更新**

```
curl -X PUT -H "Content-Type: application/json" \
     -d '{"name":"newname"}' \
     https://api.example.com/user/1
```

------

## **DELETE 删除**

```
curl -X DELETE https://api.example.com/user/1
```

------

# **十三、文件上传与下载**

### 上传文件

```
curl -F "file=@/tmp/a.zip" https://upload.example.com
```

### 下载文件（保持文件名）

```
curl -O https://example.com/a.zip
```

------

# **十四、Curl 常用参数速查表（Cheat Sheet）**

| 功能          | 命令                                                   |
| ------------- | ------------------------------------------------------ |
| 普通 GET      | `curl URL`                                             |
| 保存文件      | `curl -o file URL`                                     |
| 显示 Header   | `curl -I URL`                                          |
| 显示详细请求  | `curl -v URL`                                          |
| POST 表单     | `curl -d "a=1&b=2" URL`                                |
| POST JSON     | `curl -H "Content-Type: application/json" -d '{}' URL` |
| 上传文件      | `curl -F "file=@a.txt" URL`                            |
| 设置 Header   | `curl -H "Key: Value" URL`                             |
| 使用代理      | `curl -x http://IP:PORT URL`                           |
| 跳过 SSL 验证 | `curl -k URL`                                          |
| 限速下载      | `curl --limit-rate 100k URL`                           |
| 断点续传      | `curl -C - -O URL`                                     |
| 重试          | `curl --retry 5 URL`                                   |

------

# **十五、Curl vs Wget 对比**

| 特点             | curl     | wget   |
| ---------------- | -------- | ------ |
| API 调试         | **强**   | 弱     |
| 文件下载         | 很强     | 很强   |
| 递归下载整个网站 | 无       | **有** |
| HTTP 操作        | **最强** | 一般   |
| 自动化脚本       | 常用     | 常用   |

