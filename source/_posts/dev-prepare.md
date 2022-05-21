---
title: 通用开发环境搭建
date: 2021-08-12 15:32:34
tags: [java maven docker mysql gitlab github]
---

> 以下文件安装默认目录为~/dev  
>
> ~/dev 后续路径为解压包后的完整路径

## Java 开发环境搭建

[Java SE Downloads](https://www.oracle.com/java/technologies/javase-downloads.html)

> 一般选择压缩包(Compressed Archive)安装, 这样可以定制化安装到指定目录。
>
> 如果是ARM架构，建议使用[Zulu -CA-macos-aarch64](https://www.azul.com/downloads/?package=jdk#download-openjdk) ARM 64-bit

### 环境变量

```shell
JAVA_HOME=~/dev/zulu8.56.0.23-ca-jdk8.0.302-macosx_aarch64/zulu-8.jdk/Contents/Home
PATH=$PATH:$JAVA_HOME/bin
CLASS_PATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
```

## MAVEN 安装及配置

> 下载tar.gz 解压即可，最好不要修改settings.xml里面的localRepository，因为idea内置maven，默认的目录为.m2，因此如果要修改settings.xml，需要一并修改。

### 环境变量

```shell
MAVEN_HOME=~/dev/apache-maven-3.8.1
PATH=$PATH:$MAVEN_HOME/bin
```



## MySQL Client 安装

> MySQL Server 在mac机器上一般使用docker(也有arm版了)安装，以便切换不同的mysql版本。
>
> MySQL Client 则可以使用mysql的安装包里面工具。如果是ARM架构，可选择[macos11-arm64](https://dev.mysql.com/downloads/mysql/)  arm64.tar.gz

### 环境变量

```shell
PATH=$PATH:~/dev/mysql-8.0.26-macos11-arm64/bin
```



## Github  GitLab SSH keys 配置

> ed25519加密解密很快,生成时间短而且安全性更高,rsa则加密解密稍慢,生成时间长,安全性没有ed25519高,只是rsa基本都是默认,所以用的人更多,但是建议转换为ed25519,网站软件现在基本都支持了.

### Github (ed25519)

```shell
ssh-keygen -t ed25519 -C "your_email@example.com"
```

### GitLab (rsa)

```shell
ssh-keygen -t rsa -C "your_email@example.com"
```

### 配置ssh config

```shell
#将上面生成的id_rsa 及 id_rsa.pub 分别移动到github和gitlab目录
# cat ~/.ssh/config
# gitlab
Host gitlab.*.com
    HostName gitlab.company.com
    PreferredAuthentications publickey
    IdentityFile ~/.ssh/gitlab/id_rsa

#github
Host github.com
    HostName github.com
    AddKeysToAgent yes
    UseKeychain yes
    PreferredAuthentications publickey
    IdentityFile ~/.ssh/github/id_ed25519
```



### 验证SSH是否配置成功

```shell
ssh -vT git@github.com
ssh -vT git@gitlab.company.com
```

