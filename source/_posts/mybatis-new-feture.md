---
title: Mybatis New Feture
date: 2019-09-16 19:47:27
tags: [Mybatis MySQL]
categories: Mybatis
---







## Mybatis新特性之Stream AND MySQL Container

### commit-id： 78fbabdb

支持Mybatis对于fetchSize的支持

```java
@Options(fetchSize = Integer.MIN_VALUE)
```

之前版本报异常，连接被提前关闭

### commit-id： 25ddbb9b

Mybatis支持MySQL容器的测试

```xml
<dependency>
  <groupId>org.testcontainers</groupId>
  <artifactId>mysql</artifactId>
  <version>1.12.1</version>
  <scope>test</scope>
</dependency>
```

这样，用户就不用担心没有可用的MySQL，来进行testcase编写了，有一个缺陷就是，容器测试，效率稍微有些低下。