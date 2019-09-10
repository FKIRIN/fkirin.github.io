---
title: git常用技巧
date: 2019-08-07 09:48:37
tags: git
---

### git

- git reflog

> git reflog可以查看所有分支的所有操作记录（包括已经删除的commit 记录喝reset的操作），例如git reset --hard HEAD~1退回到上一个版本，用git log是看不出来被删除的commitId。