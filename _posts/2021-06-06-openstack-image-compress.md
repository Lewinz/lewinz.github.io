---
layout: post
title: openstack 镜像压缩
categories: [openstack,镜像]
description: openstack 镜像压缩
keywords: openstack,镜像
---

#### 使用qemu-img工具

转换镜像格式的命令：`qemu-img convert -f qcow2 -O raw test.qcow2 test.raw`

// 压缩命令是以当前镜像文件基础上 -20G，要事先计算镜像真实大小，压缩至一定空间后上传  
压缩镜像命令:`qemu-img resize test.raw -- -20G`

镜像上传命令:`glance image-create --name Ubuntu --disk-format raw --container-format bare --property os_type="linux" --property os_distro="UbuntuServer16.04-64" --file `

