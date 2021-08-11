---
title: 安装zsh及主题(Mac)
date: 2021-08-11 13:55:06
tags: [zsh oh-my-zsh]
---

## Install Homebrew

```shell
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```



## Install Oh My Zsh

```shell
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```



## Install Powerline fonts

```shell
# clone
git clone https://github.com/powerline/fonts.git --depth=1
# install
cd fonts
./install.sh
# clean-up a bit
cd ..
rm -rf fonts
```

## 配置 zsh 支持图标

![配色设置](https://img-blog.csdnimg.cn/img_convert/353b1531f225d06e3719ba6d10a14a3e.png)
![箭头图标等支持Powerline字体](https://img-blog.csdnimg.cn/img_convert/141e68d3b4b5bb5b5865317bab23c9fc.png)

