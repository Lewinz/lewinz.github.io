---
layout: post
title: 主流软件负载均衡器对比 (LVS、Nginx、HAproxy)
categories: [lvc, nginx, haproxy]
description: 主流软件负载均衡器对比 (LVS、Nginx、HAproxy)
keywords: lvc, nginx, haproxy
---

## 负载均衡的三种实现方式
- 基于 DNS 负载均衡
  - 直接通过 DNS 来实现负载均衡。优点是非常简单，缺点是调整后不知道啥时生效 (当然正常情况下几十分钟，长的也可能更长)
- 基于硬件负载均衡
  - 购买硬件，也就是我们常常说的 F5（F5 Network Big-IP），不过 F5 就贵一般来说单台硬件也得几十万块，要是搞个双机，多机就更贵了
- 基于软件负载均衡
  - 基于软件的方式也非常多，类似几个主流 LVS、Nginx、HAproxy (当然 IBM 也有个 HIS)，接下来就针对以下几种具体说明：

## 三大主流软件负载均衡器对比 (LVS、Nginx、HAproxy)
### LVS
1. 抗负载能力强，性能高，能达到 F5 的 60%，对内存和 CPU 资源消耗比较低
2. 工作在网络 4 层，通过 VRRP 协议 (仅作代理之用)，具体的流量是由 linux 内核来处理，因此没有流量的产生。
3. 稳定，可靠性高，自身有完美的热备方案 (Keepalived+lvs)
4. 不支持正则处理，不能做动静分离。
5. 支持多种负载均衡算法：rr (轮询)，wrr (带权轮询)、lc (最小连接)、wlc (带权最小连接)
6. 配置相对复杂，对网络依赖比较大，稳定性很高。
7. LVS 工作模式有 4 种：
  - nat 地址转换
  - dr 直接路由
  - tun 隧道
  - full-nat
8. 工作在网络 4 层，相对性能上较高 (网络的七层模式：物理层、数据链路层、网络层、传输层、会话层、表示层、应用层)

### Nginx
1. 工作在网络 7 层，可以针对 http 应用做一些分流的策略，比如针对域名，目录结构
2. Nginx 对网络的依赖较小，理论上能 ping 通就能进行负载功能
3. Nginx 安装配置比较简单，测试起来很方便
4. 也可以承担较高的负载压力且稳定，nginx 是为解决 c10k 问题而诞生的
5. 对后端服务器的健康检查，只支持通过端口来检测，不支持通过 url 来检测
6. Nginx 对请求的异步处理可以帮助节点服务器减轻负载压力
7. Nginx 仅能支持 http、https 和 Email 协议，这样就在适用范围较小。
8. 不支持 Session 的直接保持，但能通过 ip_hash 来解决。对 Big request header 的支持不是很好。
9. Nginx 还能做 Web 服务器即 Cache 功能。


### HAProxy
1. 支持两种代理模式：TCP（四层）和 HTTP（七层），支持虚拟主机；
2. 能够补充 Nginx 的一些缺点比如 Session 的保持，Cookie 的引导等工作
3. 支持 url 检测后端的服务器出问题的检测会有很好的帮助。
4. 更多的负载均衡策略比如：动态加权轮循 (Dynamic Round Robin)，加权源地址哈希 (Weighted Source Hash)，加权 URL 哈希和加权参数哈希 (Weighted Parameter Hash) 已经实现
5. 单纯从效率上来讲 HAProxy 更会比 Nginx 有更出色的负载均衡速度。
6. HAProxy 可以对 Mysql 进行负载均衡，对后端的 DB 节点进行检测和负载均衡。
7. 支持负载均衡算法：Round-robin（轮循）、Weight-round-robin（带权轮循）、source（原地址保持）、RI（请求 URL）、rdp-cookie（根据 cookie）
8. 不能做 Web 服务器即 Cache。

## 三大主流软件负载均衡器适用业务场景
1. 网站建设初期，可以选用 Nginx、HAProxy 作为反向代理负载均衡 (流量不大时，可以不选用负载均衡)，因为其配置简单，性能也能满足一般业务场景。如果考虑到负载均衡器是有单点问题，可以采用 Nginx+Keepalived/HAproxy+Keepalived 避免负载均衡器自身的单点问题。
2. 网站并发到达一定程度后，为了提高稳定性和转发效率，可以使用 lvs，毕竟 lvs 比 Nginx/HAProxy 要更稳定，转发效率也更高。
注：nginx 与 HAProxy 比较：nginx 只支持七层，用户量最大，稳定性比较可靠。Haproxy 支持四层和七层，支持更多的负载均衡算法，支持 session 等。

## 衡量负载均衡器好坏的几个重要的因素：
1. 会话率 ：单位时间内的处理的请求数
2. 会话并发能力：并发处理能力
3. 数据率：处理数据能力

## 负载均衡的策略
- 轮询策略
- 负载度策略
- 响应策略
- 哈希策略