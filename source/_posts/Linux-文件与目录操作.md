---
title: Linux 文件与目录操作
date: 2019-09-09 09:54:19
tags:
- 文件
- 目录
categories: Linux
---
# 目录处理
cd change directory 切换目录
```
# cd -          返回之前的目录
```
pwd print working directory 显示当前目录
```
# pwd -p        获得实际路径而非连接路径
```
mkdir 新建目录
```
# mkdir [-mp] dirname
-m ：新建自定义权限的目录 如 mkdir -m xyz test
-p ：递归创建 如 mkdir -p test1/test2/test3
```
rmdir 删除空目录
<!--more-->
```
# rmdir [-p] dirname 同上
```

# 文件处理
 cp copy
```
# cp [-adfilprsu] source destination
-a ：同-pdr
-d ：若源文件是连接文件则复制连接文件而非本身
-i ：若目标文件存在，询问是否覆盖
-l ：创建硬连接的连接文件
-p ：连同文件属性复制，常用于备份
-r ：递归复制，用于目录的复制行为
-s ：创建符号连接文件，即快捷方式
-u ：源文件新才 update
```
 默认情况下目的文件的所有者是命令操作者
 没有参数时 cp 复制的是**源文件**而非连接文件
 
 rm remove
 ```
 # rm [-fir] dirname/filename
 -f : force 强制，忽略不存在的文件且不警告
 -i ：删除前进行询问
 -r ：递归删除
 ```
 目录名以 - 开头会造成误判，可以用 ./ 避免
 
 mv move
 ```
 # mv [-fiu] source destination
 -f ：force 若目标文件已存在则直接覆盖不询问
 -i ：若目标文件存在，询问是否覆盖
 -u ：源文件新才 update
 ```
 多个源文件或目录要移动时，最后一个（即目标文件）一定是目录 <font color=#D3D3D3>废话</font>
 
 # 文件内容查阅
 ## 直接查看
- cat concatenate 从第一行开始显示文件内容
```
 # cat [-AbEnTv]
 -A ：相当 -vET 整合，特殊字符 断行字符$ Tab键＾I
 -b ：列出行号，空白行不标号
 -n ：列出行号，包括空白行
 ```
- tac cat 反写，功能也是
- nl 添加行号显示
 ```
 # nl [-bnw]
 -b ：行号的指定方式 
      a 空行也显示，类似 cat -n
      t 空行不显示行号，默认值
 -n ：行号的表示方法
      ln 行号字段最左显示
      rn 行号字段的最右显示，不加 0
      rz 行号字段的最右显示，加 0
 -w ：行号字段的位数，默认六位
 ```
 
 ## 翻页查看
 - more 
     Space ：向后翻一页
     Enter ：向下滚动一行
     /字符串 ：当前页向下查询，n 继续查询
     :f ：显示出文件名以及目前显示的行数
     b/ctrl+b ：回翻，仅限文件，FIFO无效

- less
     Space/PageDown ：向后翻一页
     PageUp ：向前翻一页
     ?字符串 ：当前页向上查询，n 继续查询
     N ：反向继续查询

## 数据选取
head 取出前几行，默认显示十行
```
# head [-n num] file      取前 num 行
# head [-n -num] file     取 num 行之前，第二个是减号ao（x
```
tail 取后面几行
```
# tail [-n num] file      取后 num 行
# tail [-n +num] file     取 num 行之后
-f 持续监测，应对数据随时写入
```
od 非纯文本文件
```
# od [-t TYPE] 文件
TYPE a ：默认字符输出
     c ：ASCII 字符输出
     dfox ：十进制 浮点数 八进制 十六进制
```

# 文件时间处理
```
# ls -l --time=?time filname
```
- mtime modification time
内容数据更改时更新，ls 默认值
- ctime status time
状态改变时更新，如权限与属性
- atime access time
内容被取用访问时更新

```
# touch [-acdmt] file
-a ：修改 atime
-c ：修改文件时间，文件不存不创建新文件
-d ：修改 atime mtime，ctime 不变
-m ：修改 mtime
-t ：修改 atime mtime，ctime 记录当前时间，格式 YYMMDDhhmm
```
默认将三个时间更新为当前，若文件不存在创建新文件

# 文件类型查看
```
# file filename
```