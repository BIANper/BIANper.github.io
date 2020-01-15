---
title: vim 的乱七八糟
date: 2019-10-27 23:50:44
tags:
- vim
- 环境配置
- 断行符
categories: Linux
---
# 文件

vim 编辑文件时会新建 .filename.swp 文件，记录所作操作，当 vim 被不正常中断（或他人正在编辑
）时暂存文件不会消失，继续编辑时允许如下操作：
- R 加载暂存文件的内容，恢复未保存的工作，记得离开 vim 后手动删掉暂存文件
- D 确定暂存文件无用，删除并新建本次使用的 .swp
- O 以只读打开，不进行编辑行为，用于他人正在编辑

环境记录文件
~/.viminfo

环境配置文件
/etc/vimrc，建议创建 ~/.vimrc

| -  | - | 
| :----: |:----:|
| :setnu :set nonu |  行号  |
| :set all | 当前环境参数 |
| :set | 与默认不同的参数 |


# 操作
1.块选择

一般模式下 [ctrl]-v 选择，y 复制，p 粘贴

-----------

2.多窗口

| 新窗口  | 光标向下 | 光标向上   |
| :-----| ----: | :----: |
| :sp / :sp{filename} | [ctrl]-w+j  | [ctrl]-w+k |

----

3.多文件
| 当前打开的文件  | 下一个 | 上一个   |
|---------------|:--------------|:--------------:|
|  :files  | :n  | :N |


# 注意
中文编码

```
# LANG=zh_CN.xxx

# iconv --list
# iconv -f 原编码 -t 新编码 filename [-o newfile]
-f from
-t to
-o newfile 保留原本文件
```

-----

断行符
DOS ：^M$，即 CR 和 LF
Linux ：$，仅 LF
```
# dos2UNIX [-kn] file [newfile]
# UNIX2dos [-kn] file [newfile]
-k 不修改 mtime
-n 输出到新文件
```