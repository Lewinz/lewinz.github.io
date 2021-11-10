---
layout: post
title: openstack 秒级创建虚机方案
categories: [openstack, nova]
description: openstack 秒级创建虚机方案
keywords: openstack, nova
---

OpenStack 管理虚拟机生命周期的组件是 Nova，Nova 创建虚拟机从后端存储类型分为本地 LVM 存储和远程分布式存储（例如：Ceph/SheepDog/GlusterFS），从启动方式一般分为镜像启动和卷启动两大类，按启动方式和存储后端可以有 4 种组合

## 本地 LVM + 镜像启动

此种方式，虚拟机镜像事先一般也会存放在远端的分布式存储上（Ceph 或 Swift）。当在计算节点首次创建虚拟机时，会从远端分布式存储下载镜像到计算节点做格式转换并缓存为 base-image，然后复制完整的镜像到 /var/lib/nova/instances/{instance-uuid} 目录下作为系统盘，耗时跟镜像大小和带宽有关，一般约数分数以上。

如果在相同的计算节点上第二次以相同的镜像创建虚拟机，因为已经有了镜像缓存，不需要再到远端分布式存储下载镜像，直接从本地计算节点拷贝镜像到虚拟机启动目录作为系统盘，耗时跟镜像大小有关，一般也得数分钟以上。

## 本地 LVM + 卷启动

此种方式，虚拟机镜像事先一般也会存放在远端的分布式存储上（Ceph 或 Swift）。当在计算节点首次创建虚拟机时，会从远端分布式存储下载镜像到计算节点做格式转换并缓存为 base-image，然后调用 cinder 在控制节点以 base-image 创建可 bootable 的卷，最后以该卷启动虚拟机。耗时跟镜像大小和带宽有关，一般也得数分钟以上。

如果在相同的计算节点上第二次以相同的镜像从卷启动创建虚拟机，因为已经有了镜像缓存，不需要再到远端分布式存储下载镜像，直接调用 cinder 在控制节点以 base-image 创建可 bootable 的卷，最后以该卷启动虚拟机。耗时跟镜像大小和带宽有关，一般也得数分钟以上。

## 远程分布式存储 Ceph + 镜像启动（采用默认配置）

此种方式，虚拟机镜像事先一般也会存放在远端的分布式存储 Ceph 上。当在计算节点首次创建虚拟机时，首先会从远程 Ceph 上下载镜像到该计算节点做格式转换并缓存为 base-image，然后上传 base-image 到远程 Ceph Rbd 的 pool 中作为系统盘，最后以 CephRbd pool 中的系统盘启动虚拟机。耗时跟镜像大小和带宽有关，一般也得数分钟以上。

在相同的计算节点上第二次以相同的镜像创建虚拟机，因为已经有了镜像缓存，不需要再到远端 Ceph 下载镜像，直接上传 base-image 到远程 Ceph Rbd 的 pool 中作为系统盘，最后以 CephRbd pool 中的系统盘启动虚拟机。耗时跟镜像大小和带宽有关，一般也得数分钟以上。

## 远程分布式存储 Ceph + 卷启动（采用默认配置）

此种方式，虚拟机镜像事先一般也会存放在远端的分布式存储 Ceph 上。当在计算节点首次创建虚拟机时，首先会从远程 Ceph 上下载镜像到该计算节点做格式转换并缓存为 base-image，然后调用 cinder 通过 base-image 在远程 Ceph Rbd 的 pool 中创建可 bootable 的启动卷，最后以 Ceph Rbd pool 中的卷启动虚拟机。耗时跟镜像大小和带宽有关，一般也得数分钟以上。

在相同的计算节点上第二次以相同的镜像以卷启动创建虚拟机，因为已经有了镜像缓存，不需要再到远端 Ceph 下载镜像，直接调用 cinder 通过 base-image 在远程 Ceph Rbd 的 pool 中创建可 bootable 的启动卷，最后以 Ceph Rbd pool 中的卷启动虚拟机。耗时跟镜像大小和带宽有关，一般也得数分钟以上。

## 秒级创建虚拟机优化方案

在优化之前，如果按照上述 4 种组合任一一种来创建虚拟机，如果批量创建几百台虚拟机，因为有镜像的下载、上传或者拷贝流程，整个创建流程会非常耗时，有些会因为接口超时导致失败。

为了达到秒级创建虚拟机的性能，Glance、Cinder 和 Nova 的后端存储统一以 Ceph 作为共享存储。Glance 上传的虚拟机镜像会上传到 Ceph images pool 中，Cinder 创建的卷会保存在 Ceph volumes pool 中，Nova 系统盘保存在 Cephinstances pool 中。

