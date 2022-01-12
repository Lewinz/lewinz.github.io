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

### 代理场景
** nc 命令在第一个远程连接结束后会结束监听。如果需要保持运行，需要添加 -k 参数 **

| 主机  | IP  | 类型  |
| --- | --- | --- |
| macOS |	192.168.10.100  |	攻击机  |
| kali  |	192.168.19.147	| 跳板机  |
| Ubuntu  |	192.168.19.153  |	目标机  |

1. 正向转发1  
macOS 能访问 kali，但是不能访问 Ubuntu。kali 能访问 Ubuntu 任意端口。

目标：macOS 想访问到 Ubuntu 的 22 端口。  
思路：用 kali 做跳板机，把访问 kali 8888 端口的数据转发到 Ubuntu 的 22 端口  
方法：在 kali 上执行一条 nc 转发命令即可  

``` shell
nc -l -p 8888 -c "nc 192.168.19.153 22"

#或者使用管道符
mkfifo /tmp/pipe && nc -l -p 8888 </tmp/pipe | nc 192.168.19.153 22 >/tmp/pipe
#or
mknod /tmp/pipe p && nc -l -p 8888 < /tmp/pipe | nc 192.168.19.153 22 >/tmp/pipe
此时在 macOS 上用 ssh 连接 kali 的 8888 端口，或者直接在 kali 上 ssh 连接本地 8888 端口，即可登陆 Ubuntu 的 22 端口。

ssh -p 8888 username@192.168.19.147
```

2. 正向转发2
macOS 能访问 kali，但是不能访问 Ubuntu。Ubuntu 防火墙有过滤，kali 不能访问 Ubuntu 的 22 端口，但是可以访问其他端口如 9999。

目标：macOS 想访问到 Ubuntu 的 22 端口。  
思路：  
目标机器 Ubuntu 上用 nc 把 22 端口转发到 9999 端口
kali 上监听 8888 端口，并使用 nc 把访问 kali 8888 端口的数据转发到 Ubuntu 的 9999 端口
macOS 通过访问 kali 的 8888 端口，即可正向连接到 ubuntu 的 22 端口。
操作：  

目标 Ubuntu：
``` shell
nc -l -p 9999 -c "nc 127.0.0.1 22"

#如果没有-c参数
mkfifo /tmp/pipe && nc -l -p 9999 </tmp/pipe | nc 192.168.19.153 22 >/tmp/pipe
# or
mkfifo /tmp/pipe && nc -k -l 9999 0</tmp/pipe | nc localhost 22 | tee /tmp/pipe
```
跳板机 kali：
``` shell
nc -l -p 8888 -c "nc 192.168.19.153 9999"

#或者使用管道符
mkfifo /tmp/pipe && nc -l -p 8888 </tmp/pipe | nc 192.168.19.153 9999 >/tmp/pipe
#or
mknod /tmp/pipe p && nc -l -p 8888 </tmp/pipe | nc 192.168.19.153 9999 >/tmp/pipe
攻击机：macOS：

ssh -p 8888 username@192.168.19.147
```

## 其他
### 端口占用情况
#### netstat
参数
- -a (all) 显示所有选项，默认不显示 LISTEN 相关
- -t (tcp) 仅显示 tcp 相关选项
- -u (udp) 仅显示 udp 相关选项
- -n 拒绝显示别名，能显示数字的全部转化成数字。
- -l 仅列出有在 Listen (监听) 的服務状态
- -p 显示建立相关链接的程序名
- -r 显示路由信息，路由表
- -e 显示扩展信息，例如 uid 等
- -s 按各个协议进行统计
- -c 每隔一个固定时间，执行该 netstat 命令。

提示：LISTEN 和 LISTENING 的状态只有用 - a 或者 - l 才能看到

查看 LISTEN 相关端口 `netstat -nat | grep LISTEN`