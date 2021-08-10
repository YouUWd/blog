---
title: tcpdump及tshark 使用
date: 2019-09-23 19:19:37
tags: [Linux, Net, Tcpdump]

---

译自 [A tcpdump Tutorial and Primer with Examples](https://danielmiessler.com/study/tcpdump/) 有所增删，可以通过 `man tcpdump` 获取更详细信息

只要和网络沾边，都可以使用tcpdump来排查问题。

## 1  基础

### 1.1 常用选项

- `-i any` ：监听任意网络设备
- `-i eth0` ：监听eth0
- `-D` ：列出所有可监听设备
- `-n` ：不要把IP解析为主机名（显示IP）
- `-nn` ：不要把IP和PORT解析为主机名和服务（显示IP和端口）
- `-q` ：显示更少信息
- `-tttt` ：更具有可读性的时间格式
- `-X` ：以hex和ascii格式显示包内容
- `-XX` ：同 `-X` , 同时包含以太网协议头
- `-v, -vv, -vvv` ：显示更多信息
- `-c x` ：最多抓取x个包，然后退出
- `-A` ：以ASCII格式打印每个包（最小化link level的头）,适合抓web页面
- `-s` ：Define the snaplength (size) of the capture in bytes. 使用 `-s0` 获取全部
- `-w file` ：把结果输出到文件file
- `-S` ：Print absolute sequence numbers.

### 1.2 表达式

表达式用于过滤抓到的包，有三类表达式

- 类型 `host` `net` `port`
- 数据流向 `src` `dst`
- 协议 `tcp` `udp` `icmp` 等

## 2 例子

### 2.1 获取所有流量

`tcpdump -i any` 当机器有多个网卡，不确定流量走哪个时，使用这个选项

### 2.2 获取eth0上的流量

```shell
tcpdump -i eth0
```

### 2.3 查看原始流量

```shell
tcpdump -nn -tttt -vv -S -c 10
```

### 2.4 查看指定host和port

```shell
tcpdump port 80 and host 10.129.204.105
```

### 2.5 以hex和ascii查看包内容

```shell
tcpdump -nn -v -X -s0 -c 1 tcp
```

### 2.6 指定数据的方向

获取所有目标端口是80的流量 `tcpdump -i any -nn -s0 -c 10 -A tcp and dst port 80`

获取所有80端口的流量 `tcpdump -i any -nn -s0 -c 10 -A tcp and port 80`

### 2.7 获取指定协议的流量

获取vrrp协议的流量，用于调试keepalive `tcpdump vrrp`

### 2.8 获取ipv6的流量

```shell
tcpdump ip6
```

### 2.9 获取某个端口范围的流量

```shell
tcpdump -nn portrange 21-23 and src host 10.136.110.179
```

### 2.10 基于包大小过滤

```shell
tcpdump -i any -nn port 80 and less 80
tcpdump -i any -nn port 80 and greater 80
```

## 3 进阶

### 3.1 表达式组合

通过 `and 或 &&` `or 或 ||` `not 或 ！` 可以组合表达式

表达式的结合律有点奇怪，需要自己体会，例如： 监听80或8080并且来自10.129.204.105的请求 `tcpdump -i any -nn -vv port 80 or 8080 and src host 10.129.204.105` `tcpdump -i any -nn -vv port 80 or port 8080 and src host 10.129.204.105`

监听80并且来自10.129.204.105的请求或监听8080的请求（没有来源IP限制） `tcpdump -i any -nn -vv port 80 and src host 10.129.204.105 or port 8080`

为了避免这些微妙的差别，可以使用 `()` ，注意这里的单引号 `tcpdump -i any -nn -vv '(port 80 or port 8080)' and src host 10.129.204.105`

### 3.2 指定源host和目标端口

```
tcpdump -i any -nn -vv port 80 and src host 10.129.204.105
```

### 3.3 目标是某台机器的所有非tcp请求

```
tcpdump -i any -nn dst host 10.136.110.179 and not tcp
```

### 3.4 来自某台机器的所有非22请求

```
tcpdump -i any -nn src host 10.129.204.105 and not dst port 22
```

### 3.5 获取tcp flag

先了解下TCP和IP协议头，虽然最后有0~40个可选bytes，这里没用到，可以先忽略它们。TCP头20bytes，其中tcp flags在第13个byte（下标为0） `|C|E|U|A|P|R|S|F|` 。

```
TCP
 0                            15                              31
-----------------------------------------------------------------
|          source port          |       destination port        |
-----------------------------------------------------------------
|                        sequence number                        |
-----------------------------------------------------------------
|                     acknowledgment number                     |
-----------------------------------------------------------------
|  HL   | rsvd  |C|E|U|A|P|R|S|F|        window size            |
-----------------------------------------------------------------
|         TCP checksum          |       urgent pointer          |
-----------------------------------------------------------------
|                 optional data 0~40 bytes                      |
-----------------------------------------------------------------

 |---------------|
 |C|E|U|A|P|R|S|F|
 |---------------|
 |7   5   3     0|

IP
  0| 1| 2| 3| 4| 5| 6| 7| 8|9|10|11|12|13|14|15|16|17|18|19|20|21|22|23|24|25|26|27|28|29|30|31
-----------------------------------------------------------------------------------------------
| version   |    IHL    |      DSCP       | ECN |             total length                     
-----------------------------------------------------------------------------------------------
|             Identification                    | Flags  |       Fragment Offset               
-----------------------------------------------------------------------------------------------
|          TTL          |      Protocol         |               Header Checksum                
-----------------------------------------------------------------------------------------------
|                                         Source IP                                            
-----------------------------------------------------------------------------------------------
|                                        Destination IP                                        
-----------------------------------------------------------------------------------------------
```

#### 3.5.1 flag列表

- URGENT (URG) `tcp[13] & 32 != 0` 或 `tcp[tcpflags] == tcp-urg`
- ACKNOWLEDGE (ACK) `tcp[13] & 16 != 0` 或 `tcp[tcpflags] == tcp-ack`
- PUSH (PSH) `tcp[13] & 8 != 0` 或 `tcp[tcpflags] == tcp-psh`
- RESET (RST) `tcp[13] & 4 != 0` 或 `tcp[tcpflags] == tcp-rst`
- SYNCHRONIZE (SYN) `tcp[13] & 2 != 0` 或 `tcp[tcpflags] == tcp-syn`
- FINISH (FIN) `tcp[13] & 1 != 0` 或 `tcp[tcpflags] == tcp-fin`

例如：获取所有SYN请求 `tcpdump -i any -nn -A -s0 'tcp[13] & 2 != 0'`

#### 3.5.2 获取tcp连接的开始和结束

```shell
tcpdump -i any -nn -s0 -X 'tcp[tcpflags] & (tcp-syn|tcp-fin) != 0'
```

#### 3.5.3 获取所有HTTP GET请求

`tcp[12]` 的高4位是TCP HEADER的长度（通常是20 bytes）,TCP之后是应用协议 `0x47455420` 是 GET 外加一个空格 `tcpdump -i any -nn -s0 -X 'tcp[((tcp[12] & 0xf0) >> 2):4] = 0x47455420'` 通常 `tcpdump -i any -nn -s0 -X 'tcp[20:4] = 0x47455420'` 也是可以的

#### 3.5.4 获取所有SSH连接

```shell
0x5353482D` 是 `SSH-` `tcpdump -i any -nn -s0 -X 'tcp[20:4] = 0x5353482D'
```

#### 3.5.5 获取所有带数据的包

```shell
ip[2:2]` 整个IP包的长度 `(ip[0] & 0xf) <<2` 是IP头的长度 `tcpdump -i any -nn -s0 -X port 80 and '(((ip[2:2] - ((ip[0]&0xf)<<2)) - ((tcp[12]&0xf0)>>2)) != 0)'
```