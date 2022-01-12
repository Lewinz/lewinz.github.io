---
layout: post
title: nginx 配置文件加载顺序
categories: [nginx, config, load]
description: nginx 配置文件加载顺序
keywords: nginx, config, load
---

这几天频繁的创建主机搭建站点，突然碰到个与自己直觉不匹配的情况：

当没有配置默认的虚拟主机时，用一个无法匹配 server_name 的链接访问服务器时，会如何？

我直觉认为会显示错误信息。但现实世界是残酷的，nginx 找了一圈都没有匹配后，会直接路由到它找到的第一个配置的虚拟主机上（前提是监听的端口是一样的）。

何为第一个？是靠虚拟主机的 conf 文件名字的字母表顺序！惊不惊喜？意不意外？？

[博客链接](https://blog.kazaff.me/2018/10/17/nginx%E7%9A%84vhost%E6%96%87%E4%BB%B6%E5%8A%A0%E8%BD%BD%E9%A1%BA%E5%BA%8F/)