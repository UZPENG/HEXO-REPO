---
title: git常用操作汇总
date: 2018-08-01 10:37:12
tags:
- cheatsheet
---
## 用户初始化

git config --global user.name `<your name>`  
git config --global user.password `<your password>`  
ssh-keygen 生成秘钥  
vim ~/.ssh/authorized_keys 添加客户端秘钥  
cat ~/.ssh/id_rsa.pub 输出公钥  
```
su git #切换用户
sudo chsh -s /usr/bin/git-shell 更改登录的shell脚本
```

<!--more-->

## 初始化仓库

git init --bare 创建纯仓库（只能拉取之后修改，不能直接修改）  
git clone `<your  url>` 克隆已有的仓库

## 仓库基本操作

git add `<your file>` 添加文件到暂存区  
git commit `<your file>` 提交文件
> -a 提交的时候，先将所有改变添加到暂存区

git rm `<your file>` 撤销暂存区文件  
git mv `<your file>` 重命名文件  
git status 查看状态
git log 查看提交记录
> -p 查看变化  
> -number 查看最近几次  
> --pretty=format 格式化输出内容  
> --since=2.weeks 输出最近两周的记录。

vim .gitignore 添加忽略的文件  
`git commit --amend` 修改提交信息或者往上一个commit里面多加几个文件。
git checkout your file 还原已经修改的文件

## 远程操作

git remote -v 查看远程分支状态  
git add remote `<name>` `<url>` 添加分支  
git fectch `<branch name>`从远程仓库拉取更新(默认分支为origin，可以指定分支)  
git push `<branch name>`向远程仓库推送（默认分支为origin，可以指定分支）  
git remote show `<name>` 输出分支详细信息  
git remote remove `<name>` 删除分支  
git remote mv `<old name>` `<new name>` 重命名分支
git push `<remote>` --delete `<branch name>` 


## 技巧

git config --global alias.co chechout

## 分支

git branch `<your branch>` 创建分支  
git checkout -b `<branch name>` 创建并切换分支  
git checkout `<branch name>` 切换分支  
git log --graph 查看分支情况  
git merge `<branch name>`  
git branch -v 查看分支详情  
git branch --merged 查看已合并的分支  
git branch --no-merged 查看未合并的分支  
git branch -d `<branch name>` 删除指定分支  

git fetch `<remote branch name>` 从服务器拉取指定远程分支的更新  
git push `<remote> \<local branch>` 推送指定本地分支到远程分支  
git push --force  强制覆盖服务端推送
git checkout -b `<local branch>` `<remote / local branch>` 以指定远程分支为副本，创建指定本地分支  

git rebase `<branch>` 设当前分支和指定分支的共同祖先节点为A,指定分支的最新节点为B，当前分支的A节点后一个节点为C，将C到A的指针指向B。  
git rebase --onto `<dist> <branch 1> <branch 2>` 同理如上。

rebase和merge的差异如下图所示：
![img](/img/1.png)

## 标签

git tag -a `<tag name>` -m `<message>`  
git tag show `<tag name>` 显示tag相关信息  
git tag -l `<regx string>` 过滤指定tag  
git tag -a `<tag name>` `<hash str>` 后期打标签  
git push `<remote branch>` `<tag name>` 推送指定tag到服务器  
git push `<remote branch>` --tags 推送所有tag到服务端  

## 储藏和清理

git stash save 保存还没提交的暂存区信息  
git stash list 输出所有已存储的信息  
git stash apply `<id>` 应用栈顶的储存，指定id时，应用指定id的储存  
git stash clear 清理所有储存

git clean 清理所有已被跟踪的文件

## 代理

git config --global https.proxy http://127.0.0.1:1080  
git config --global https.proxy https://127.0.0.1:1080

git config --global --unset http.proxy
git config --global --unset https.proxy

## 删除提交记录

git reset head~1(回退到head指针往回一个版本)

## 中文支持

vim  ~/.bashrc  
> export LANG=zh_CN.UTF-8