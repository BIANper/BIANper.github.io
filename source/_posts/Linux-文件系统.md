---
title: Linux 文件系统
date: 2019-09-18 10:45:51
tags:
- 目录树
- 日志
- 存储器
- 文件系统
categories: Linux
---
# 回顾硬盘
## 分区
磁盘分区指定分区的起始与结束**柱面**，分区的柱面范围记录在第一个扇区的分区表里面

硬盘的第一个扇区中有主引导记录 **MBR 446bytes** 和分区表 **partition table 64bytes**，由于大小限制最多只能记录四条分区的记录，即主分区或扩展分区
- GPT + UEFI 则扫描整块磁盘上的分区，读取 EFI 分区里的引导文件，所以不再限制在扇区开头的 512bytes 中

硬盘限制主分区和扩展分区最多可以有四个，操作系统限制扩展分区只能有一个，无法被格式化即无法作为数据访问，但扩展分区可以再分出逻辑分区

## 格式化
文件系统通过格式化将 inode 和 block 规划好，并不再变动；不同文件系统格式化方法不同，所以不能相互识别利用
- inode 记录文件属性和此文件的数据所在的 block，一个文件占用一个 inode
- block 实际记录文件内容，可占用多个
- super block 记录此文件系统的整体信息，如 inode/block 总量使用量剩余量，文件系统的格式等

碎片整理即因为文件写入的 block 过于离散，影响文件读取性能，需要将它们汇合到一起；这针对没有 inode 的 FAT 系统，Linux 的索引式文件系统基本不需要

----------------------

# 文件系统
当 inode 和 block 数量极大时不容易管理，Ext2 在格式化时区分为多个块组 block group，每个块组有独立的 inode/block/superblock 系统
![avatar](http://cn.linux.vbird.org/linux_basic/0230filesystem_files/ext2_filesystem.jpg)
boot sector 启动扇区
在文件系统最前面，用于安装引导装载程序，这样不用覆盖整块硬盘唯一的 MBR，制作多重引导环境

data block 数据块
放置文件内容，每个 block 最多放置一个文件的数据，若有剩余空间则浪费；Ext2 支持 1KB 2KB 4KB 三种 block 大小，其决定最大总容量和最大单一文件容量

inodetable
至少记录访问模式 所有者和组 大小 ctime atime mtime 内容指向 文件特性，系统读取文件时会先分析 inode 所记录的权限与用户是否符合；每个文件占用一个inode，所以文件系统能创建的文件数量与 inode 数量有关
inode 大小固定为 128bytes，当文件较大则不足以记录下所有的 block 编号，故定义了 12 个直接，一个间接，一个双间接，一个三间接记录区
- 用一个 block 来记录额外的编号，若依然不够则用一个 block 指出下一个记录编号的 block，依此最多三层；以 1KB 大小的 block 计算，12\*1K + 256\*1K + 256\*256\*1K + 256\*256\*256\*1K = 16GB，即该文件系统最大容量

superblock
一般为 1024bytes，除了第一个 blockgroup 中含有 superblock 外，后面的不一定含有，若有则为备份

文件系统描述 File system Dscription
描述每个 blockgroup 的开始与结束编号，说明每个区段介于哪个 block 之间
- 区段指 superblock bitmap inodemap datablock
- block bitmap 标记/修改某个 block 是否被使用
- inode bitmap同上

查询文件系统
```
# df         <== 查询目前挂载的设备
# dumpe2fs [-bh] devname
-b ：列出坏道部分
-h ：仅列出 superblock 数据
```

新增文件的过程
- 检查目录是否有 w 和 x 权限
- 根据 inode bitmap 查找没有使用的 inode 编号并写入新文件的权限属性
- 根据 block bitmap 查找没有使用的 block 编号并写入实际数据，更新 inode 的指向
- 将 inode 和 block 信息更新到 inode bitmap 与 block bitmap，并更新 superblock

由于 superblock inodebitmap blockbitmap 的数据经常变动，被称为 metadata 中间数据，而 inode table 和 data block 称为数据存放区域

异步处理 asynchronoly
提高效率：系统加载一个文件到内存后，若文件没有被改动，则被设置成 **clean**，更改过则为 **dirty**，系统不定时将 dirty 数据写回硬盘
- 可以 sync 命令手动写回
- 正常关机会主动调用 sync 命令

----------------

# 与目录树的关系
ext2 中的目录
新建目录时，会分配一个 inode 和至少一块 block，inode 记录目录的相关权限，属性，和分配到的 block 编号；block 记录此目录下的文件名与文件名的 inode 编号数据

ext2 中的文件
新建一般文件时，会分配一个 inode 和对应大小的 block 数量，由于只有 12 个直接指向，可能分配额外的 block 用来记录块编号

**inode 本身不记录文件名，而是在目录的 block 中，所以增删改文件名与目录的 w 权限有关。
当读取某个文件时务必会经过目录的 inode 和 block，然后找到文件的 inode 最终获得 block 中的数据
系统通过挂载信息获得根目录的 inode 号码，并据此获得根目录 block 内的文件名数据，再一层层向下**

----------------

# 日志文件系统
inconsistent 状态
文件在写入文件系统时发生中断，未同步更新 metadata 导致 metadata 的内容和实际数据存放区不一致

避免 inconsistent 发生
- 写入文件前先在日志记录块记录准备要写入的信息
- 实际写入，包括更新 metadata 的数据
- 在日志记录块中完成该文件的记录

发生问题时通过检查日志记录块定位到文件，不必检查整个文件系统，Ext2 中没有文件日志系统，需要对比 metadata 区域和数据存放区
