---
layout: wiki
title: Linux 磁盘分区、格式化、挂载
categories: [linux,fdisk,cli]
description: linux,fdisk,cli
keywords: Linux 磁盘分区、格式化、挂载
---

## fdisk
查看磁盘使用情况和磁盘分区

### 补充说明
`fdisk` 命令 用于观察硬盘实体使用情况，也可对硬盘分区。它采用传统的问答式界面，而非类似 DOS fdisk 的 cfdisk 互动式操作界面，因此在使用上较为不便，但功能却丝毫不打折扣。

### 语法
`fdisk(选项)(参数)`

#### 选项
``` shell
-b <大小>             扇区大小 (512、1024、2048 或 4096)
-c[=<模式>]           兼容模式：“dos”或“nondos”(默认)
-h                    打印此帮助文本
-u[=<单位>]           显示单位：“cylinders”(柱面) 或“sectors”(扇区，默认)
-v                    打印程序版本
-C <数字>             指定柱面数
-H <数字>             指定磁头数
-S <数字>             指定每个磁道的扇区数
```

#### 参数
设备文件：指定要进行分区或者显示分区的硬盘设备文件。

### 示例
**磁盘情况查询**
``` shell
[root@test-lewin ~]# fdisk -l

磁盘 /dev/vda：42.9 GB, 42949672960 字节，83886080 个扇区
Units = 扇区 of 1 * 512 = 512 bytes
扇区大小 (逻辑/物理)：512 字节 / 512 字节
I/O 大小 (最小/最佳)：512 字节 / 512 字节
磁盘标签类型：dos
磁盘标识符：0x000940fd

   设备 Boot      Start         End      Blocks   Id  System
/dev/vda1   *        2048    83886046    41941999+  83  Linux

磁盘 /dev/vdb：53.7 GB, 53687091200 字节，104857600 个扇区
Units = 扇区 of 1 * 512 = 512 bytes
扇区大小 (逻辑/物理)：512 字节 / 512 字节
I/O 大小 (最小/最佳)：512 字节 / 512 字节
磁盘标签类型：dos
磁盘标识符：0x57cc28a8

   设备 Boot      Start         End      Blocks   Id  System
/dev/vdb1            2048      821247      409600   83  Linux
/dev/vdb2          821248    63735807    31457280    5  Extended
/dev/vdb5          823296     2920447     1048576   83  Linux
/dev/vdb6         2922496     5019647     1048576   83  Linux
```

**选择磁盘进行操作**
``` shell
[root@test-lewin ~]# fdisk /dev/vda
欢迎使用 fdisk (util-linux 2.23.2)。

更改将停留在内存中，直到您决定将更改写入磁盘。
使用写入命令前请三思。

命令 (输入 m 获取帮助)：
```

**使用命令进行磁盘操作**

``` shell
a   toggle a bootable flag
b   edit bsd disklabel
c   toggle the dos compatibility flag
d   delete a partition                          删除分区
g   create a new empty GPT partition table
G   create an IRIX (SGI) partition table
l   list known partition types
m   print this menu
n   add a new partition                         创建分区
o   create a new empty DOS partition table
p   print the partition table
q   quit without saving changes
s   create a new empty Sun disklabel
t   change a partition's system id
u   change display/entry units
v   verify the partition table
w   write table to disk and exit
x   extra functionality (experts only)
```

**格式化分区**
`mkfs.ext2 /dev/sdb1`

**查看磁盘分区文件系统格式**
`parted -l`

**挂载分区**
`mount /dev/sdb1 /oracle`

**开机自动挂载分区**
修改 `/etc/fstab` 文件

``` shell
[root@localhost ~]# vim /etc/fstab

/dev/VolGroup00/LogVol00 /                       ext3    defaults        1 1
LABEL=/boot             /boot                   ext3    defaults        1 2
tmpfs                   /dev/shm                tmpfs   defaults        0 0
devpts                  /dev/pts                devpts  gid=5,mode=620  0 0
sysfs                   /sys                    sysfs   defaults        0 0
proc                    /proc                   proc    defaults        0 0
/dev/VolGroup00/LogVol01 swap                   swap    defaults        0 0
/dev/sdb1               /oracle                 ext2    defaults        0 0
/dev/sdb6               /web                    ext3    defaults        0 0
```

**可能出现的问题**  
格式化分区的时候报错：`The device apparently does not exist; did you specify it correctly?`  
错误出现大多是因为之前操作的分区尚在使用，保存操作时未成功，分区操作还未写入分区表，使用 `partprobe` 命令将信息推入分区表即可解决。

