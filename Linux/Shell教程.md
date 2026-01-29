参考文档：

1.https://blog.csdn.net/weixin_42313749/article/details/120524768



# 🐚 Shell 脚本入门与实战教程


## 第 1 章：Shell 基础

### 1.1 什么是 Shell

Shell 是 Linux/Unix 系统的命令解释器（Command Interpreter），
 它连接用户与操作系统内核，既能交互执行命令，也能批量执行脚本。

常见 Shell 类型：

| Shell | 可执行文件路径 | 说明                        |
| ----- | -------------- | --------------------------- |
| bash  | /bin/bash      | 最常用，功能丰富            |
| sh    | /bin/sh        | 兼容 POSIX 标准，常用于脚本 |
| zsh   | /bin/zsh       | 功能增强型 Shell            |
| ksh   | /bin/ksh       | Korn Shell，商用系统常见    |

------

### 1.2 第一个 Shell 脚本

新建文件 `hello.sh`：

```bash
#!/bin/bash
echo "Hello, Shell!"
```

执行方式：

```bash
# 方式1：直接执行
bash hello.sh

# 方式2：添加执行权限
chmod +x hello.sh
./hello.sh
```

输出：

```
Hello, Shell!
```

------

### 1.3 Shebang（#!）

脚本开头的 `#!/bin/bash` 指定该脚本由哪种解释器执行。

例如：

```bash
#!/usr/bin/env bash
```

这种写法更通用，会自动在环境变量 PATH 中寻找 bash。

------

### 1.4 注释与可执行权限

- 单行注释：以 `#` 开头
- 多行注释（建议使用 `:<<EOF ... EOF` 形式）：

```bash
:<<EOF
这是多行注释示例
可以包含多行内容
EOF
```

设置执行权限：

```bash
chmod +x script.sh
```

------

## 第 2 章：变量与数据类型

### 脚本变量

```shell
# Shell常见的变量之一系统变量，主要是用于对参数判断和命令返回值判断时使用，系统变量详解如下：

$0 		当前脚本的名称；
$n 		当前脚本的第n个参数,n=1,2,…9；
$* 		当前脚本的所有参数(不包括程序本身)；
$# 		当前脚本的参数个数(不包括程序本身)；
$? 		命令或程序执行完后的状态，返回0表示执行成功；
$$ 		程序本身的PID号。
$?      最后一个后台进程的PID
```



### 2.1 定义变量

```bash
name="Tom"
age=20
```

变量名规则：

- 只能包含字母、数字、下划线
- 不能以数字开头
- 赋值时 `=` 前后**不能有空格**

引用变量：

```bash
echo "Name: $name"
echo "Age: ${age}"
```

------

### 2.2 环境变量与局部变量

设置环境变量（仅当前 shell 有效）：

```bash
export PATH=$PATH:/usr/local/bin
```

查看变量：

```bash
echo $PATH
```

删除变量：

```bash
unset name
```

------

### 2.3 只读变量

```bash
readonly company="OpenAI"
company="Google"   # 报错：只读变量
```

------

### 2.4 字符串操作

#### 拼接字符串

```bash
str1="Hello"
str2="World"
str3=$str1" "$str2
echo $str3
```

#### 获取字符串长度

```bash
str="abcdef"
echo ${#str}    # 输出 6
```

#### 子串截取

```bash
text="ShellTutorial"
echo ${text:0:5}   # Shell
echo ${text:5:4}   # Tuto
```

#### 替换子串

```bash
str="I like banana"
echo ${str/banana/apple}
```

------

### 2.5 数值运算

#### 方法 1：$((...))

```bash
a=10
b=3
echo $((a+b))   # 13
echo $((a*b))   # 30
```

#### 方法 2：`expr`

```bash
a=5
b=2
expr $a + $b
```

#### 方法 3：浮点计算（bc）

```bash
echo "scale=2; 10/3" | bc
```

------

## 第 3 章：输入与输出

### 3.1 输出命令

```bash
echo "Hello World"
printf "Name: %s\nAge: %d\n" "Tom" 18
```

#### 常用转义字符

| 转义符 | 含义       |
| ------ | ---------- |
| `\n`   | 换行       |
| `\t`   | 制表符     |
| `\\`   | 反斜线本身 |
| `\"`   | 双引号     |

