---
title: Linux 正则和文件格式化
date: 2019-11-28 21:22:53
tags:
- 正则表达式
- 文件
categories: Linux
---
正则表达式即以行为单位，通过特殊符号查询删除替换某特定字符串的方法，在支持的工具里通用
# 基础正则表达式
**语系**
避免编码造成的区别，建议使用特殊符号

|  特殊符号  |  意义  |
| :---: | :---: |
|  :alnum:  |  A-Z a-z 0-9  |
|  :alpha:  |  A-Z a-z  |
|  :upper:  |  A-Z  |
|  :lower:  |  a-z  |
|  :digit:  |  0-9  |

**字符**
|  RE  |  意义  |
| :---: | :---: |
|  ˆword  |  待查找字符串在首行  |
|  word$  |  待查找字符串在尾行  |
|  .  |  一定有一个任意字符  |
|  \  |  转义  |
|  *  |  重复的零到无穷个前一个字符  |
|  [list]  |  list 中任意一个  |
|  [n1-n2]  |  要选取的字符范围  |
|  [ˆlist]  |  上上个取反  |
|  \{n,m\}  |  	连续 n 到 m 个前一个字符  |
|  \{n,\}  |  连续 n 个以上的前一个字符 |
`# grep -n 'go\{2,3}g' regular_express.txt` 

**sed 工具**

```
# sed [-nefr] [动作]
-n 安静模式，只显示 sed 处理过的列/操作
默认列出所有 stdin 的数据
-e 在命令行模式上进行 sed  的动作编辑
-f 将 sed 的动作写入一个文件，可使用 -f filename 执行
-r 使用扩展型正则表达式的语法
-i 直接修改读取的文件内容而不由屏幕输出
[n1],[n2] 动作 选择进行动作的行数，在 n1 到 n2 行之间进行

function 参数
a 新增，a 后的字符串在目前的下一行新增一行出现
c 替换，c 后的字符串替换 n1,n2 之间的行
d 删除
i 插入，在目前的上一行新增一行出现
p 打印，通常与 sed -n 用
s 替换，通常接正则表达式
```
部分数据的查找并替换
`# sed 's/旧字符串/新字符串/g'`

例子🌰
```
nl /etc/passwd | sed '3,$d'
删除第 3 到最后一行
nl /etc/passwd | sed '2a text1...\
> text2'
增加 2 行字符串
nl /etc/passwd | sed -n '2,3p'
仅显示 2-3 行，注意 -n 
/sbin/ifconfig eth0 | grep 'inet addr' | sed 's/ˆ.*addr://g' | sed 's/Bcast.*$//g'
```
sed 可以直接修改文件内容，使用 -i

# 文件的格式化
**printf**
```
# printf '打印格式' 实际内容
\a 警告音
\b backspace 退格键
\f 清除屏幕
\n 输出新一行
\r enter 回车键
\t  水平 [tab]
\v 垂直 [tab]
\xNN 转换两位数 NN 数值为字符
```
 
**awk**
处理每一行的字段内的数据，默认分隔符是空格键或 [tab] 键

内置变量

|  名称  |  意义  |
| :---: | :---: |
|  NF  |  每行的字段总数  |
|  NR  |  目前是第几行  |
|  FS  |  目前的分隔符  |

```
# awk '条件类型1{动作1} 条件类型2{动作2} ...' filename   
```

# 文件比较工具
以行为单位，用于同一文件新旧版本的区别，也可以比较目录，找出 only in 文件
```
# diff [-bBi] fromfile tofile
fromfile 欲比较的文件
tofile 基准比较文件
-b 一行中的多个空白视为一个
-B 忽略空白行
-i 忽略大小写
```
补丁文件，使用 diff 制作出来的比较文件扩展名为 .patch，以行为单位新文件看到 - 会删除，+ 会加入
```
# patch -R -pN < patch_file
-R 还原，不加则为恢复
-p 取消 N 层目录

patch -p0 < passwd.patch
```
以字节为单位，用于比较二进制文件
```
# cmp [-s] file1 file2
-s 列出所有不同点的字节处
```

