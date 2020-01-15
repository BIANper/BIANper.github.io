---
title: Linux 文件与目录补充
date: 2019-09-11 21:10:07
tags:
- 文件
- 目录
- 属性权限
categories: Linux
---
# 默认权限
- 文件
默认没有可执行权限，即最大为 666
- 目录
默认有所有权限，即 777

umask 指定目前用户新建文件或目录时的权限默认值
- 查看
```
# umask [-S]
-S ：以字母显示默认权限，否则是数字
```
 其数字结果是该默认值要被拿掉的权限
<!--more-->
- 设置
 ```
# umask xyz
```
 root 默认是 022，一般用户是 002
用字母进行计算更准确，如 (-rw-rw-rw-) - (--------wx) = -rw-rw-r--,而非 666 - 003 = 663

# 隐藏属性
## chattr
设置文件的隐藏属性
仅在 Ext2/Ext3 的文件系统生效
```
# chattr [+-=][ASacdistu] dirname/filname
+ ：仅增加某一个参数
- ：仅减少某一个参数
= ：仅有后面指定的参数

A ：访问此文件/目录时，访问时间 atime 将不会被修改
S ：对文件的修改将 同步 写入磁盘
a ：root 限定，只能增加数据不能删改
c ：文件将被自动压缩，读取时自动解压，存储时先压缩再存储
d ：dump 程序被执行时，该文件/目录不会被 dump 备份
i ：root 限定，该文件无法被增删改和设置连接
```

## lsattr
显示文件的隐藏属性
```
# lsattr [-adR] filname/dirname
-a 包括隐藏文件
-d 目录属性而非内容
-R 递归，子目录数据也显示
```

# 特殊权限
## SUID 4 u+s
SetUID 只对文件有效，文件所有者的 x 权限为 s
- 仅对二进制程序有效，对于 shell script 无效，靠看其根本的二进制文件设置
- 执行者对改程序有 x 权限 <font color=#D3D3D3>废话</font>
- 仅在执行过程 run-time 中有效
- 执行者将拥有 owner 的权限

例：/etc/shadow 记录了所有账号的密码，**-rwsr-xr-x root root** 仅有 root 可读可强制写入，但用户也能修改自己的密码
## SGID 2 g+s
SetGID 用户组的 x 权限为 s
文件同 SUID
目录
- 用户对此目录有 r 与 x 权限时可进入
- 用户在此目录下的有效用户组将变成该目录的用户组
- 若在此目录下有 w 权限，则新文件的用户组和此目录的用户组相同
- 
## SBIT 1 o+t
StickyBit 只对目录有效，other 的 x 的权限为 x，仅用户自己和 root 有权利删除该目录下的文件/目录

# 查询文件
## 脚本文件名
```
# which [-a] command
-a 列出所有 PATH 目录中可以找到的同名命令，否者显示找到的第一个
```
根据 PATH 环境变量规范的路径查找，查不到 bash 内置的命令

## 文件名
- whereis寻找特定文件 数据库
```
# whereis [-bmsu] filname/filname
-b 仅找二进制文件
-m 仅找说明文件 manual 路径下的文件
-s 仅找 source 源文件
-u 其它特殊文件
```
 查找文件，故不在 PATH 中也能找到；利用数据库，速度快但结果有延迟

- locate 数据库
 ```
 # locate [-ir] keyword
 -i 忽略大小写
 -r 允许接正则表达式
 ```
 可以只输入文件的部分名称(包括路径中)，根据 /var/lib/mlocate/ 中的数据，默认每天更新一次可能有延迟，可手动 updatedb 更新
 
- find 硬盘
```
# find [PATH] [option] [action]
与时间有关的参数
-mtime n 在 n 天前的 一天之内 被更改过的文件，前24小时即 0 
-mtime +n 在 n 天之前被更改过的文件，不含本身 
-mtime -n 在 n 天之内被更改过的文件，含本身 
-newer file 列出比 file 更新的文件

与用户或用户组名有关的参数
-uid n 即 UID，记录在 /etc/passwd
-gid n 即 GID，记录在 /etc/group
-user name 用户账号名称
-group name 用户组名
-nouser 所有者不在 /etc/passwd 的文件
-nogroup 所有用户组不在 /etc/passwd 的文件

与文件权限及名称有关的参数
-name filename 根据文件名查询
-size [+-]SIZE 查找比 SIZE 大或小的文件 k代表1024bytes
-type TYPE 查找文件类型为 TYPE 的；一般文件 f，设备文件 b c，目录 d，连接文件 1，socket s，FIFO p
-perm mode 文件权限刚好是 mode 的文件，类似 chmod 的属性值
-perm +-mode 文件权限全部包含/包含任意 mode 的文件
    例
    #find /bin /sbin -perm +6000
    查找/bin /sbin 两个目录下具有 SUID SGID 的文件
    
其他操作
-exec command 后面可以接其他命令处理查到的结果
    #find / -perm +7000 -exec ls -l {}\;
    {} 代表 find 查找到的内容，\; 转义 本身代表额外命令结束
利用通配符查找
    #find /etc -name '*httpd*'
```