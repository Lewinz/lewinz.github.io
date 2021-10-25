---
layout: post
title: Apache http server
categories: [apache, http, server]
description: Apache http server
keywords: apache, http, server
---
## Apache 服务概览
- 软件包： `httpd`, `httpd-devel`, `httpd-manual`
- 服务类型：由 `systemd` 启动的守护进程
- 配置单元： `/usr/lib/systemd/system/httpd.service`
- 守护进程： `/usr/sbin/httpd`
- 端口： 80 (http), 443 (https)
- 配置： `/etc/httpd/`
- Web 文档： `/var/www/html/`

Apache 日志记录目录：`/var/log/httpd/`  
该目录下有两种文件：
``` sh
access_log      # 记录客户端访问Apache的信息，比如客户端的ip
error_log       # 记录访问页面错误信息
```

Apache 服务启动的记录日志：
``` sh
/var/log/messages   # 这个日志是系统的大集合
```

### CentOS

**安装 Apache**
``` sh
yum install httpd -y
```

**设置 httpd 服务开机启动**
``` sh
$ systemctl enable httpd
Created symlink from /etc/systemd/system/multi-user.target.wants/httpd.service to /usr/lib/systemd/system/httpd.service.
```

**启动 Apache**
systemctl start httpd

**查看 Apache 的状态**
systemctl status httpd

### Ubuntu
**安装 Apache**
``` sh
apt install apache2 -y
```

**设置 httpd 服务开机启动**
``` sh
$ systemctl enable httpd
Created symlink from /etc/systemd/system/multi-user.target.wants/httpd.service to /usr/lib/systemd/system/httpd.service.
```

**配置 UFW 防火墙**
``` sh
$ ufw allow 'Apache'
```

**启动 Apache**
systemctl start httpd

**查看 Apache 的状态**
systemctl status httpd

即可通过服务器 IP 直接访问，端口分别是 80 (http)、443 (https)

## Apache 命令
### 停止 Apache 服务
`systemctl stop httpd`

### 启动 Apache 服务
`systemctl start httpd`

### 重启 Apache 服务
`systemctl restart httpd`

### 禁用 Apache 开机自启动
`systemctl disable httpd`

### Apache 开机自启动
`systemctl enable httpd`

### 更改配置后重新加载 Apache 服务
`systemctl reload httpd`

## Apache 的主配置文件
`/etc/httpd/conf/httpd.conf` 安装完后就可以到 Apache 的默认目录 `/var/www/html` 添加一个简单的 index.html

``` html
<!DOCTYPE html>
<head>
        <title>test</title>
</head>
<body>
        <p>Hello World!</p>
</body>
</html>
```
