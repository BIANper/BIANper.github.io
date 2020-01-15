---
title: Linux bash
date: 2019-11-06 17:23:39
tags:
- bash
- 数据流
categories: Linux
---
# shell

用户默认取得的 shell 记录于 /etc/passwd，默认是 bash
上一次登录的历史命令记录在 ~/.bash_histroy，本次登录的会在注销后记录进去
内置命令和外部命令
```
# type [-ta] name
-t 仅显示执行时的依据
-a 显示所有含 name 的命令
```
内置命令是 shell 解释程序内建的，由 shell 直接执行，不需要派生新的进程；外部命令 shel会创建一个新的进程，新的进程在 PATH 变量内所列出的目录中寻找特定命令执行，默认 shell 将等待直到该进程结束

# 变量
env 查看当前 shell 环境下的环境变量和内容
set 查看所有变量，包含环境变量和自定义变量

**设置规则**
不能以数字开头，连接的 = 两边不能直接接空格，变量内容若有空格使用 " 或 '，必须成对
- " 可以保持特殊字符的特性，如 var=lang is $LANG 相当于 lang is en_US
    ' 仅为纯文本
- \`命令\` 和 $(命令) 用于在命令中使用其他命令的值，如 echo $version
    $变量名称 ${变量名称} 用于显示变量或增加变量内容，如 PATH="$PATH":/home/bin
    
转义字符 / 将特殊符号变成一般符号，命令过长可以 /[enter] 换行
若变量需要在子进程进行需要用 export 变成环境变量，不加参数时和 env 相似
unset 取消变量

**语系问题**
locale 不加参数查询所有语系变量
可以逐一设置，当设置 LANG 或者 LC_ALL 时，其他语系变量都会被这两个替代
locale -a 查询所有支持的语系，语系文件放置在 /usr/lib/locale/
整体系统默认的语系定义在 /etc/sysconfig/i18h

**键盘输入**
读取来自键盘输入的变量
```
# read [-pt] variable
-p 后面可接提示符
-t 后面可接等待秒数
```

**声明变量类型**
declare 或 typeset 功能一样。变量类型默认为字符串，数值运算默认最多达到整数型
数组变量类型，var[index] = content 设置，${数组}读取
```
# declare [-aixrp] variable
-a 定义成数组 array 类型
-i 定义成整数 integer 类型
-x 与 export 一样，变成环境变量
-r 设置成只读 readonly 类型，不可被变更内容也不能重设
-p 查看变量类型
不接任何参数则显示所有变量与内容，类似 set
将 - 换成 + 可进行取消操作

