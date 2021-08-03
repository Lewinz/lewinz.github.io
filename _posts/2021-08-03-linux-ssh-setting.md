---
layout: post
title: Linux ssh 相关问题
categories: [linux, ssh]
description: Linux ssh 相关问题
keywords: linux, ssh
---

## ssh client 设置轮询导致 ssh 禁用
``` sh
# vi /etc/hosts.allow
sshd:all
```

## ssh 请求连接时间长
### 使用了 DNS 反查导致耗时
``` sh
# vi /etc/ssh/sshd_config

UseDNS no
```

默认情况下会有一行被注释的配置 `# UseDNS yes`, 但是 ssh 缺省情况下默认是 yes ，所以需要显式配置为 no

这个配置会导致 ssh 在 dns 解析的时候，如果 dns 中没有域名解析记录，会等待 dns 服务器超时返回。

### Kerberos 方式验证导致耗时
``` sh
# vi /etc/ssh/sshd_config

GSSAPIAuthentication no
```

一般 ssh 依次进行的认证方式是 publickey, gssapi-keyex, gssapi-with-mic, password，一般我们常用的是 publickey、password，但是 gssapi（基于 Kerberos） 每次验证还是会尝试，非常耗时，修改 GSSAPIAuthentication 配置可关闭 gssapi 验证过程。

## 重启
`service sshd restart`