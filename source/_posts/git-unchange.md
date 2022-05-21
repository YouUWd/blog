---
title: Git 不再跟踪及提交某个文件
date: 2019-09-18 15:06:22
tags: [Git]
categories: Git

---

## 查看文件修改状态

```shell
git status

On branch feature/branch_1
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        modified:   dir/file1.txt
        modified:   dir/file2.txt

```



## 不再跟踪指定文件

```shell
git update-index --assume-unchanged dir/file1.txt
```



## 查看文件修改状态

```shell
git status

On branch feature/branch_1
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        modified:   dir/file2.txt
```



## 查看本地不再跟踪的文件列表

```shell
git ls-files -v | grep '^h\ '

h dir/file1.txt
```



## 恢复已忽略文件的追踪

```shell
# 恢复指定文件
git update-index --no-assume-unchanged dir/file1.txt
# 全部恢复
git ls-files -v | grep '^h\ ' | awk '{print $2}' | xargs git update-index --no-assume-unchanged  

git status

On branch feature/branch_1
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        modified:   dir/file1.txt
        modified:   dir/file2.txt
```

