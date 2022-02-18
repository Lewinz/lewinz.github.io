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
### 端口扫描
#### nmap
参数：

参数 | 描述 
----- | ----- 
-sP           |    Ping扫描,效率高,返回信息少.  例: nmap -sP 192.168.1.110 
-P0(Pn)       |    无Ping扫描,可以躲避防火墙防护,可以在目标主机禁止ping的情况下使用
-PS           |    TCP SYN Ping扫描
-PA           |    TCP ACK Ping扫描
PR            |    ARP Ping扫描
-n            |    禁止DNS反向解析
-T            |    时序选项, -TO-T5. 用于IDS逃逸,0=>非常慢,1=>缓慢的,2=>文雅的,3=>普通的,4=>快速的,5=>急速的
-p            |    指定端口扫描
-F            |    快速扫描
-f            |    报文分段
-D            |    ip地址欺骗  例 nmap -D RND:5 192.168.1.110  RND:为生成随机ip地址
-sS           |    TCP SYN 扫描,速度快, 1秒1000次左右. 
-sT           |    TCP连接扫描
-sU           |    UDP扫描,扫描非常慢,容易被忽视
-sN,-sF       |    隐蔽扫描
-sI           |    空闲扫描,允许端口完全欺骗,可以允许不使用自身ip的情况下发起扫描,非常之隐蔽的扫描.较为隐蔽.但得先寻找空闲主机,指令为 nmap -p80 -open -script ipidseq 192.168......,第二种是往事随机寻找, nmap -p80 -open -script  ipidseq -iR 2000 (iR选项代表随机选择目标.2000带表选择的数量,-open代表只选择端口开放的空闲主机)
-p-           |    扫描所有端口  1-65535
-top-ports    |    只扫描开发概率最高的端口 后面跟上数值  例  nmap -top-ports  100 , 就是扫描概率最高的前100个端口

版本探测相关：
参数 | 描述 
----- | ----- 
-sV                  |    版本探测 ,通过相应的端口探测对应的服务,根据服务的指纹识别出相应的版本.
-sV --allports       |    只有使用--allports才能扫描所有的端口,默认情况下回跳过如 TCP9100端口(hp打印机专用端口)
--version-intersity  |    设置扫描强度 0-9 ,数值越大越有可能被识别,花费时间越多,默认是7
--version-ligth      |    扫描强度,轻量级扫描(2) ,等用于--version-intersity 2
--version-all        |    扫描强度,重量级扫描(9)  ,等同于--version-intersity 9
--version-trace      |    获取详细的版本信息
-sR                  |    判断开放端口是否为RPC端口, 如果是返回程序和版本号.
--resaon             |    显示主机存活原因

实例：
``` shell
1.Nmap的纯扫描,默认情况下，nmap会发出一个arp ping扫描，且扫描目标tcp端口,范围为1-10000。
nmap 127.0.0.1
2.NMAP普通扫描增加输出冗长（非常详细）
nmap -vv 127.0.0.1
3.端口扫描
nmap 127.0.0.1 -p 80  (指定定单个端口）
nmap 127.0.0.1 -p 1-100 （多个端口）
nmap 127.0.0.1 -p- (所有端口)
nmap -sP 10.1.112.89 （Ping扫描）
nmap -sS 127.0.0.1 -p 80 （SYN半连接扫描）
nmap -sT 127.0.0.1 -p 80  （TCP全连接扫描）
nmap -sU 127.0.0.1 （UDP扫描）
nmap -sF  127.0.0.1 （FIN，目标可能有IDS/IPS系统的存在，防火墙可能过滤掉SYN数据包，发送一个FIN标志的数据包不需要完成TCP的握手。）
4.路由追踪
nmap --traceroute 127.0.0.1
5.nmap设置扫描一个网段下的ip
nmap -sP 127.0.0.1 /24
6.扫描目标主机版本（不是很准确）
nmap -O 127.0.0.1 -p 80
7.扫描目标服务版本
nmap -O -sV 127.0.0.1 -p 80
8.全面扫描（包含了1-10000端口ping扫描，操作系统扫描，脚本扫描，路由跟踪，服务探测）
nmap -A 127.0.0.1 -p-
9.保存结果
nmap -A 127.0.0.1 -p- -oN nmap1
10.nmap命令混合式扫描
nmap -vv -p1-100 -o 127.0.0.1
11.扫描多个目标
nmap 127.0.0.1 127.0.0.2
nmap 127.0.0.1-100 (扫描IP地址为127.0.0.1-127.0.0.100内的所有主机)
nmap -iL target.txt （namp在同一目录下,扫描这个txt内的所有主机）

注：使用时请将127.0.0.1更换为目标IP地址
```
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

#### ps
查看相关进程
参数
- a：显示一个终端的所有进程，除会话引线外；
- u：显示进程的归属用户及内存的使用情况；
- x：显示没有控制终端的进程；
- -l：长格式显示更加详细的信息；
- -e：显示所有进程；

查看占用端口的进程 `ps aux | grep 6379`
``` shell
# 输出格式
USER PID %CPU %MEM  VSZ  RSS   TTY STAT START TIME COMMAND
root   1  0.0  0.2 2872 1416   ?   Ss   Jun04 0:02 /sbin/init
root   2  0.0  0.0    0    0   ?    S   Jun04 0:00 [kthreadd]
root   3  0.0  0.0    0    0   ?    S   Jun04 0:00 [migration/0]
root   4  0.0  0.0    0    0   ?    S   Jun04 0:00 [ksoftirqd/0]
```