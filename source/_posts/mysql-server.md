---
title: mysql-server
date: 2020-06-02 10:16:25
tags: [mysql server]
---

## 源码

```reStructuredText
https://github.com/mysql/mysql-server.git
```

```powershell
git clone git@github.com:mysql/mysql-server.git
```



## 编译

```
cmake \
-DCMAKE_INSTALL_PREFIX=/tmp/mysql_data/mysql-8.0-rc \
-DMYSQL_DATADIR=/tmp/mysql_data/mysql-8.0-rc/data \
-DSYSCONFDIR=/tmp/mysql_data/mysql-8.0-rc \
-DMYSQL_UNIX_ADDR=/tmp/mysql_data/mysql-8.0-rc/data/mysql.sock \
-DWITH_DEBUG=1  \
-DDOWNLOAD_BOOST=1 \
-DWITH_BOOST=/tmp/boost_1_72
```