------

### 3.2 读取输入

```bash
read -p "请输入你的名字: " name
echo "你好, $name!"
```

#### 常用参数

| 参数 | 说明               |
| ---- | ------------------ |
| `-p` | 提示文字           |
| `-s` | 隐藏输入（如密码） |
| `-t` | 设置超时（秒）     |

示例：

```bash
read -s -p "请输入密码：" pwd
echo
echo "密码长度：${#pwd}"
```

------

### 3.3 重定向输出

```bash
echo "写入文件" > output.txt      # 覆盖写入
echo "追加内容" >> output.txt     # 追加写入
```

错误输出重定向：

```bash
ls /not/exist 2> error.log
```

同时输出到标准输出和文件：

```bash
echo "日志内容" | tee log.txt
```

------

### 3.4 管道（Pipe）

管道 `|` 把前一个命令的输出传给下一个命令。

示例：

```bash
cat /etc/passwd | grep root | wc -l
```

意思是：

1. 输出 `/etc/passwd` 内容
2. 查找包含 “root” 的行
3. 统计行数

------

## 第 4 章：条件判断与流程控制

### 注意：推荐使用双[]替代单[]

> **能用 `[[ ]]` 就别用 `[ ]`**（写 Bash 脚本时） 双 **`[]`** 是单 **`[]`** 的扩展版 只有当兼容POSIX 脚本时才用单 **`[]`** 

[bash双[]教程](./concept/bash双[]条件判断完整教程.md)

### 4.1 if 判断基础语法

```bash
if [ 条件 ]; then
    命令
fi
```

或多行写法：

```bash
if [ 条件 ]
then
    命令
fi
```

------

### 4.2 if-else

```bash
if [ $a -gt 10 ]; then
    echo "大于 10"
else
    echo "小于等于 10"
fi
```

------

### 4.3 if-elif-else

```bash
if [ $score -ge 90 ]; then
    echo "优秀"
elif [ $score -ge 60 ]; then
    echo "及格"
else
    echo "不及格"
fi
```

------

### 4.4 常用条件运算符

#### 数值比较

| 运算符 | 含义     |
| ------ | -------- |
| `-eq`  | 等于     |
| `-ne`  | 不等于   |
| `-gt`  | 大于     |
| `-lt`  | 小于     |
| `-ge`  | 大于等于 |
| `-le`  | 小于等于 |

#### 字符串比较

| 运算符   | 含义       |
| -------- | ---------- |
| `=`      | 相等       |
| `!=`     | 不相等     |
| `-z str` | 字符串为空 |
| `-n str` | 字符串非空 |

------

### 4.5 文件测试运算符

| 运算符    | 含义           |
| --------- | -------------- |
| `-e file` | 文件是否存在   |
| `-f file` | 是否为普通文件 |
| `-d file` | 是否为目录     |
| `-r file` | 是否可读       |
| `-w file` | 是否可写       |
| `-x file` | 是否可执行     |

示例：

```bash
file="/etc/passwd"
if [ -f "$file" ]; then
    echo "$file 存在"
else
    echo "$file 不存在"
fi
```

------

### 4.6 多条件判断

```bash
if [ $a -gt 0 ] && [ $a -lt 10 ]; then
    echo "a 在 0~10 之间"
fi

if [ $b -eq 1 ] || [ $b -eq 2 ]; then
    echo "b 是 1 或 2"
fi
```

------

### 4.7 命令执行结果判断

```bash
if ping -c 1 8.8.8.8 > /dev/null 2>&1; then
    echo "网络连通"
else
    echo "网络不通"
fi
```

------

### 4.8 case 多分支结构

```bash
read -p "请输入一个字符: " char
case $char in
    [a-z])
        echo "小写字母";;
    [A-Z])
        echo "大写字母";;
    [0-9])
        echo "数字";;
    *)
        echo "其他字符";;
esac
```

### 4.9 for 循环

遍历

```shell
for item in a b c; do
    echo "Item: $item"
done
```

输出

```
Item: a
Item: b
Item: c
```

C语言 风格循环

```shell
for ((i=1; i<=5; i++)); do
    echo "i = $i"
done
```

遍历数组

