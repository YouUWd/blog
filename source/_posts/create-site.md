---
title: 建站主题配置
date: 2019-09-11 15:49:56
tags: [建站]
categories: 建站
---

## 选择主题

这里选择了 hipaper

```shell
#in blog folder
# 选择主题
git clone https://github.com/iTimeTraveler/hexo-theme-hipaper.git themes/hipaper
# 配置主题
# 把Hexo主目录下 _config.yml 文件中的theme字段改为 hipaper
```

## 配置网站标题

修改hexo配置文件`<blog_root>/_config.yml` 

```reStructuredText
title: #网站标题
subsite：#网站副标题
author: #作者名称
```

支持图片logo

```reStructuredText
# Put your avatar.jpg into `hexo-site/themes/hipaper/source/` directory.
# url is target link (E.g. `url: https://hexo.io/logo.svg` or `url: css/images/mylogo.jpg`)
avatar: 
  enable: true
  width: 124
  height: 124
  bottom: 10
  url: https://hexo.io/logo.svg
```



## 配置tag及category

```shell
# 创建tags页面
hexo new page tags
```

```reStructuredText
# source/tags/index.md 修改为
---
#title: tags
date: 2019-09-11 16:39:26
type: tags
layout: tags
---
# caffolds/post.md  scaffolds/draft.md  里面的tags: 修改为 tags: {{ tags }}
```



```shell
# 生产站点文档
hexo g
```

