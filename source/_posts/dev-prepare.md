---
title: 通用开发环境搭建
date: 2021-08-12 15:32:34
tags: [java maven docker mysql]
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





