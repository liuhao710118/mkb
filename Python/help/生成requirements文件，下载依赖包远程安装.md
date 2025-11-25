方法 1：使用 `pip download` 下载依赖包

1. **创建依赖文件**：

   确保你的脚本有一个 `requirements.txt` 文件，列出所有依赖的第三方库。可以使用以下命令生成：

   ```bash
   复制代码
   pip freeze > requirements.txt
   ```
   
2. **下载依赖包**：

   在有网络的机器上，运行以下命令下载所有依赖包：

   ```bash
   复制代码
   pip download -r requirements.txt -d ./packages
   ```
   
   这会将依赖包下载到 `./packages` 目录下。
   
3. **复制到目标环境**：

   将 `requirements.txt` 和 `packages` 目录复制到内网环境。

4. **安装依赖包**：

   在内网环境中，使用以下命令安装依赖：

   ```bash
     复制代码
    pip install --no-index --find-links=./packages -r requirements.txt
   ```
   
   