# sum=1+2+3    <--x
# declare -i sum=1+2+3    <--√
# echo $sum
```

**变量内容的删除和替换**
从变量内容向右删除，取最短的那个
```
# ${variable#}
```
从变量内容向右删除，取最长的那个
```
# ${variable##}
```
从变量内容向左删除，取最短的那个，最长同上
```
# ${variable%}
```

|   设置   |   说明   |
| :--: | :--: |
|   ${变量#关键字}   |   变量内容从头开始的内容符合关键字，则删除符合的最短数据   |
|   ${变量##关键字}   |   变量内容从头开始的内容符合关键字，则删除符合的最长数据   |
|   ${变量%关键字}   |   变量内容从后向前的内容符合关键字，则删除符合的最短数据   |
|   ${变量%%关键字}   |   变量内容从后向前的内容符合关键字，则删除符合的最短数据   |
|   ${变量/旧字符串/新字符串}   |   变量内容符合旧字符串，则替换第一个旧字符串   |
|   ${变量//旧字符串/新字符串}   |   变量内容符合旧字符串，则替换全部的旧字符串   |

**变量的测试和内容替换**
判断变量是否存在，不存在便给予设置
```
# new_var=${old_var-content}
```
判断变量是否存在或为空字符串，不存在或为空便给予设置
```
# new_var=${old_var:-content}
```

|   设置方式   |   str 没有设置   |   str 为空字符串   |   str 已设置非空   |
| :--: | :--: | :--: | :--: |
|   var=${str-expr}   |   var=exper   |   var=   |   var=$str   |
|   var=${str:-expr}   |   var=exper   |   var=expr   |   var=$str   |
|   var=${str+expr}   |   var=   |   var=expr   |   var=expr   |
|   var=${str:+expr}   |   var=   |   var=   |   var=expr   |
|   var=${str=expr}   |   var=expr str=expr   |   var=  str 不变|   var=$str str 不变   |
|   var=${str:=expr}   |   var=expr str=expr   |   var=expr str=expr   |   var=$str str不变   |
|   var=${str?expr}   |   expr 输出至 stderr   |   var=   |   var=str   |
|   var=${str:?expr}   |   expr 输出至 stderr   |   expr 输出至 stderr   |   var=str   |

# 命令
**命令别名设置**
alias 单用查询当前命令别名
```
# alias rm='rm -i'
```
unalias 取消命令别名

**历史命令**
登录主机后，系统由 ~./bash_history 读取历史命令，记录的条数由 HISTSIZE 变量设定，注销时会将最近的 HISTSIZE 条记录到记录文件中
```
# history [n]
# history [-c]
# history [-raw] histfiles
n 最近 n 条命令
-c 清除目前 shell 中的 history
-a 将新增的 history 更新进 histfiles，若没有则写入默认文件中
-r 将 histfiles 的内容读到目前 shell 的 history 中
-w 将目前 history 更新进 histfiles
```
```
# !number
# !command
# !!
number 执行第 number 条指令
command 由近向远搜寻 command 开头的命令并执行
!! 执行上一条命令
```

# 操作环境
**命令运行的顺序**
- 相对/绝对路径执行
- 由 alias 找到执行
- bash 内置命令
- $PATH 变量找到的第一个

**bash 的登录欢迎信息**
登录后的欢迎字符串在 /etc/issue 中设置
telnet 远程登录用的是 /etc/issue.net

|   -   |   -   |
| :--: | :--: |
|   \d   |   本地日期   |
|   \l   |   第几个终端机接口   |
|   \m   |   硬件等级   |
|   \n   |   主机的网络名称   |
|   \0   |   domain name   |
|   \r   |   操作系统版本 (uname -r)   |
|   \t   |   本地时间   |
|   \s   |   操作系统名称   |
|   \v   |   操作系统版本   |

每个用户登录后都显示的信息加入 /etc/moted 中

**bash 的环境配置**
login shell ：完整登录流程取得的 bash
non-login shell ：没有重复登录取得的 bash，如 X Window 登录后图形界面启动终端机，没有再次要求输入密码

读入环境配置文件
source 或小数点 . 都可以将配置文件读进当前 shell 环境而不必注销
```
# source 配置文件名
```

login shell 读取的文件：
1. /etc/profile 系统整体设置，会调用外部数据
 - /etc/inputrc
 - /etc/profile.d/*.sh
 - /etc/sysconfig/i18n
2. ~/.bash_profile 或 ~/.bash_login 或 /.profile 用户个人设置，仅读取依此顺序第一个找到的，照顾其他 shell 的用户

non-login shell 仅读取 ~.bashrc

**终端机环境设置**
查阅按键内容
```
# stty [-a]
-a 列出目前所有的按键和按键内容
^ 代表 [ctrl]
```
设置按键
```
# stty 行为 按键
```
set 可设置输出/输入环境

**通配符**
\* 零到无穷个字符；? 一定有一个字符；[] 一定有一个在其中的字符；[-] 编码顺序内的所有字符；[^] 非

# 数据流重定向
将某个命令执行后的输出传输到其他地方
标准输入 stdin ：代码 0，用 < <<
标准输出 stdout ：代码 1，用 > >>
标准错误输出 stderr ：代码 2，用 2> 2>>

**将数据输出到指定文件或设备**
\> 输出到已存在文件时，会先清空再写入
\>> 会写入文件的最下方
垃圾桶黑洞设备（丢弃错误信息），/dev/null 可以吃掉任何导向该设备的信息
```
$ find /home -name .bashrc 2> /dev/null
```
将正确与错误信息写入到同一文件，使用 & 语法，否则会交叉写入
```
$ find /home -name .bashrc > list 2>&1
```
cat 命令后加入 >，会主动创建文件，并且其内容即键盘输入的数据
```
# cat > catfile
testing
<==[ctrl]+d 离开
```
**用文件内容代替键盘输入**
\< 指定文件内容替代键盘输入
\<< 代表输入结束，键盘输入指定字符便结束输入
```
# cat > catfile < ～/.bashrc
```

# 命令的判断依据
**;** 一次执行多条命令，用 ; 隔开
**$?** 命令回传码，若前一个命令执行的结果为正确，将回传 $?=0
**&&** $?=0 才执行下一句
**||** $?≠0 才执行下一句
cmd1 && cmd2 || cmd3 是安全的，其余顺序要注意有效性和安全性

# 管道命令 pipe
使用 | 符号，仅处理由前一个命令传来的正确信息，并且接收数据成为 stdin 继续处理
- less more head 是可以接收 stdin 的管道命令
- ls cp mv 则不是

### 选取命令
cut 取出一段信息的某一段，常用于将同一行的数据进行分解
```
# cut -d '分隔字符' -f fields
# cut -c 字符范围
-d -f 依据分隔字符将信息切成数段，取其中 fields 段
-c 以字符的单位取出固定字符区间

echo $PATH | cut -d ':' -f 3,5
export | cut -c 12-
```
grep 对行信息进行分析
```
# grep [-acinv] [--color=auto] '查找字符串' filename
-a 将 binary 文件以 text 文件的方式查找
-c 计算找到 '' 的次数
-i 忽略大小写
-n 输出行号
-v 反选

last | grep 'root' | cut -d ' ' -f 1
```

### 排序命令
```
# sort [-fbMnrtuk] [file or stdin]
-f 忽略大小写
-b 忽略最前面的空格符
-M 以月份排序
-n 使用纯数字排序，默认是文字类型
-r 反向排序
-u 相同数据仅显示一条
-t 设置分隔符，默认 [TAB]
-k 以区间（field）进行排序

cat /etc/passwd | sort -t ':' -k 3 -n
```
```
# uniq [-ic]
-i 忽略大小写
-c 进行计数

last | cut -d ' ' -f 1 | sort | uniq -c
```
```
# wc [-lwm]
-l 仅列出行
-w 仅列出多少英文单字
-m 仅显示多少字符
默认显示 行数 字数 字符数
```

