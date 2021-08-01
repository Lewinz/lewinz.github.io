---
layout: post
title: 静态 IP/动态 IP/浮动 IP/虚拟 IP 的区别
categories: [静态 IP,动态 IP,浮动 IP,虚拟 IP]
description: 静态 IP/动态 IP/浮动 IP/虚拟 IP 的区别
keywords: 静态 IP,动态 IP,浮动 IP,虚拟 IP
---

## 名词对应
static ip ==> 静态 IP  
dynamic ip ==> 动态 IP  
floating ip ==> 浮动 IP  
virtual ip ==> 虚拟 IP

## 区别
### 静态 IP 与动态 IP
static ip 就是固定分配的 ip，需要手工管理，非常麻烦。为了减少麻烦，人们发明了 dhcp 协议，来自动为电脑分配 ip，这就是 dynamic ip。

### 浮动 IP
floating ip 跟 dynamic ip 有点像，参考各公有云厂商的弹性 ip。

### 虚拟 IP
但不论 static ip、dynamic ip 还是 floating ip，一个 ip 只能分配给一台电脑。  
在有些情况（比如高可用场景）下我们需要多台电脑共用一个 ip，也就是说一个 ip 「属于」多台电脑。那怎么实现呢？是给两台电脑设置同一个 ip 吗？显然不是，因为为产生 ip 冲突。这就需要 virtual ip。  
比如我们有两台服务器 AAA 和 BBB，它们的 IP 分别是 10.0.0.1 和 10.0.0.2。它们功能相同，提供相同的服务。理论上大家可以直接能过 10.0.0.1 或者 10.0.0.2 来访问 AAA 或 BBB 的服务。但如果某一台机器宕机，就没法访问了。要解决这个问题就需要 virtual ip。

