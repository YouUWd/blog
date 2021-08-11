## 站点准备

1. 安装node

```shell
brew install node
```

2. 安装hexo

```shell
npm install hexo
# 将 Hexo 所在的目录下的 node_modules 添加到环境变量之中即可直接使用 hexo <command>：
echo 'PATH="$PATH:./node_modules/.bin"' >> ~/.bash_profile
```

3. hexo 初始化

> 由于hexo初始化需要在一个空白文件夹下执行，这里假设文件夹为 ~/site, 在site文件夹执行 hexo init

```shell
# in ~/site
hexo init
```

4. 安装站点主题

```
#in site folder
# 选择主题
git clone https://github.com/iTimeTraveler/hexo-theme-hipaper.git themes/hipaper
# 配置主题
# 把Hexo主目录下 _config.yml 文件中的theme字段改为 hipaper
```



5. 关联已有站点文档

```shell
# in ~/site
# 去掉初始化的配置及文档，使用本仓库的已有文档
rm -rf _config.yml source
git clone https://github.com/YouUWd/blog.git

ln -s blog/_config.yml  _config.yml
ln -s blog/source source
```

6. 新建文档

```shell
hexo new filename
```

7. 生成对外发布文件

```shell
#当前主题有搜索插件，需要在site目录额外安装hexo-generator-json-content
npm install hexo-generator-json-content
hexo g //手动将public文件夹发布到站点的GitHub
hexo s //预览
```