```shell
arr=(apple banana orange)
for fruit in "${arr[@]}"; do
    echo "Fruit: $fruit"
done
```

### 4.10 while 循环

基本语法

```shell
count=1
while [ $count -le 5 ]; do
    echo "Count: $count"
    ((count++))
done
```

当条件为真时循环执行，否则退出。



文件内容遍历

```shell
while IFS= read -r line; do
    echo "Line: $line"
done < file.txt
```

## 第 5 章：函数与作用域

### 5.1 函数定义与调用

Shell 中函数定义方式：

```bash
# 方法1
function my_func {
    echo "Hello from my_func"
}

# 方法2
my_func2() {
    echo "Hello from my_func2"
}

# 调用函数
my_func
my_func2
```

------

### 5.2 函数参数

函数参数通过 `$1`、`$2`、`$@`、`$#` 获取：

```bash
greet() {
    echo "Hello, $1!"
    echo "Total parameters: $#"
}

greet Tom
```

输出：

```
Hello, Tom!
Total parameters: 1
```

- `$1`, `$2` ... 表示第 1、第 2 个参数
- `$@` 表示所有参数
- `$#` 表示参数个数

------

### 5.3 函数返回值

Shell 函数可以通过 `return` 返回整数状态码（0-255）：

```bash
is_even() {
    if [ $(($1 % 2)) -eq 0 ]; then
        return 0
    else
        return 1
    fi
}

is_even 4
echo $?  # 输出 0，表示偶数
```

> 注意：如果想返回字符串或其他数据，请用 `echo` 输出，再用命令替换捕获：

```bash
get_name() {
    echo "Tom"
}

name=$(get_name)
echo $name
```

------

### 5.4 变量作用域

默认变量为全局，使用 `local` 可声明局部变量：

```bash
foo() {
    local x=10
    echo "x in function: $x"
}

foo
echo "x outside function: $x"  # 输出空
```

------

## 第 6 章：数组与字符串处理

### 6.1 数组定义与访问

```bash
# 定义数组
fruits=("apple" "banana" "orange")

# 访问元素
echo ${fruits[0]}   # apple

# 数组长度
echo ${#fruits[@]}  # 3

# 遍历数组
for fruit in "${fruits[@]}"; do
    echo $fruit
done
```

------

### 6.2 字符串处理

#### 1. 长度

```bash
str="Hello Shell"
echo ${#str}  # 11
```

#### 2. 子串截取

```bash
echo ${str:6:5}  # Shell
```

#### 3. 替换

```bash
echo ${str/Hello/Hi}   # Hi Shell
echo ${str//l/X}       # HeXXo SheXX
```

#### 4. 分割字符串

```bash
IFS=" " read -r -a arr <<< "$str"
echo ${arr[0]}   # Hello
echo ${arr[1]}   # Shell
```

------

### 6.3 高级模式匹配

```bash
file="report.txt"

# 删除后缀
echo ${file%.txt}  # report

# 删除前缀
echo ${file#*.}    # txt
```

------

## 第 7 章：文件与目录操作

### 7.1 文件与目录判断

```bash
file="test.txt"
dir="mydir"

if [ -f "$file" ]; then
    echo "$file 是普通文件"
fi

if [ -d "$dir" ]; then
    echo "$dir 是目录"
fi
```

------

### 7.2 创建和删除

```bash
# 创建文件和目录
touch newfile.txt
mkdir newdir

# 删除文件和目录
rm newfile.txt
rmdir newdir
```

> ⚠️ 注意：`rm -rf` 可以删除非空目录，请谨慎使用。

------

### 7.3 遍历目录

```bash
for file in /etc/*.conf; do
    echo "配置文件: $file"
done
```

使用 `find` 查找文件：

```bash
find /var/log -type f -name "*.log"
```

------

### 7.4 读取文件内容

```bash
while IFS= read -r line; do
    echo "内容: $line"
done < /etc/hosts
```

### 7.5 文件复制与移动

```bash
cp file1.txt backup/
mv file2.txt newname.txt
```

------

### 7.6 文件权限操作

```bash
# 查看权限
ls -l file.txt

# 修改权限
chmod 755 script.sh
chmod u+x script.sh

# 修改所有者
chown user:group file.txt
```

------

