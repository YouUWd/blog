---
title: tcpdump及tshark 使用
date: 2019-09-23 19:19:37
tags: [Linux, Net, Tcpdump, Tshark]
---

## 通用抓包工具

1. tcpdump
2. tshark (wireshark 命令行版)

## tshark 安装

`yum install -y wireshark`

## http请求抓包

* tcpdump方式：

```shell
sudo tcpdump -A -s 0 'tcp port 8080 and (((ip[2:2] - ((ip[0]&0xf)<<2)) - ((tcp[12]&0xf0)>>2)) != 0)'
```

* tshark方式：

```shell
# 加 -V 显示包体内容，否则仅url，response_code等
sudo tshark -i eth0 -R 'ip.addr ==100.100.17.97 and http'
```



## mysql 抓包

* tcpdump方式

```shell
sudo tcpdump -i any -s 0 -l -w - dst port 3306 | strings | perl -e '
while(<>) { chomp; next if /^[^ ]+[ ]*$/;
    if(/^(SELECT|UPDATE|DELETE|INSERT|SET|COMMIT|ROLLBACK|CREATE|DROP|ALTER|CALL)/i)
    {
        if (defined $q) { print "$q\n"; }
        $q=$_;
    } else {
        $_ =~ s/^[ \t]+//; $q.=" $_";
    }
}'
```

* tshark方式

```shell
# time_delta 本次回包耗时（RTT）
sudo tshark -i eth0 -R "ip.addr==ip" -d tcp.port==3306,mysql -o tcp.calculate_timestamps:true -T fields -e frame.number -e frame.time_epoch -e frame.time_delta_displayed -e ip.src -e tcp.srcport -e ip.dst -e tcp.dstport -e tcp.time_delta -e tcp.stream -e tcp.len -e mysql.query
```



