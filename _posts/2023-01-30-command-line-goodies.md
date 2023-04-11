---
layout: post
title: 'Command-line goodies'
subtitle: ''
author: 'Will'
header-style: text
lang: en
tags:
  - Bash
  - Tricks
  - 编程语言
  - 笔记
---

我整理了一些小而精的命令行工具，在编写 Bash 脚本的时候可能有奇效。

> 本文部分代码由 AI 生成

![](https://i.imgur.com/bSBObMu.png)

#### xargs

将标准输入（stdin）中的数据转换成命令行参数

```bash
# 删除包含特定文本的所有文件
grep -rl "text" . | xargs rm

# 删除所有空文件和空目录
find . -type f -empty -print0 | xargs -0 rm
find . -type d -empty -print0 | xargs -0 rmdir

# 删除所有名称匹配的文件
find . -name "*.txt" -print0 | xargs -0 rm

# 复制所有名称匹配的文件
find . -name "*.txt" -print0 | xargs -0 cp -t /path/to/destination

# 移动所有名称匹配的文件
find . -name "*.txt" -print0 | xargs -0 mv -t /path/to/destination

# 在所有文件中查找特定文本并替换
find . -name "*.txt" -print0 | xargs -0 -I {} sed -i 's/text/replacement/g' {}

# 将多个文件合并为一个文件
find . -name "*.txt" -print0 | xargs -0 -I {} cat {} >> all_files.txt

# 批量更改文件扩展名
ls *.old | xargs -n 1 -I {} mv {} {}.new

# 批量压缩文件
find . -name "*.txt" -print0 | xargs -0 -n 1 gzip

# 在所有文件上运行命令
find . -name "*.txt" -print0 | xargs -0 grep "text"

# 并行运行命令
find . -name "*.txt" -print0 | xargs -0 -P 4 grep "text"

# 并行运行命令，每次处理 4 个文件
find . -name "*.txt" -print0 | xargs -0 -n 4 -P 4 grep "text"
```

#### ps

列出系统中当前运行的进程信息

```bash
# 按 CPU 利用率排序显示前 10 个进程
ps -aux --sort=-%cpu | head -n 11

# 按内存利用率排序显示前 10 个进程
ps -aux --sort=-%mem | head -n 11

# 按 PID 显示进程树
ps -ef --forest
```

#### pgrep

查找正在运行的进程，并返回它们的 PID

```bash
# 查找名为 "myprocess" 的进程并杀死它们
pgrep myprocess | xargs kill

# 检查名为 "myprocess" 的进程是否正在运行
if pgrep myprocess > /dev/null; then
    echo "myprocess is running"
else
    echo "myprocess is not running"
fi

# 查找属于用户 "myuser" 的进程
pgrep -u myuser
```

#### free

显示系统内存的使用情况，包括物理内存、交换内存和内核缓冲区的使用情况

```bash
# 显示系统内存的使用情况，以人类可读的格式输出
free -h
```

#### df

显示文件系统磁盘空间利用率，包括总容量、已用空间、可用空间和文件系统的挂载点

```bash
# 显示文件系统的剩余空间，以人类可读的格式输出
df -h

# 显示指定文件系统的剩余空间，以人类可读的格式输出
df -h /dev/sda1
```

#### du

递归地遍历指定目录下的所有文件和子目录，并计算它们的大小

```bash
# 按目录大小从大到小排序
du -h --max-depth=1 | sort -hr

# 按目录大小从小到大排序
du -h --max-depth=1 | sort -h
```

#### curl

HTTP 请求

```bash
# -f 错误时终止
# -s 静默模式，不显示进度条和报错
# -S 错误时报错
# -L 跟随重定向
# -o 保存输出到文件
curl -fsSL https://code-server.dev/install.sh | sh
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
curl -LSso ~/.vim/autoload/pathogen.vim https://tpo.pe/pathogen.vim
```

#### wget

从 Web 上下载文件

```bash
# 下载文件并保存到当前目录
wget <URL>

# 下载文件并保存到指定目录
wget -P <directory> <URL>

# 下载文件并重命名
wget -O <filename> <URL>
```

#### tee

[sudo echo "something" >> /etc/privilegedFile doesn't work [duplicate]](https://stackoverflow.com/a/550808)

```bash
curl -fsSL https://foo/bar/baz.key | gpg --dearmor | sudo tee /etc/apt/trusted.gpg.d/baz.gpg > /dev/null
```

#### tr

替换或删除字符

```bash
# 将所有的小写字母转换为大写字母
echo "hello, world!" | tr '[:lower:]' '[:upper:]'

# 删除文本中的所有数字
echo "abc123def456" | tr -d '[:digit:]'

# 将逗号分隔的文本转换为换行符分隔的文本
echo "a,b,c,d" | tr ',' '\n'

# 将文件中的所有制表符转换为空格
tr '\t' ' ' < input.txt > output.txt

# 将多个空格压缩为一个空格
echo "a  b      c" | tr -s ' '
```

#### tail

```bash
# 持续输出文件的更新
tail -f file.log

# 显示文件的最后10行并持续输出更新
tail -f file.log | tail -n 10
```

#### shuf

打乱

```bash
# 随机选择文件中的一行
shuf -n 1 file.txt

# 随机选择文件中的多行
shuf -n 5 file.txt

# 将文件中的所有行随机排列
shuf file.txt
```

#### uniq

```bash
# 从文件中删除重复行并将其输出到新文件
uniq input.txt > output.txt

# 将重复行计数并输出到屏幕
uniq -c input.txt

# 仅显示重复行
uniq -d input.txt

# 仅显示不重复的行
uniq -u input.txt
```

#### sort

排序，默认按字母表顺序

```bash
# 逆序
sort -r file.txt

# 忽略大小写
sort -f file.txt

# 按数字大小
sort -n file.txt

# 按第二列
sort -k2 file.txt

# 以逗号为分隔符按第二列
sort -t, -k2 file.csv
```

#### cut

提取字符串

```bash
# 提取字符串的第一个字符
first_char=$(echo "$string" | cut -c1)

# 提取字符串的前 N 个字符
first_n_chars=$(echo "$string" | cut -c1-$n)

# 提取字符串的后 N 个字符
last_n_chars=$(echo "$string" | rev | cut -c1-$n | rev)

# 提取字符串的第 N 个字符到第 M 个字符
substring=$(echo "$string" | cut -c$n-$m)
```

提取数据列

```bash
# 提取系统所有 USERNAME
cut -d: -f1 < /etc/passwd

# 提取 CSV 文件的第 N 列数据
cut -d',' -f$n "$filename.csv"
```

#### command

[What is the bash command: `command`?](https://askubuntu.com/a/538818)

```bash
# 使用 command -v 命令检查命令是否存在
if command -v command_name > /dev/null; then
    echo "Command exists"
else
    echo "Command does not exist"
fi

# 使用 command -p 命令查找命令的路径
echo "The path of ls command is: $(command -p ls)"

# 在脚本中使用 command 命令来调用命令，而不受别名或函数的影响
# 别名
alias ls='ls -la'
# 函数
function ls() {
    command ls -lh
}
# 调用原始的 ls 命令
command ls
```

#### printf

> If there are more strings passed to `printf` than a format string can
> handle, that format string will be repeated.

```bash
# 打印字符串
printf "Hello, World!\n"

# 格式化输出
printf "My name is %s and I am %d years old.\n" "Alice" 25

# 对齐文本
printf "| %-10s | %-10s |\n" "Name" "Age"
printf "| %-10s | %-10s |\n" "Alice" "25"
printf "| %-10s | %-10s |\n" "Bob" "30"

# 红色字体
printf "\033[0;31mError: something went wrong.\033[0m\n"

# 绿色背景
printf "\033[42mSuccess: task completed.\033[0m\n"

# 输出脚本所有参数
printf "%s\n" "$@"
```