如果是以镜像启动创建虚拟机，在同一个计算节点选择相同的镜像不论是第一次还是第二次创建虚拟机，会直接基于 ceph images pool 的镜像先做 snapshot，然后基于该 snapshot 进行 clone（copy on write）到 Cephinstances pool，最后以该系统盘启动虚拟机。利用 ceph 写时复制特性，不存在镜像的上传、下载和完整拷贝，所以创建速度非常快，可以达到秒级。

如果是以卷启动创建虚拟机，在同一个计算节点选择相同的镜像不论是第一次还是第二次创建虚拟机，会直接基于 ceph images pool 的镜像先做 snapshot，然后调用 cinder 基于该 snapshot 进行 clone（copy on write）到 Ceph volumes pool，最后以该卷启动虚拟机。利用 ceph 写时复制特性，不存在镜像的上传、下载和完整拷贝，所以创建速度非常快，可以达到秒级。

## 秒级创建虚拟机的优化步骤

1. 在 controller 控制节点上修改 /etc/glance/glance-api.conf 镜像配置文件，把 show_image_direct_url 参数设置为 True。

vim /etc/glance/glance-api.conf

![openstack_create_nova_1](https://cdn.jsdelivr.net/gh/Lewinz/lewinz.github.io@master/images/posts/openstack_create_nova_1.png)

注意：一定要在【DEFAULT】下添加。

然后利用命令 serviceopenstack-glance-api restart 重启镜像管理服务。

2. 转换镜像格式，通过 glance 上传的镜像一定要是 raw 格式。

在上传之前需要命令转换好后上传，转换命令：

qemu-img convert -O raw src-img.qcow2dst-img.raw

用命令行转换成 raw 格式，主要是解决如下 no bootable device 问题。

![openstack_create_nova_2](https://cdn.jsdelivr.net/gh/Lewinz/lewinz.github.io@master/images/posts/openstack_create_nova_2.png)

3. 在计算节点上进入 /var/lib/nova/instances/_base/ 目录，清空该目录下所有的缓存镜像。由于后台程序会先检测该目录下有没有缓存镜像，如果有，会把该缓存镜像上传到 ceph 中，如果没有，直接在 ceph 中 clone 镜像。

步骤如下：
``` shell
[root@openstack _base]# cd/var/lib/nova/instances/_base/

[root@openstack _base]# ll

-rw-r--r--. 1 qemu qemu 41126400 Jul 24 01:01d7fca384a7c355afa3b70667b60f04dd08cd6f35

[root@openstack _base]# rm -rfd7fca384a7c355afa3b70667b60f04dd08cd6f35

[root@openstack _base]# ll

total 0
```
 

4. 相关的流程源码如下：

![openstack_create_nova_3](https://cdn.jsdelivr.net/gh/Lewinz/lewinz.github.io@master/images/posts/openstack_create_nova_3.png)

![openstack_create_nova_4](https://cdn.jsdelivr.net/gh/Lewinz/lewinz.github.io@master/images/posts/openstack_create_nova_4.png)

`[root@openstack ~]# vim /usr/lib/python2.7/site-packages/nova/virt/libvirt/imagebackend.py`

![openstack_create_nova_5](https://cdn.jsdelivr.net/gh/Lewinz/lewinz.github.io@master/images/posts/openstack_create_nova_5.png)

`[root@openstack ~]# vim /usr/lib/python2.7/site-packages/nova/virt/libvirt/driver.py`

![openstack_create_nova_6](https://cdn.jsdelivr.net/gh/Lewinz/lewinz.github.io@master/images/posts/openstack_create_nova_6.jpeg)

`[root@openstack ~]# vim/usr/lib/python2.7/site-packages/glance/api/v2/images.py`

![openstack_create_nova_7](https://cdn.jsdelivr.net/gh/Lewinz/lewinz.github.io@master/images/posts/openstack_create_nova_7.jpeg)

## 总结

上述介绍的不同种类的创建虚拟机的组合方式，从核心原理分析其实只有两类。一种是需要完整拷贝镜像，另一种是写时复制（copy on write）。

1. 完整拷贝镜像创建虚拟机

优点：是每个虚拟机独立，不会相互影响。

缺点：存在镜像下载、上传或者拷贝，创建速度慢。

2. 写时复制（copyon write）

优点：每次创建虚拟机都只有很小的增量文件，不存在全量镜像拷贝，创建速度很快。

缺点：以相同的镜像创建的所有虚拟机依赖共同的 base-image，如果 base-image 意外损坏或删除，上层依赖的虚拟机都会受到影响。

速度和安全性往往是一对矛盾体，两种方式需要做一定的权衡。可以在完整拷贝镜像创建虚拟机的方案中提升硬件性能，比如通过高配的磁盘和带宽来降低拷贝镜像的时间。也可以在写时复制（copy on write）的方案中，在虚拟机创建成功后的某个恰当时刻，通过后台执行 ceph 的 rbd flatten 命令断开 base-image 和增量 clone 虚拟磁盘的依赖链，达到每个虚拟机相互独立。