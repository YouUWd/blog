---
title: Git Commit Rewrite
date: 2019-09-18 15:06:22
tags: [Git]
categories: Git
---

## commit message 重写

1. commit 未 push

` git commit --amend `

2. commit 且 push

```shell
#查看需要修改的commit id
git log
# rebase到指定id
git rebase -i commit-id
# 把需要修改的commit-id pick修改为edit，保存
# 修改commit信息并保存
git commit --amend
# 继续直到回到当前分支
git rebase --continue
# 强制push
git push -f

```

> rebase 使用风险极大，请确保当前分支是最新代码，且已经提交了所有本地修改，还要确保rebase期间没有其他人提交代码。