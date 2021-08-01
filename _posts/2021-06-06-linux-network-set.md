---
layout: post
title: Linux 网卡设置
categories: [linux,网卡 ]
description: Linux 网卡设置
keywords: linux,网卡
---

## centos7.4 设置网卡
修改配置文件
``` sh
// ens34 为网卡名
vim /etc/sysconfig/network-scripts/ifcfg-ens34
```

``` sh
TYPE=Ethernet    # 网卡类型：为以太网
PROXY_METHOD=none    # 代理方式：关闭状态
BROWSER_ONLY=no      # 只是浏览器：否
BOOTPROTO=dhcp  #设置网卡获得 ip 地址的方式，可能的选项为 static(静态)，dhcp(dhcp 协议) 或 bootp(bootp 协议).
DEFROUTE=yes        # 默认路由：是, 不明白的可以百度关键词 `默认路由`
IPV4_FAILURE_FATAL=no     # 是不开启 IPV4 致命错误检测：否
IPV6INIT=yes         # IPV6 是否自动初始化: 是[不会有任何影响, 现在还没用到 IPV6]
IPV6_AUTOCONF=yes    # IPV6 是否自动配置：是[不会有任何影响, 现在还没用到 IPV6]
IPV6_DEFROUTE=yes     # IPV6 是否可以为默认路由：是[不会有任何影响, 现在还没用到 IPV6]
IPV6_FAILURE_FATAL=no     # 是不开启 IPV6 致命错误检测：否
IPV6_ADDR_GEN_MODE=stable-privacy   # IPV6 地址生成模型：stable-privacy [这只一种生成 IPV6 的策略 ]
NAME=ens34     # 网卡物理设备名称  
UUID=8c75c2ba-d363-46d7-9a17-6719934267b7   # 通用唯一识别码，没事不要动它，否则你会后悔的。。
DEVICE=ens34   # 网卡设备名称, 必须和 `NAME` 值一样
ONBOOT=no #系统启动时是否设置此网络接口，设置为 yes 时，系统启动时激活此设备 
IPADDR=192.168.103.203   #网卡对应的 ip 地址
PREFIX=24             # 子网 24 就是 255.255.255.0
GATEWAY=192.168.103.1    #网关  
DNS1=114.114.114.114        # dns
HWADDR=78:2B:CB:57:28:E5  # mac 地址
```

## ip 命令常用参数
``` sh
Ip  [选项 ]  操作对象 {link|addr|route...}

ip link show                           # 显示网络接口信息
ip link set eth0 upi                   # 开启网卡
ip link set eth0 down                  # 关闭网卡
ip link set eth0 promisc on            # 开启网卡的混合模式
ip link set eth0 promisc offi          # 关闭网卡的混个模式
ip link set eth0 txqueuelen 1200       # 设置网卡队列长度
ip link set eth0 mtu 1400              # 设置网卡最大传输单元

ip addr show                           # 显示网卡 IP 信息
ip addr add 192.168.0.1/24 dev eth0    # 设置 eth0 网卡 IP 地址 192.168.0.1
ip addr del 192.168.0.1/24 dev eth0    # 删除 eth0 网卡 IP 地址

ip route list                                            # 查看路由信息
ip route add 192.168.4.0/24  via  192.168.0.254 dev eth0 # 设置 192.168.4.0 网段的网关为 192.168.0.254,数据走 eth0 接口
ip route add default via  192.168.0.254  dev eth0        # 设置默认网关为 192.168.0.254
ip route del 192.168.4.0/24                              # 删除 192.168.4.0 网段的网关
ip route del default                                     # 删除默认路由
```
