---
layout: post
title: 端口代理
categories: [server, port, proxy]
description: 端口代理
keywords: server, port, proxy
---
## 术语与约定
本地主机：形式为 IP 或域名，你当前正在使用的这台机器；  
远程主机：形式与本地主机一样。这里的远程并不是指实际的距离有多远，准确地说是另一台；  

## ssh 端口代理
### 本地转发
指设置代理后，远程主机的端口被映射到本地端口上

![server-port-proxy_1.png](https://cdn.jsdelivr.net/gh/Lewinz/lewinz.github.io@master/images/posts/server-port-proxy_1.png)

#### 前置条件
1. Host A 能连接 Host B
2. Host A 不能连接到 Host C
3. Host B 能连接到 Host C 相应端口

#### 示例
``` shell
ssh -NL (Host A).IP:(Host A).Port:(Host C).IP:(Host C).Port root@(Host B).IP <-p (Host B).SSH.Port>

示例：
Host A: 127.0.0.1
Host B: 10.23.43.1
Host C: 10.23.43.2

ssh -NL :5900:10.23.43.2:5900 root@10.23.43.1
等于
ssh -NL 127.0.0.1:5900:10.23.43.2:5900 root@10.23.43.1 -p 22

这时候连接本地 5900 就相当于连接 (Host C) 10.23.43.2:5900

其他场景介绍：
1. 机器 Host C 有进程监听 8080 端口，但安全组未放开 8080 端口，则可使用以下命令通过此机器的公网 IP 代理出 8080 端口。
（其他信息：Host C 内网 IP 为 192.168.11.23 ，外网 IP 为 10.23.43.2）

ssh -NL :8080:192.168.11.23:8080 root@10.23.43.2
```

参数解释：  
-N 告诉 SSH 客户端，这个连接不需要执行任何命令。仅仅做端口转发

-f 告诉 SSH 客户端在后台运行

-L 做本地映射端口，被冒号分割的三个部分含义分别是最后一个参数是我们用来建立隧道的中间机器的 IP 地址 (IP: 18.16.200.134)

### 远程转发
指设置代理后，本地的端口被映射到远程主机的端口上

![server-port-proxy_2.png](https://cdn.jsdelivr.net/gh/Lewinz/lewinz.github.io@master/images/posts/server-port-proxy_2.png)

#### 前置条件
1. Host A 没有公网 IP
2. Host A 可以连接到 Host B

``` shell
ssh -NR (Host B).IP:(Host B).Port:(Host A).IP:(Host A).Port root@(Host B).IP <-p (Host B).SSH.Port>

示例：
Host A: 127.0.0.1
Host B: 10.23.43.1

ssh -NL :5900:127.0.0.1:5900 root@10.23.43.1
等于
ssh -NL 10.23.43.1:5900:127.0.0.1:5900 root@10.23.43.1 -p 22

这时候连接 (Host B) 10.23.43.1:5900 端口就相当于连接本地 5900

可能适合的场景：
1. 内网渗透
2. vpn 端口转发

注意：
本地 sshd_config 里要打开 AllowTcpForwarding 选项，否则 -R 远程端口转发会失败。
默认转发到远程主机上的端口绑定的是 127.0.0.1，如要绑定 0.0.0.0 需要打开 sshd_config 里的 GatewayPorts 选项。这个选项如果由于权限没法打开也有办法，可配合 ssh -L 将端口绑定到 0.0.0.0。
```

## nc
### nc 的作用
- 实现任意 TCP/UDP 端口的侦听，nc 可以作为 server 以 TCP 或 UDP 方式侦听指定端口
- 端口的扫描，nc 可以作为 client 发起 TCP 或 UDP 连接
- 机器之间传输文件
- 机器之间网络测速

nc 的控制参数不少，常用的几个参数如下所列：  
1) -l  
用于指定 nc 将处于侦听模式。指定该参数，则意味着 nc 被当作 server，侦听并接受连接，而非向其它地址发起连接。

2) -s  
指定发送数据的源 IP 地址，适用于多网卡机

3) -u  
指定 nc 使用 UDP 协议，默认为 TCP

4) -v  
输出交互或出错信息，新手调试时尤为有用

5) -w  
超时秒数，后面跟数字

6) -z  
表示 zero，表示扫描时不发送任何数据

### 使用示例
``` shell
1. 测试端口通不通
nc -z -v <IP> <Port> #tcp
nc -z -v -u <IP> <Port> # udp

2. 监听和扫描端口
监听
nc -l <Port> -v

扫描
nc -vz -w 5 <IP> <Port>

3. 收发文件
接受文件端
nc -l <Port> > file.txt

发送文件端
nc <IP> <Port> < file.txt
```