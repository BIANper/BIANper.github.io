---
title: Git-分支乱七八糟版
date: 2019-12-07 19:39:54
tags:
- 版本控制系统
- 分支
- 变基
categories: Git
---
使用 git branch <name> 命令创建分支。git branch -d <name> 删除分支。git branch -v 命令查看每个分支的最后一次提交。git branch --no-merged 查看未合并的分支，未合并的分支 -D 才能删除。
切换到一个已存在的分支，使用 git checkout 命令。
```
git checkout -b name
等同于
git branch name
git checkout name
```

运行 git log --oneline --decorate --graph--all ，会输出你的提交历史、各个分支的指向以及项目的分支分叉情况。
使用 git log --decorate命令查看各个分支当前所指的对象。

当合并两个分支时，如果顺着一个分支走下去能够到达另一个分支，那么 Git 在合并两者的时候，只会简单的将指针向前推进，因为这种情况下的合并操作没有需要解决的分歧：即“快进（fast-forward）”。

使用 git merge 命令将指定分支合并到当前分支。
在合并冲突后的任意时刻使用 git status 命令查看因包含合并冲突而处于未合并（unmerged）状态的文件。解决了文件里的冲突后，使用 git add 命令来将其标记为冲突已解决。 一旦暂存原本有冲突的文件，Git 就会将它们标记为冲突已解决。

git push (remote) (branch):(newname) 通过这种格式来推送本地分支到一个命名不相同的远程分支。

git fetch 抓取到新的远程跟踪分支时，本地不会自动生成可编辑的拷贝。即不会有一个新的 serverfix 分支，只有一个不可以修改的 origin/serverfix 指针。可以运行 git merge 将这些工作合并到当前所在的分支。若要在自己的分支上工作，可以 git checkout -b 将其建立在远程跟踪分支之上，这会给你一个用于工作的本地分支，并且起点位于远程分支。
当 git fetch 命令从服务器上抓取本地没有的数据时，并不会修改工作目录中的内容，它获取数据后让你自己合并。而 git pull 在大多数情况下是一个 git fetch 紧接着一个git merge 命令。

从一个远程跟踪分支检出一个本地分支会自动创建“跟踪分支”（它跟踪的分支叫做“上游分支”）。跟踪分支是与远程分支有直接关系的本地分支。 如果在一个跟踪分支上输入 git pull，Git 会自动识别去哪个服务器上抓取、合并到哪个分支。
当克隆一个仓库时，Git 会自动地创建一个跟踪 origin/master 的 master 分支，通过 git checkout -b [branch] [remotename]/[branch] 跟踪其他分支（ Git 提供了 --track 快捷方式）。将本地分支与远程分支设置为不同名字 git checkout -b branchname origin/branch。

设置已有的本地分支跟踪一个刚刚拉取下来的远程分支，或者想要修改正在跟踪的上游分支，可以使用 -u 或 --set-upstream-to 选项运行 git branch 来显式地设置。
查看设置的所有跟踪分支，可以使用 git branch 的 -vv 选项。 这会将所有的本地分支列出来并且包含如每一个分支正在跟踪哪个远程分支，本地分支是否是领先、落后等。

删除服务器上的分支 git push origin --delete branch。

----

变基合并 git rebase。
`git rebase --onto master server client` ：取出 client 分支，找出处于 client 分支和 server 分支的共同祖先之后的修改，然后在 master 分支上重放一遍