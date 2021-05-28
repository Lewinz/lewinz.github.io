---
layout: post
title: openstack 快照与镜像的区别
categories: [openstack,快照,镜像]
description: openstack 快照与镜像的区别
keywords: openstack,快照,镜像
---

OpenStack的快照对于虚拟机来说，就是镜像。因为Openstack是采用创建镜像的方式创建快照，而不是通过正统的virsh和其他快照方式创建快照函数。虚拟机快照做完后以镜像形式存在于glance（镜像组件）中。

## libvirt 主流快照实现
采用virDomainSnapshotCreateXML()函数(CLI为virsh snapshot-create)。新建的快照与虚拟机有关联：若为内置快照，快照信息和虚拟机存在同一个qcow2镜像中；若为外置快照，新建一个qcow2文件，原虚拟机的disk将变为一个read only的模板镜像，新qcow2镜像仅记录与2.模板镜像的差异数据。这种快照包含快照链信息，可保留disk和ram信息，可回滚至快照点。

## openstack快照实现
openstack并未采用virDomainSnapshotCreateXML()来实现快照，而是单纯的对虚拟机镜像做转换和拷贝，生成一个与虚拟机无关联的镜像，最后上传至glance中。这种快照不包含快照链信息，只保留disk信息，无法回滚至快照点，只能采用该快照镜像创建一个新的虚拟机。

## 限制与缺点

没有快照链信息，不支持revert恢复虚拟机到某一个快照点  

只对系统盘进行快照  

不支持内存快照，不支持同时对虚拟机和磁盘做快照  

需要用户进行一致性操作  

不支持含元数据导出  

不支持含元数据导入  

只支持虚拟机全量数据快照（与快照的实现方式有关，因为是通过image进行保存的）  

过程较长（需要先通过存储快照，然后抽取并上传至glance)  

快照以Image方式保存，而非以cinder卷方式保存，无法充分利用存储本身能力加快快照的创建和使用  

当前限制openstack的虚拟机快照只快照root盘，不快照内存/CPU状态以及挂载磁盘。挂载磁盘需要事先卸载磁盘(数据盘），然后进行快照，然后再挂载磁盘  

没有快照链信息  

**参考博客**：  
<https://blog.csdn.net/zhongbeida_xue/article/details/82257461>
<https://blog.csdn.net/liukuan73/article/details/46457439>