---
title: Linux 文件对比命令
date: 2019-11-06 13:34:37
tags: [Linux diff comm grep]
---

## comm 指令

```shell
comm [-123][--help][--version][第1个文件][第2个文件]
```

**参数**：

- -1 不显示只在第1个文件里出现过的列。
- -2 不显示只在第2个文件里出现过的列。
- -3 不显示只在第1和第2个文件里出现过的列。
- --help 在线帮助。
- --version 显示版本信息。

> 需要排序，即便排序也不一定正确，所以不建议使用，适合对比极少数同行差异的文件。

## diff 指令

同comm指令，也是逐行对比，如果同样的内容处于不同行中，对比结果也不是期望的

```shell
diff [-abBcdefHilnNpPqrstTuvwy][-<行数>][-C <行数>][-D <巨集名称>][-I <字符或字符串>][-S <文件>][-W <宽度>][-x <文件或目录>][-X <文件>][--help][--left-column][--suppress-common-line][文件或目录1][文件或目录2]
```

## grep 指令

```shell
grep [-abcEFGhHilLnqrsvVwxy][-A<显示列数>][-B<显示列数>][-C<显示列数>][-d<进行动作>][-e<范本样式>][-f<范本文件>][--help][范本样式][文件或目录...]
```

```shell
# 查看file2比file1多出的内容
grep -v -x -f file1 file2
# 查看file1和file2中单独存在的内容（差集1+差集2）
grep -v -x -f file1 file2 &&  grep -v -x -f file2 file1 
# 查看file1和file2的交集
grep -x -f file1 file2
```

> -x --line-regexp : 只显示全列符合的列。
>
> -f <规则文件> 或 --file=<规则文件>: 指定规则文件，其内容含有一个或多个规则样式，让grep查找符合规则条件的文件内容，格式为每行一个规则样式。

