---
layout: post
title: CentOS 6 添加 dns 缓存
categories: [centos, dns, cache]
description: CentOS 6 添加 dns 缓存
keywords: centos, dns, cache
---
### 安装 dnsmasq
`yum install dnsmasq -y`

### 修改配置文件：
#### 修改配置文件：`/etc/dnsmasq.conf` 添加以下内容：
``` shell
resolv-file=/etc/resolv.dnsmasq.conf
listen-address=127.0.0.1
no-dhcp-interface=lo
bind-interfaces
no-negcache
```

#### 修改配置文件： `/etc/resolv.conf`  添加以下内容：
``` shell
nameserver 127.0.0.1
nameserver 119.28.28.28
nameserver 114.114.114.114
```

#### 修改配置文件：`/etc/resolv.dnsmasq.conf`  添加以下内容：
``` shell
nameserver 119.29.29.29
nameserver 119.28.28.28
nameserver 114.114.114.114
nameserver 114.114.115.115
```
#### 重启 dnsmasq 服务:
`service dnsmasq restart`