### 7.7 文件大小与空文件判断

```bash
file="log.txt"
if [ -s "$file" ]; then
    echo "$file 非空"
else
    echo "$file 空文件或不存在"
fi
```

## 第 8 章：进程与信号

### 8.1 启动后台进程

```bash
# 后台运行
sleep 60 &

# 获取最后一个后台进程的 PID
pid=$!
echo "后台进程 PID: $pid"
```

------

### 8.2 获取进程 PID

#### 方法 1：`$!` 获取最近后台进程

```bash
./myscript.sh &
echo $!   # 输出 PID
```

#### 方法 2：`pgrep` 根据名称查找

```bash
pid=$(pgrep -f "myscript.sh")
echo "PID: $pid"
```

#### 方法 3：`pidof`

```bash
pid=$(pidof bash)
echo "bash PID: $pid"
```

------

### 8.3 停止进程

```bash
kill $pid       # 正常终止
kill -9 $pid    # 强制终止
```

------

### 8.4 trap 捕获信号

```bash
#!/bin/bash
trap "echo 捕获到 SIGINT, 退出程序; exit" SIGINT

while true; do
    echo "按 Ctrl+C 尝试中断"
    sleep 2
done
```

------

### 8.5 nohup 守护进程

```bash
nohup ./myscript.sh > mylog.log 2>&1 &
pid=$!
echo "守护进程 PID: $pid"
```

------

## 第 9 章：实战示例

### 9.1 定时任务示例

```bash
#!/bin/bash
# backup.sh: 自动备份目录

src="/home/user/data"
dest="/home/user/backup/$(date +%Y%m%d)"
mkdir -p "$dest"
cp -r "$src"/* "$dest"
echo "备份完成: $dest"
```

> 可通过 `crontab -e` 添加定时任务：每天 2 点备份
>  `0 2 * * * /home/user/backup.sh`

------

### 9.2 检测服务是否运行

```bash
#!/bin/bash
service_name="nginx"

if pgrep -x "$service_name" > /dev/null; then
    echo "$service_name 正在运行"
else
    echo "$service_name 未运行"
fi
```

------

### 9.3 日志清理脚本

```bash
#!/bin/bash
log_dir="/var/log/myapp"
find "$log_dir" -type f -name "*.log" -mtime +7 -exec rm {} \;
echo "7天前日志已清理"
```

------

### 9.4 多进程监控

```bash
#!/bin/bash
services=("nginx" "mysql" "redis")

for s in "${services[@]}"; do
    if pgrep -x "$s" > /dev/null; then
        echo "$s 正常"
    else
        echo "$s 未运行, 启动中..."
        systemctl start "$s"
    fi
done
```

------

### 9.5 脚本参数处理

```bash
#!/bin/bash
if [ $# -lt 1 ]; then
    echo "Usage: $0 filename"
    exit 1
fi

file=$1
if [ -f "$file" ]; then
    echo "$file 存在"
else
    echo "$file 不存在"
fi
```

------

## 第 10 章：调试与优化

### 10.1 输出调试信息

```bash
#!/bin/bash
set -x   # 打开调试模式
a=5
b=0
echo $((a/b))   # 会显示每条命令的执行过程
set +x   # 关闭调试模式
```

------

### 10.2 错误捕获与退出码

```bash
#!/bin/bash
set -e  # 遇到错误立即退出

mkdir /tmp/mytest
cd /tmp/mytest
cp /notexist/file ./   # 文件不存在，脚本立即退出
```

查看上条命令退出码：

```bash
echo $?
```

------

### 10.3 提高可读性

- 使用缩进与空行分块
- 注释说明每段逻辑
- 使用函数封装重复操作
- 统一变量命名规范（小写或驼峰）

------

### 10.4 常见陷阱

1. **空格问题**

```bash
a=10   # 正确
a = 10 # 错误
```

1. **引号问题**

```bash
name="Tom"
echo $name    # 可以
echo "$name"  # 推荐，防止空格或特殊字符
```

1. **变量展开**

```bash
file="/tmp/test file.txt"
rm $file      # 错误，空格会被分割
rm "$file"    # 正确
```

1. **命令替换**

```bash
count=$(ls | wc -l)
echo "文件数量: $count"
```

------

