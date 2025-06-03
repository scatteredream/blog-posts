---
name: git-usage
title: git 使用总结
date: 2025-05-18
tags: git
---



第一次从远端复制项目： `git clone` GUI

想让本地仓库关联到远程仓库：`git remote add` 远程->添加远程存储库

更改加到暂存区：`git add`   GUI

更改提交到本地仓库：`git commit -M "first commit"`

从本地仓库将提交推送到远程仓库：`git push`

合并 merge 保留分叉，在原来远端的分支生成一个新的 M 合并提交。

变基 rebase 合并成线性

在 `feature` 分支上执行 `git rebase main`，`feature` 分支的提交被重演到 `main`，形成新的提交，

从远端拉取：`git pull` 默认会采用 merge 合并的方式，加`--rebase`改用 rebase 变基形式

回滚更改：`git reset`

抓取/获取：`git fetch` 获取远程最新的提交，但是不合并，仅仅用作查看进度。



最佳实践：

fetch查看远端更改，看一下提交的内容以及结构，然后就可以 pull，这里可以选择 merge 或者 rebase 两种方式，都可以，注意处理冲突情况，在本地合并好就可以push到远端了。