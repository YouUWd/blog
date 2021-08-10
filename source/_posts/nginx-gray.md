---
title: nginx-gray
date: 2019-11-20 17:52:25
tags: [nginx 灰度]
---

## Nginx 灰度测试

> 灰度测试在系统重构过程中是必不可少的，很多公司有自己的灰度解决方案。对于初创型或者小型团队，缺少必要的工具，因此这里借用nginx的转发（反向代理）能力做一下灰度测试。

## Nginx配置

### Nginx配置文件主要分4部分

-  **main(全局设置)：**  main部分的指令将影响其他所有的设置
-  **server(主机设置)：** server部分的指令主要作用于指定的主机和端口
-  **upstream(负载均衡服务器设置)：** upstream指令主要作用于负载均衡的设置
-  **location(指定网页的设置):** 主要用于匹配上的网页的设置。首先匹配 =，其次匹配^~, 其次是按文件中顺序的正则匹配，最后是交给 / 通用匹配。当有匹配成功时候，停止匹配，按当前匹配规则处理请求。

### proxy_pass

> 在nginx中配置proxy_pass代理转发时，如果在proxy_pass后面的url加/，表示绝对根路径；如果没有/，表示相对路径，把匹配的路径部分也给代理走。

假设下面四种情况分别用 http://192.168.1.1/proxy/test.html 进行访问。

**CASE 1:**

```reStructuredText
location /proxy/ {
    proxy_pass http://127.0.0.1/;
}
```

代理到URL：http://127.0.0.1/test.html

**CASE 2:**

```reStructuredText
location /proxy/ {
    proxy_pass http://127.0.0.1;#少了一个/
}
```

代理到URL：http://127.0.0.1/proxy/test.html

**CASE 3:**

```reStructuredText
location /proxy/ {
    proxy_pass http://127.0.0.1/v1/;#加了一个子路径
}
```

代理到URL：http://127.0.0.1/v1/test.html

**CASE 4:**

```reStructuredText
location /proxy/ {
    proxy_pass http://127.0.0.1/v1;#加了一个前缀
}
```

代理到URL：http://127.0.0.1/v1test.html



## 灰度粒度

### URL级别

```reStructuredText
location  /api_to_gray {
	proxy_pass http://target_endpoint/api_to_gray;
}
```

### 参数级别

```reStructuredText
location  /api_to_gray {
  //$arg_param 是url里面的参数，$http_param 是header里面的参数
  if ($arg_param = "value"){
    add_header Version 'V2';
    proxy_pass http://target_endpoint/api_to_gray;
  }
	proxy_pass http://original_endpoint;
}
```





## 参考文档

[Nginx 内置变量](http://yzone.net/blog/53)