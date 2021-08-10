---
title: docker_mysql
date: 2020-06-01 10:20:46
tags: [docker mysql]
---

## 拉取镜像

```shell
docker pull mysql
```

## 启动指令

```shell
# 简单启动
docker run --name mysql -e MYSQL_ROOT_PASSWORD=my-secret-pw -d mysql:tag

docker run --name mysql -v /tmp/mysql:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=pass -d mysql:tag

# 端口及目录映射
docker run --name mysql-3306 -v /tmp/mysql:/var/lib/mysql -p 33060:3306 -e MYSQL_ROOT_PASSWORD=pass -d mysql:tag


```



```shell
# 常用
docker run --name mysql-3306 -v /tmp/mysql:/var/lib/mysql -p 33060:3306 -e MYSQL_ROOT_PASSWORD=pass -d mysql --default-authentication-plugin=mysql_native_password

ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY 'pass';
```

