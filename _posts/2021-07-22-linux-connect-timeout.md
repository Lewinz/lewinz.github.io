---
layout: post
title: Linux ssh 连接超时断开问题
categories: [linux, ssh, timeout]
description: Linux ssh 连接超时断开问题
keywords: linux, ssh, timeout
---

## 现象
在使用 `ssh` 连接 `Linux` 之后，搁置一段时间会自动断开，`ctrl + c` 也没反应。

## 原因
在 iptables 的一些 NAT 配置说明里有提到 ——

> 4.3.6 State match 状态匹配扩展要有内核里的连接跟踪代码的协助，因为它是从连接跟踪机制中得到包的状态的。这样我们就可以了解连接所处的状态。它几乎适用于所有的协议，包括那些无状态的协议，如 ICMP 和 UDP。针对每个连接都有一个缺省的超时值，如果连接的时间超过了这个值，那么这个连接的记录就被会从连接跟踪的记录数据库中删除，也就是说连接就不再存在了。这个 match 必须有 - m state 作为前提才能使用。状态机制的详细内容在章节状态机制中。[2]

> NAT firewalls like to time out idle sessions to keep their state tables clean and their memory footprint low.
> NAT 防火墙喜欢对空闲的会话进行超时处理，以确保它们状态表的干净和内存的低占用率。
> Some firewalls are nice, and let you idle for up to a day or so; some are gestapo and terminate your session after 5 minutes.
> 一些防火墙比较友好，允许你的空闲会话时间为一天甚至超过一天；另一些却如盖世太保，5 分钟空闲就终止你的会话。[3]

通过这段描述我们就比较能大致想到断开的原因了 ——

通过 ssh 连接后，客户端和服务端长时间没响应时，在两方机器设置中均没任何限制，但在各自的防火墙，或是中转网络连接路由的防火墙中，出现了「闲置超时断开」的缺省机制！

## 解决方案
### 修改服务端配置
`vi ~/.ssh/config`

``` sh
TCPKeepAlive yes #表示 TCP 保持连接不断开
ClientAliveInterval 300 #指定服务端向客户端请求消息的时间间隔，单位是秒，默认是 0，不发送。设置个 300 表示 5 分钟发送一次（注意，这里是服务端主动发起），然后等待客户端响应，成功，则保持连接。
ClientAliveCountMax 3 #指服务端发出请求后客户端无响应则自动断开的最大次数。使用默认给的 3 即可。
#注意：TCPKeepAlive 必须打开，否则直接影响后面的设置。ClientAliveInterval 设置的值要小于各层防火墙的最小值，不然，也就没用了。
```
注意：最后要重启 sshd 服务才生效
`sudo /etc/init.d/ssh restart`

### 修改客户端配置（推荐！！）
`vi ~/.ssh/config`

``` sh
Host * #表示需要启用该规则的服务端（域名或 ip）
  ServerAliveInterval 60 #表示没 60 秒去给服务端发起一次请求消息（这个设置好就行了）
  ServerAliveCountMax 3 #表示最大连续尝试连接次数（这个基本不用设置）
```

### 修改连接工具的配置
通过改变连接工具的一些默认配置，把 keepalive 的配置打开起来即可：

- secureCRT：会话选项 - 终端 - 反空闲 - 发送 NO-OP 每 xxx 秒，设置一个非 0 值。
- putty：Connection - Seconds between keepalive (0 to turn off)，设置一个非 0 值。
- iTerm2：profiles - sessions - When idle - send ASCII code.
- XShell：session properties - connection - Keep Alive - Send keep alive message while this session connected. Interval [xxx] sec.

当然，用这个办法的副作用也是有的，比如 iTerm2 会出现一些并不想输入的字符、vim 会有些多余字符插入等等，这些情况就按个人的需要酌情取舍了。

### 连接参数 - o
`ssh -o ServerAliveInterval=30 user@host`

## 扩展
### linux hosts 的 allow 和 deny
网络防火墙是阻挡非授权主机访问网络的第一道防护，但是它们不应该仅有一道屏障。

Linux 使用了两个文件 `/etc/host.allow` 和 `/etc/hosts.deny`，根据网络请求的来源限制对服务的访问。

host.allow 文件列出了允许连接到一个特定服务的主机，而 hosts.deny 文件则负责限制访问。

不过，这两个文件只控制对有 hosts_access 功能的服务（如 xinetd 所管理的那些服务、sshd 和某些配置的 sendmail）的访问。

**`/etc/hosts.allow`文件格式**  
``` sh
#
# hosts.allow This file describes the names of the hosts which are
# allowed to use the local INET services, as decided
# by the ‘/usr/sbin/tcpd’ server.
#
sshd:210.13.218.*:allow
sshd:222.77.15.*:allow
```
以上写法表示允许 210 和 222 两个 ip 段连接 sshd 服务（这必然需要 hosts.deny 这个文件配合使用），当然:allow 完全可以省略的。

当然如果管理员集中在一个 IP 那么这样写是比较省事的  
all:218.24.129.110 // 他表示接受 110 这个 ip 的所有请求！

**`/etc/hosts.deny` 文件格式**
``` sh
#
# hosts.deny This file describes the names of the hosts which are
# *not* allowed to use the local INET services, as decided
# by the ‘/usr/sbin/tcpd’ server.
#
# The portmap line is redundant, but it is left to remind you that
# the new secure portmap uses hosts.deny and hosts.allow. In particular
# you should know that NFS uses portmap!
sshd:all:deny
```
注意看：`sshd:all:deny` 表示拒绝了所有 sshd 远程连接。:deny 可以省略。

所以：当 hosts.allow 和 host.deny 相冲突时，以 hosts.allow 设置为准。

注意修改完后：  
`service xinetd restart`  
才能让刚才的更改生效。

/etc/hosts.allow（允许）和 /etc/hosts.deny（禁止）这两个文件是 tcpd 服务器的配置文件  
tcpd 服务器可以控制外部 IP 对本机服务的访问  
linux 系统会先检查 /etc/hosts.deny 规则，再检查 /etc/hosts.allow 规则，如果有冲突 按 /etc/hosts.allow 规则处理

比如：
1. 禁止所有 ip 访问 linux 的 ssh 功能  
可以在 /etc/hosts.deny 添加一行 sshd:all:deny

2. 禁止某一个 ip（192.168.11.112）访问 ssh 功能  
可以在 /etc/hosts.deny 添加一行 sshd:192.168.11.112

3. 如果在 /etc/hosts.deny 和 /etc/hosts.allow 同时 有 sshd:192.168.11.112 规则，则 192.168.11.112 可以访问主机的 ssh 服务