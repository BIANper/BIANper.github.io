---
title: Linux 文件与目录
date: 2019-09-08 19:24:50
tags:
- 文件
- 目录
- 属性权限
categories: Linux
---
# 文件属性

## 示例
```
-rw-r--r-- 1 root root 63428 Sep 8 19:24 xxxxx.md
```
### 第一列 第一位
- [d] 目录
- [-] 文件
- [l] 连接文件 linkfile
- [b] 可供存储的设备 是设备文件
- [c] 串行端口设备 是设备文件
<!-- more -->
### 第一列 第二~九位
三个一组，分别为 
- 文件所有者权限
- 同用户组权限
- 其他用户权限

每组三个参数依次为 r读-4 w写-2 x执行-1
Linux 中文件的可执行性与文件名无绝对关系
文件与目录的权限**意义不同**


### 第二列
链接到此节点 i-node 的不同文件名数
&ensp;&ensp;文件名集合权限与属性记录到文件系统的 i-node 中

### 第六列
创建日期或最近修改日期
显示完整时间格式：
```
# ls -l --full-time
```
&ensp;&ensp;中文无法在终端显示所以：
```
# LANG= en_US
```

## 改变文件属性
> 复制行为 cp 不会更改源文件的属性

### chgrp 改变所属用户组
change group
```
# chgrp [-R] groupname dirname/filename
```
组名在 /etc/group 中，否则报错

### chown 改变所有者
change owner
```
# chown [-R] ownername dirname/filename
# chown [-R] ownername:groupname dirname/filename
```
组名在 /etc/passwd 中，否则报错
ownername:groupname 同时更改 owner 和 group，不要用 . 防止误判

### chmod 改变权限
change modify
```
# chmod [-R] xyz dirname/filename
# chmod u/g/o/a +/-/= r/w/x dirname/filename
```

# 目录
树状，记录文件名**列表**

## FHS
/
根目录，开机相关
FHS建议 / 所在分区应越小越好，但以下不应该与根目录分开：
- /etc 配置文件
- /bin 执行文件，可被 root 和一般用户共用
- /dev 设备文件
- /lib 函数库和内核所需模块
- /sbin 执行文件，多用于设定系统文件，root 限定
因为与开机过程有关，开机仅挂载根目录，其他分区在开机完成后进行
根目录也有 . 和 ..，但两者都是根目录本身

/usr 
UNIX software resourse 软件资源数据，可分享 shareable 不可变动 static

/var 
variable 常态性变动文件，缓存，登陆文件，软件运行产生的文件等

## 权限之于目录
- r read contents in directory
读取目录结构列表的权限，即
    - 文件名数据，可利用 ls 查询内容列表
- w modify contents in directory
读取目录结构列表的权限，即
    - 新建新的文件和目录
    - 删除已存在的文件与目录（无视其它权限）
    - 重命名已存在的文件与目录
    - 转移目录内的文件与目录位置
- x access directory
 用户进入该目录成为工作目录的权限
    - 目前所在的目录

例：r--（目录） 用户访问 root 的目录
仅可查询目录下的文件名列表但不能切换到此目录，无法执行任何该目录下的命令
例：rwx（目录） 用户访问 root 的 --- 文件
不可读 编辑 执行 但可以删除

# 路径 PATH
相对路径
相对于当前工作目录，shell scrits 下执行环境不同可能导致问题

绝对路径
一定由根目录 / 写起，正确度更好

系统依照 PATH 的设置去 PATH 定义的目录下查询对应文件名的可执行文件
不同用户默认 PATH 不同，故默认可直接执行指令也不同
但 PATH 可以修改，一般用户可通过修改 PATH 执行 /sbin 或 /usr/sbin 的命令来查询
```
# echo $PATH                查询
# PATH="$PATH":/dirname     添加目录进 PATH
```
相同命令在不同目录，先被查询到的目录先执行
```
# basename 取文件名
# dirname  取目录名
```

