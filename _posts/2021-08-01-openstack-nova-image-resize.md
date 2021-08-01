---
layout: post
title: nova 虚拟机镜像从创建到文件系统 resize 完整流程
categories: [openstack, nova, image, resize]
description: nova 虚拟机镜像从创建到文件系统 resize 完整流程
keywords: openstack, nova, image, resize
---
## 虚拟机镜像的创建和 resize 流程
nova 创建虚拟机涉及的组件比较多，调用比较复杂，这里只列出跟虚拟机镜像创建相关的流程，方便理清虚拟机状态变化的整个流程。

**nova-api**
``` python
nova.api.openstack.compute.servers.ServersController.create() # 接受创建请求，解析出image_uuid
  nova.compute.api.API.create ()
    nova.compute.api.API._create_instance() # 调用glance api获取image对象
      nova.conductor.api.LocalComputeTaskAPI.build_instances()
        nova.conductor.manager.ConductorManager.build_instances() # 此处虽然接收block_device_mapping参数，但是是为了兼容旧版，没有使用。实际通过nova.objects.BlockDeviceMappingList.get_by_instance_uuid()获取
          nova.compute.rpcapi.ComputeAPI.build_and_run_instance() # 使用cast方法调用nova-compute的build_and_run_instance方法。
```

**nova-compute**
``` python
nova.compute.manager.ComputeManager.build_and_run_instance()
  nova.compute.manager.ComputeManager._do_build_and_run_instance()
    nova.compute.manager.ComputeManager._build_and_run_instance()

      nova.compute.manager.ComputeManager._build_resources()
        nova.compute.manager.ComputeManager._prep_block_device()
          nova.virt.block_device.attach_block_devices()
            nova.virt.block_device.DriverImageBlockDevice.attach()
              nova.volume.cinder.API.create()

      nova.virt.libvirt.driver.LibvirtDriver.spwan()
        nova.virt.libvirt.driver.LibvirtDriver._create_image() # 此处会判断如果不是从volume启动，则调用imagebackend去创建虚拟机镜像
          nova.virt.libvirt.driver.LibvirtDriver._try_fetch_image_cache()
            nova.virt.libvirt.imagebackend.Image.cache()

              nova.virt.libvirt.imagebackend.Rbd.create_image()
                nova.virt.libvirt.imagebackend.Rbd.clone()
                  nova.virt.libvirt.storage.rbd_utils.RBDDriver.clone() # 创建虚拟机镜像，此处如果所使用的image后端不支持clone，或者镜像不可clone（比如rbd中不是raw格式的镜像），会触发异常，create_image调用下面的fetch_image函数
                
                nova.virt.libvirt.utils.fetch_image()
                  nova.virt.images.fetch_to_raw()
                    nova.virt.images.fetch()
                      nova.image.API.download()
                    nova.virt.images.convert_image()
                      nova.virt.images._convert_image() # 将镜像拷贝到本地的/var/lib/instances/_base/目录下，文件名为md5(image).part，然后用qemu-img convert转换为raw格式，名为md5(image).converted，最后重命名为md5(image)
                nova.virt.libvirt.storage.rbd_utils.RBDDriver.import_image() # 这一步是在clone失败，执行fetch_image的情况下，判断虚拟机镜像不存在，执行import_image将fetch的镜像导入到RBD后端作为虚拟机镜像。

                nova.virt.libvirt.storage.rbd_utils.RBDDriver.resize() # 调整虚拟机镜像大小

              nova.virt.libvirt.imagebackend.Rbd.resize_image() # 调整虚拟机镜像大小，RBD后端实际上在create_image时已经resize了，不会执行这一步，这里应该是为了确保其他后端能够正确设置虚拟机镜像的大小
```

为了便于分析，用 graphviz 画了在 nova-compute 的调用关系图：

![openstack_nova_image_resize](https://cdn.jsdelivr.net/gh/Lewinz/lewinz.github.io@master/images/posts/openstack_nova_image_resize.png)

注：存储后端用的是 Ceph，所以调用的后端代码是 nova.virt.libvirt.imagebackend.Rbd，如果 nova 使用了不同的后端，比如本地的 qcow2 镜像、raw 镜像、lvm 等，只需要对照 nova.virt.libvirt.imagebackend 中提供的对应实现，出入不会太大，因为它们都继承 nova.virt.libvirt.imagebackend.Image，有相同的接口。

至此，虚拟机的镜像已经创建完毕，并且 resize 为 flavor 所设置的大小。后面是虚拟机启动后，resize 分区和文件系统的过程。

一般虚拟机镜像中会安装 cloud-init 或者配置启动脚本来对虚拟机做初始化配置。在 cloud-init 或启动脚本中调用 growpart 和 resizefs 来完成分区和文件系统的扩容。

## 分区的 resize
cloud-init 支持使用 growpart 和 gpart 对分区进行扩容，时配置的 mode 而定，默认会按顺序检测系统中是否安装了这两个工具，使用第一个找到的。

growpart 是 AWS 的扩展分区工具，它分别使用 sfdisk 和 sgdisk 对 MBR 和 GPT 分区表操作，先将分区表导出，然后改写分区的其实扇区位置，最后将改写后的分区表导入，完成分区的扩容。
``` sh
# growpart [diskdev] [partnum]
```
gpart 是 FreeBSD 推出的磁盘管理工具，GPT 分区表将 metadata 的主本保存在硬盘的开始，将副本保存在硬盘的末尾，所以当虚拟机镜像被扩容，相当于硬盘的容量变大，在 GPT 看来末尾的 metadata 副本丢失了，需要先执行 recover 命令恢复，然后再进行扩容。
``` sh
# gpart recover [diskdev]
# gpart resize -i [partnum] [diskdev]
```

## 文件系统的 resize
cloud-init 通过依次尝试解析 /proc/$$/mountinfo、/etc/mtab 和 mount 命令的输出，来获取根目录所挂载的分区和文件系统格式。

针对不通的文件系统，使用不同的命令扩容：
``` sh
# resize2fs [devpth]    # ext文件系统
# xfs_growfs [devpth]    # xfs文件系统
# growfs [devpth]        # ufs文件系统
# btrfs filesystem resize max [mount_point]    # btrfs文件系统
```