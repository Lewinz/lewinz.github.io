---
layout: post
title: 跳板机使用
categories: [OpenSSH,ProxyJump]
description: 跳板机使用
keywords: OpenSSH,ProxyJump
---

### OpenSSH
OpenSSH 是 SSH （Secure SHell） 协议的免费开源实现。SSH协议族可以用来进行远程控制， 或在计算机之间传送文件。而实现此功能的传统方式，如telnet(终端仿真协议)、 rcp ftp、 rlogin、rsh都是极为不安全的，并且会使用明文传送密码。OpenSSH提供了服务端后台程序和客户端工具，用来加密远程控件和文件传输过程中的数据，并由此来代替原来的类似服务。

### ProxyJump
需要 OpenSSH 7.3 以上版本才可以使用 ProxyJump, 使用下列命令查看OpenSSH 版本：  
`ssh -V`

ProxyJump 命令行使用方法：
`ssh -J [email protected]:port1,[email protected]:port2`

可以直接使用上述命令通过跳板机直接登录内网机器，比如：  
`ssh username@目标机器IP -p 22 -J username@跳板机IP:22`

如果需要通过多个跳板机则以 , 分割：  
`ssh username@目标机器IP -p 22 -J username1@跳板机IP1:22,username2@跳板机IP2:22`

如果你觉得每次都需要加上 -J 的配置很多麻烦，可以写到配置文件里。修改配置文件 ~\.ssh\config，默认没有需要自己创建。增加以下内容：
``` txt
Host target    # 代表目标机器的名字
    HostName 目标机器 IP    # 这个是目标机器的 IP
    Port 22    # 目标机器 ssh 的端口
    User username_target    # 目标机器的用户名
    ProxyJump username@跳板机IP:port

Host 10.10.0.*    # 使用通配符 * 代表 10.10.0.1 - 10.10.0.255
    Port 22    # 服务器端口
    User username    # 服务器用户名
    ProxyJump username@跳板机IP:port
```

也可以为跳板机器一个“别名”方便使用：
```txt
Host tiaoban1    # 代表跳板机 1
    HostName 跳板机 1 的 IP
    Port 22    # ssh 连接端口
    User username1    # 跳板机 1 的用户名

Host tiaoban2    # 代表跳板机 2
    HostName 跳板机 2 的 IP
    Port 22    # ssh 连接端口
    User username2    # 跳板机 2 的用户名

Host target    # 代表目标机器的名字
    HostName 目标机器 IP    # 这个是目标机器的 IP
    Port 22    # 目标机器 ssh 的端口
    User username_target    # 目标机器的用户名
    ProxyJump tiaoban1,tiaoban2

Host 10.10.0.*    # 使用通配符 * 代表 10.10.0.1 - 10.10.0.255
    Port 22    # 服务器端口
    User username    # 服务器用户名
    ProxyJump tiaoban1,tiaoban2
```

使用方法：
```txt
ssh target
ssh 10.10.0.1
ssh username@target -p22
ssh username@10.10.0.1 -p22
```

### ProxyCommand
如果 OpenSSH 版本低于 7.3 可以使用 ProxyCommand达到同样的效果。

ProxyCommand 命令行使用方法：

`ssh username@目标机器IP -p 22 -o ProxyCommand='ssh -p 22 username@跳板机IP -W %h:%p'`

同样可以在 ~/.ssh/config 文件中增加配置文件：
```txt
Host tiaoban   # 任意名字，随便使用

    HostName 跳板机的 IP，支持域名

    Port 22      # 跳板机端口

    User username_tiaoban       # 跳板机用户

 

Host target      # 同样，任意名字，随便起

    HostName 目标服务器 IP    # 真正登陆的服务器，不支持域名必须IP地址

    Port 22   # 服务器的端口

    User username   # 服务器的用户

    ProxyCommand ssh tiaoban -W %h:%p



Host 10.10.0.*      # 可以用*通配符

    Port 22   # 服务器的端口

    User username   # 服务器的用户

    ProxyCommand ssh tiaoban -W %h:%p
```
使用方法同上：
``` sh
ssh target
ssh 10.10.0.1
ssh username@target -p22
ssh username@10.10.0.1 -p22
```
### 参考博客
<https://zhuanlan.zhihu.com/p/74193910>