![VIP_1](https://cdn.jsdelivr.net/gh/Lewinz/lewinz.github.io@master/images/posts/VIP_1.jpeg)

首先，我们从 AAA 和 BBB 中选一个作主，另一个作备。然后要求它们互相探测，确保对方都在线。然后给 AAA 和 BBB 同时「分配」一个 virtual ip 10.0.0.100。其他主机需要通过 10.0.0.100 来访问 AAA 或 BBB 提供的服务。

一般来说，其他主机要访问 10.0.0.100 需要通过 ARP 获取对应的 MAC 地址。如果 AAA 和 BBB 同时应答 ARP 请求，就会产生冲突。因为 10.0.0.100 是 virtual ip，所以，只有主服务器 AAA 才能应答。BBB 收到 ARP 请求后发现 AAA 还活着，就自动闭嘴。

![VIP_2](https://cdn.jsdelivr.net/gh/Lewinz/lewinz.github.io@master/images/posts/VIP_2.jpeg)

之后所有访问 10.0.0.100 这个 virtual ip 的请求都会发到 AAA。

如果 AAA 出现故障呢？这个时候其他主机发现 10.0.0.100 不通了，于是发出新的 ARP 请求

![VIP_3](https://cdn.jsdelivr.net/gh/Lewinz/lewinz.github.io@master/images/posts/VIP_3.jpeg)

同时，BBB 也探测不到 AAA，它知道自己的高光时刻来到了，于是 BBB 大声响应 ARP 请求说「向我开炮」。

![VIP_4](https://cdn.jsdelivr.net/gh/Lewinz/lewinz.github.io@master/images/posts/VIP_4.jpeg)

总结下来，virtual ip 就是多主机设置相同 ip，但只有一台主机可以在特定条件下响应 arp 请求。

## 浮动 IP 补充
`What is a floating IP?`  
什么是浮动 IP

`The internet – plainly put – consists of many computers connected by cables, fiber optic cables, and wireless receivers. They exchange data based on a common ‘language'. This common standard is known as the Internet Protocol (IP). Data is arranged in such a way that computers, which understand the common protocol, can interpret it.`

因特网简单来说说是许多计算机由电缆、光纤、无线接收器连接组成的网络。网络中设备间数据交互是通过 IP 协议进行，数据以 IP 封装其他计算根据协议才能解析数据。

`An IP address, also referred to as an 'IP', makes digital devices detectable in a network. It is a crucial prerequisite so that electronic data packets can be delivered reliably. The devices communicate with one another, for example, over the internet. The IP address ensures that data from the sender reaches the correct recipient – for example, from a web browser to a web server or vice versa. An IP address can be assigned to both single and multiple devices at the same time. Likewise, a single device can have multiple IP addresses at the same time.`

IP 地址简称 IP，数字设备以此作为身份标识，才能被其他设备发现和识别。IP 地址是设备间交互数据的先决条件。IP 地址保证数据的发送者发出的数据能正确到相应的接收者，反之也是如此。一个 IP 地址可分配多个设备，一个设备也可拥有多个 IP。

`However, in order to be able to understand exactly what a floating IP is, you first need to know the difference between dynamic and static IP addresses.`

为了更好的弄明白什么是浮动 IP，首先需要搞明静态 IP 和动态 IP 之间的区别。

`Dynamic IP`

动态 IP

`When a computer connects to the internet, in most cases the Internet Service Provider (ISP) assigns a dynamic IP address to it. Dynamic IP addresses are the most cost-effective standard for users and providers. They are characterized by the fact that they are only assigned temporarily and change after a certain time, which is either fixed (e.g. for 24 hours), or is irregular. The user then receives a new dynamic IP address for their computer from the respective internet service provider and the previous address will then be signed to a different user.`

当一个计算机接入到互联网，网络服务接入商会分配一个动态 IP 给这台计算机。动态 IP 对用户和接入商来说都是最经济的。动态 IP 是不固定的，过一段时间会变。过一段时间用户的电脑会收到一个新的 IP 地址，原来的 IP 地址有可以已经分配给了别的电脑。

`Static IP`

静态 IP

`A static IP, on the other hand, is a fixed address and is permanently assigned to a device. Static IP addresses are found mainly in the web server or e-mail server area, or wherever offers or website content must be accessible via a fixed URL , so that users or processes can (re)find them without any problems. Computers in a network or peripheral devices (such as printers) have fixed IPs, so that the individual devices within the network can easily communicate with one another.`

从一个方面来说，静态 IP 是 一个固定的 IP 地址，被永久的分配给一个设备。静态 IP 多用于 Web 服务器或者电子邮件服务器或者一个网站。这些网站通过一个固定的 URL 进行访问，用户可以通过 URL 找到 IP 地址。在一个网络中的计算机或者外围设备都有固定的 IP，这样设备间才能很容易的交互数据。

`So that users don’t have to remember complex numbers, it’s possible to assign a domain name to a static IP address e.g. www.example.org. The numerical IP, the 'connection number' of a device in the network, is therefore translated into a name that can easily be remembered. This is generally only reserved for static IPs. It doesn’t make much sense for dynamic IPs since the user changes so frequently.`

给一个静态 IP 分配域名后，用户就不需要记住复杂的 IP 地址。使用域名 IP 地址被转成了容易记忆的名字。域名一般只用于静态 IP，因动态 IP 频繁变动使用域名意义不大。

`Floating IP – definition`

浮动 IP

`A floating IP is usually a public, routable IP address that is not automatically assigned to an entity. Instead, a project owner assigns them to one or more entities temporarily. The respective entity has an automatically assigned, static IP for communication between instances in a private, non-routable network area, as well as via a manually assigned floating IP. This makes the entity’s services outside a cloud or network recognizable and therefore achievable.`

一个浮动 IP 通常是一个公开的、可以路由到的 IP 地址，并且不会自动分配给实体设备。项目管理者临时分配动态 IP 到一个或者多个实体设备。这个实体设备有自动分配的静态 IP 用于内部网间设备的通讯。这个内部网使用私有地址，这些私有地址不能被路由到。通过浮动 IP 内网实体的服务才能被外网识别和访问。

`In appropriately configured failover scenarios, an IP 'floats' to another active unit in the network so that it can take on the function of a dormant entity without a time delay, and can then answer incoming requests.`

在一个配置好浮点 IP 的切换场景是，IP 地址飘到网络中的另一台设备。新设备无延迟的接替当掉的设备，并对外提供服务。

`How is a floating IP generated?`

浮点 IP 是如何产生的？

`Users obtain floating IPs for their projects from different pools that the system administrator configures and provides as server resources. As soon as a user receives a floating IP, they become the 'owner'. They can assign it to an entity, remove it, and then assign it to another at any time. Even if an entity is terminated, the user does not 'lose' the associated floating IP. It remains as a resource and can still be assigned to another entity when needed.`

用户从系统管理员配置的资源池中为他们的项目获取 IP 地址。一旦用户获取一个浮动 IP，就拥有了这个 IP。他可以分配这个 IP 到一个计算实体，或者在任一时间移除分配给其他设备。就算设备关机，用户还拥有他属于他的浮动 IP。浮动 IP 就像一种资源，当需要时可以分配给其他设备。

`A major reason for using several parallel floating IP pools is that each pool can be operated by another internet service provider or can also be assigned by other external networks. This ensures that the connectivity or availability is maintainable even if an internet service provider should fail due to a malfunction.`

使用多个平行的浮动 IP 主要是为了防止当其中的一个不可能用时使用其他地址以保证服务的正常可用。

`When are floating IPs used?`

什么时候会用浮动 IP

`Maximum availability is one of the key factors in every production environment. In the communication network, however, a single error can cause applications to fail. Developers do sleep better knowing that their applications are designed to withstand any conceivable error scenarios. The goal is to provide a highly available piece of infrastructure with minimal downtime.`

最大的可用性是浮动 IP 在生产环境中使用的一个关键因素。在网络中，单个错误可能会导致应用的不可用。如果系统能成功应对任何可以想到的应用场景，开发人员就可以安枕无忧。浮动 IP 的目标就最小当机下提供高可用的基础设施。

`A floating IP can serve as a flexible load balancing address, helping to balance peak loads by distributing incoming network traffic to different network nodes. Network nodes are devices which connect two (or more) transmission paths of a telecommunication network. As with a computer that distributes workflows across multiple processors, load balancing also handles large amounts of simultaneous requests or more complex calculations by splitting the load across multiple parallel systems.`

浮动 IP 可以用于灵活的负载均衡地址，用于高峰时的负载均衡，分流访问流量到不同的网络节点。网络节点是连接到两个或者多个通讯网络。就像一台电脑分配工作流到不同的处理器，负载均衡大量并发的请求或者复杂的计算分配到并行系统中。

`Failover and switchover`

故障恢复和地址切换

`If a primary load balancer or a central application server in a cluster fails on one side, a floating IP can be immediately assigned a redundant application server or a secondary load balancer in a correspondingly configured system. The IP 'floats' to the active unit, which immediately carries out the desired processes. An unplanned change between network services is referred to as 'failover'. This kind of protection is especially recommended for critical applications.`

如果一个主要的负载均衡器或者集群中一个主要的业务服务器当掉，浮动 IP 立即被分配到冗余的应用器或者备用的负载均衡器，这些都需要提前配置好。当浮动 IP 飘到一个活动单元，活动单元立即承担相应的业务。故障恢复指的是非计划的网络服务切换。这种特别的保护推荐用于关键应用。

`A planned change from a primary to a secondary system is referred to as a 'switchover'. The targeted transmission of services is not triggered by errors, but is usually controlled by a system administrator. A classic reason for a switchover is, for example, routine maintenance of the primary or secondary systems where a parallel instance temporarily takes over its function.`

一个有计划的从主切换到从，通常被称为切换。切换不是由故障或者错误引起，而是系统管理员操作完成。切换的典型应用场景时，当对一个系统时行例常的维护时，由另一服务接替他的功能。

`What advantages does a floating IP offer?`

浮动 IP 优点

`One of the main advantages of floating IPs is their flexibility – the free and needs-oriented assignability. Floating IPs are therefore suitable for use in both failover and switchover environments – for example, for performing upgrades of applications or entire sites with minimal downtime. While an upgrade is applied to one entity, another one takes on the traffic. Once the upgrade has been successfully completed, the traffic is redirected to the updated unit.`

浮动 IP 的主要优点是灵活，自由的根据需要分配。浮动 IP 即适用于故障恢复又适用于服务切换。比如对某个应用或者整个站点的升级，并能保证对业务有最小的影响。当对一个应用升级时，另一个应用分配输入流量。一旦升级完成，流量会被重新导入到升级节点。

`Another advantage: even if several or even many different entities are concealed behind a service being offered, the floating IP appears on the surface to users (who make use of the service) rather than the server’s IP that offers the respective service.`

另一个优点是：浮动 IP 对外提供统一的 IP，而不是实际对外提供服务的 IP 地址。