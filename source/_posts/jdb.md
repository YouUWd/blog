---
title: jdb
date: 2020-08-26 18:55:44
tags: [debug jdb]
---

[jdb官方](https://docs.oracle.com/javase/8/docs/technotes/guides/troubleshoot/introclientissues005.html)

## 操作实例

```java
import java.util.concurrent.TimeUnit;

public class Hello {
    public static void main(String[] args) throws InterruptedException {
        int wait = 20;
        if (args.length > 0) {
            wait = Integer.parseInt(args[0]);
        }
        int i = 0;
        while (i++ < wait) {
            System.out.println(i);
            TimeUnit.SECONDS.sleep(10);
        }
        System.out.println("exit...");
    }
}
```



## 编译

```shell
// com.sun.tools.example.debug.expr.ParseException: Name unknown: wait
// wait = null
javac -g Hello.java #没有-g参数，jdb print 变量 如上错误

```



## 执行

```shell
# win 系统指令有差异
java -Xdebug -Xrunjdwp:transport=dt_socket,address=8888,server=y,suspend=y Hello
```



## IDEA Remote Debug

![](https://youuwd.github.io/images/rd.png)

