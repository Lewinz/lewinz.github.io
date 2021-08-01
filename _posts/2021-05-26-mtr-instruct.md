---
layout: post
title: mtr 命令
categories: [mtr,路由 ]
description: mtr 命令
keywords: mtr,路由
---
### MTR
mtr 命令是 ping 和 tracert 命令的结合，同时具有 测试丢包率（ping） 和 跟踪路由（tracert） 的功能

### 安装
#### mac
1. 命令安装 `brew install mtr`

2. MTR 安装之后并未把程序文件复制到 /usr/local/bin 目录下，需要手动复制
```sh
cd /usr/local/Cellar/mtr/0.92/sbin
cp mtr /usr/local/bin/
cp mtr-packet /usr/local/bin/
``` 

3. MTR 命令的使用，需要管理员权限，所以在运行时必须加上 sudo  
`sudo mtr www.baidu.com`

### 命令详解
```sh
mtr -h 提供帮助命令  

mtr -v 显示 mtr 的版本信息  

mtr -r 已报告模式显示  

mtr -c 设置每秒发送数据包的数量  

mtr -s 用来指定 ping 数据包的大小  

mtr -n no-dns 不对 IP 地址做域名解析  

mtr -a 来设置发送数据包的 IP 地址 这个对一个主机由多个 IP 地址是有用的  

mtr -i 使用这个参数来设置 ICMP 返回之间的要求默认是 1 秒  

mtr -4 IPv4  

mtr -6 IPv6  
```

数据含义：  
​Host 列是途经的 IP 或本机域名

Loss% 列就是对应 IP 行的丢包率了，值得一提的是，只有最后的目标丢包才算是真正的丢包

Last 列则是最后一次返回的延迟，按毫秒计算的

Avg 列是所有返回时延的一个平均值

Best 列是最快的一次返回时延

Wrst 列是最长的一次返回时延

StDev 列是标准偏差