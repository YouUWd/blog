---
title: nginx-variables
date: 2019-10-15 19:44:12
tags: [nginx]
---

## Nginx 内置变量大全

[源码](https://github.com/nginx/nginx/blob/master/src/http/ngx_http_variables.c)

```tex
$args //请求中的的参数名，即“?”后面的arg_name=arg_value形式的arg_name
$arg_PARAMETER //这是参数的一个匹配模式,PARAMETER为具体的参数名,$arg_PARAMETER就表示获取具体的参数值,例如上面的$arg_name就是获取url中name的值
$is_args //判断url是否带参数，如果带，则返回一个？,否则返回一个空字符串

$http_user_agent //获取的是客户端访问代理的类型,请求头中的信息
$sent_http_content_type //获取的是http响应头中content_type的值
$sent_http_content_length //获取的是http响应头重的content_length的值

$request_filename  //该变量获取的是请求的文件在linux服务器上的完整的绝对路径
$request_method  //该表示获取的是http请求的方法
$request_uri  //该变量表示的原始请求的uri，包括参数。所谓原始请求就是即使在内部做了重定向之后也不会变化
$uri  //获取的是当前请求的uri，不包括参数
$content_length  //获取的是http请求头中Content-Length的值
$content_type   //获取的是http请求头中的Content-Type字段，不过这里也没显示。。。
$document_root   //获取的是请求url的文件所在的目录路径
$document_uri   //当前请求的uri，从上面的信息来看,和uri的效果是一样的
$remote_addr  //获取的是客户端的ip地址,这里为什么是10.0.10.11呢，因为我是在本机上用curl测试的，即使客户端也是服务器
$remote_port  //获取客户端的访问端口，这个端口是随机的
$remote_user  //获取客户端的认证用户信息，这里因为没有用认证，所谓显示为空
$server_protocol  //表示服务器端想客户端发送响应的协议
$server_addr  //服务器的地址
$server_name  //客户端访问服务端的域名，即url中的域名(通配符不自动解析)
$server_port //服务器端做出响应的端口号
$binary_remote_addr  //显示二进制的客户端地址
$host  //和server_name一样，表示的是域名
$hostname  //表示服务器端的主机名

$proxy_add_x_forwarded_for //获取的是客户端的真实ip地址
$proxy_host //该变量获取的是upstream的上游代理名称，例如upstream backend 
$proxy_port   //该变量表示的是要代理到的端口
$proxy_protocol_addr  //代理头部中客户端的ip地址，或者是一个空的字符串
$upstream_addr  //代理到上游的服务器地址信息
$upstream_cache_status    //proxy的缓存状态，例如这里第一次访问为MISS，第二次访问时为HIT
$upstream_response_length  //上游服务器响应报文的长度
$upstream_response_time  //上游服务器响应的时间
$upstream_status  //上游服务器响应的状态码

$scheme  //表示的是使用http的访问协议 http or https
$limit_rate  //表示当前连接的限速是多少，0表示无限制
$query_string  //表示的是查询字符串，也就是url中的参数，和$args一样
$realpath_root  //表示的是请求页面的真实所在目录的路径  和$document_root是一样的
```

