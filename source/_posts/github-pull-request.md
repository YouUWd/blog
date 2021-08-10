---
title: GitHub PR
date: 2019-09-11 15:49:56
tags: [GitHub PR]
categories: Git
---

## PR 流程

```shell
# fork 源代码到自己仓库
git clone git@github.com:YouUWd/mybatis-3.git
# 建立上游连接
git remote add upstream git@github.com:mybatis/mybatis-3.git
# 创建开发分支 (非必须)
git checkout -b dev
# 修改提交代码
git status
git add . 
git commit -m "fix xxx"
git push origin dev
# 同步代码
git fetch upstream
git rebase upstream/master
git push origin master
# GitHub 到自己仓库提交pr

```



## 我的第一个开源PR

[issue](https://github.com/mybatis/mybatis-3/issues/1654)

[pull request](https://github.com/mybatis/mybatis-3/pull/1663)