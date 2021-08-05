---
layout: post
title: linux nc 监听 TCP/UDP
categories: [linux, nc, listener]
description: linux nc 监听 TCP/UDP
keywords: linux, nc, listener
---

Linux 提供了一个很好的工具用来监控 TCP 或者 UDP 端口

## 安装
`yum install nc -y`

## 监听 TCP 端口与测试
### 监听在tcp的3307端口
`nc -lv 3307`

### 往tcp的3307端口发送消息
`nc localhost 3307`

## 监听 UDP 端口与测试
### 监听在UDP的3307端口
`nc -lvu 3307`

### 往UDP端口发送消息
`echo -n "hello world,UDP" >/dev/udp/localhost/3307`