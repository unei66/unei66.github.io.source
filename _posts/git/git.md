---
layout: post
title:  "git command"
date:   2018-04-17
categories: git
---

Git常用命令

git init 将当前目录初始化为git目录，当前目录会出现一个.git目录

git add *.c                将c结尾的文件加入版本控制

git add 目录 将目录下所有文件加入版本控制

git commit –m ‘注释’ 将版本控制的文件提交到本地仓库 –m加注释

git commit –a –m ‘注释’ 修改的内容不用再次加入暂存区（git add）,此命令会把所有跟踪的文件暂存起来并一起提交

git clone ‘git://github.com//….git’ 从远程仓库clone到本地

git status 查看文件状态，如果出现修改的，未跟踪或删除的文件会显示出来

git add file/dir 将修改的文件加入暂存，加入暂存后如果又对其进行修改，则需要重新添加文件到暂存

.gitignore 忽略某些文件，编辑此文件即可

*.a                              #忽略所有.a结尾的文件

!lib.a        #lib.a除外

/TODO    #仅忽略根目录下的TODO文件

build/       #忽略build目录下的所有文件

doc/*.txt                  #会忽略doc/notes.txt，但不包括doc/server/arch.txt

 

git diff 查看尚未暂存的文件更新了哪些部分

git diff –cached 查看已经暂存的文件和上次提交时的快照之间的差异

git diff –staged 同上

rm file 删除本地，

git rm file 使git记录此次移除文件的操作

git rm –cached file 从git暂存区中删除，但仍保留在本地目录、

git rm \*~ 递归删除当前目录机器

git mv file_from file_to git移动文件

git log 查看提交历史

​                  -p 展开显示每次提交的内容差异

​                  -n 仅显示最近的几次更新

​                  --stat仅显示简要的增改行数统计

git commit –amend 撤销刚才的提交操作，并用当前的暂存区域快照提交。

git reset file 取消暂存区域中的文件

git checkout -- file 取消修改，回到之前的版本。 

 

Git保存用户名密码：

编辑.git/config文件，添加块

[credential]

​                  helper=store