---
layout: post
title: 物理机安装 Ubuntu
categories: [install,ubuntu]
description: 物理机安装 Ubuntu
keywords: install,ubuntu
---
## 启动 U 盘制作
### 下载官方镜像
目前稳定的长期支持的版本是 Ubuntu 16.04.3 LTS 。

下载地址：  
官网：<https://www.ubuntu.com/download/desktop>  
镜像：<http://mirrors.163.com/ubuntu-releases/16.04.3/>  
选择国内的镜像源下载

### 下载 Rufus
![ubuntui_install](https://cdn.jsdelivr.net/gh/Lewinz/lewinz.github.io@master/images/posts/ubuntu_install_1.jpg)

| 设置项  | 说明  |
| - | - |
| 设备 | 选择你的 U 盘，为了避免选错，只插一个 U 盘  |
| 分区方案和目标系统类型 | 如果电脑启动方式是 UEFI 的，则选带 UEFI 的那个；如果是 BIOS 的，则选兼容 BIOS 的  |
| 文件系统 | 默认 FAT32 即可  |
| 簇大小 | 默认即可  |
| 新卷标 | 设置 U 盘的名称，这一项在选择 Ubuntu 的 iso 文件之后会自动修改  |

#### 文件格式差异
- Fat32 文件格式是一种通用格式，任何 USB 存储设备都会预装该文件系统，可以在任何操作系统平台上使用。最主要的缺陷是只支持最大单文件大小容量为 4GB，因此日常使用没有问题，只有在传输大文件时才会显现出缺点。

- exFAT 文件是微软自家创建的用来取代 FAT32 文件格式的新型文件格式，它最大可以支持 1EB 的文件大小，非常适合用来存储大容量文件，还可以在 Mac 和 Windows 操作系统上通用。虽然是微软的技术，苹果批准在系统中使用该文件格式，因此在 Mac 系统中格式化存储设备时会出现 exFAT 文件格式选项。最大的缺点是没有文件日志功能，这样就不能记录磁盘上文件的修改记录。

- NTFS 是微软为硬盘或固态硬盘（SSD）创建的默认新型文件系统，NTFS 的含义是 New Technology File System，它基层了所有文件系统的优点：日志功能、无文件大小限制、支持文件压缩和长文件名、服务器文件管理权限等。最大的缺点是 Mac 系统只能读取 NTFS 文件但没有权限写入，需要借助第三方工具才能实现。因此跨平台的功能非常差。

#### 簇大小
簇是系统可以识别的最小单位。也就是对于文件，占用的簇数量都是整数，也就是不会有两个文件占用一个簇的情况发生。

每个簇可以包括 2、4、8、16、32 或 64 个扇区。显然，簇是操作系统所使用的逻辑概念，而非磁盘的物理特性。

为了更好地管理磁盘空间和更高效地从硬盘读取数据，操作系统规定一个簇中只能放置一个文件的内容，因此文件所占用的空间，只能是簇的整数倍；如果文件实际大小小于一簇，它也要占一簇的空间。如果文件实际大小大于一簇，根据逻辑推算，那么该文件就要占两个簇的空间。

## 出现问题
### 安装 Ubuntu 后无可引导设备
我的安装设备是弘基笔记本，具体型号忘记了（多年前使用的电脑，一直闲置），安装完 Ubuntu 之后出现无可引导设备（no bootable device found）错误

![ubuntui_install](https://cdn.jsdelivr.net/gh/Lewinz/lewinz.github.io@master/images/posts/ubuntu_install_2.jpg)

起初怀疑是制作启动 U 盘有问题（最开始是在 mac 上用命令行写到 U 盘上的），但换了好几次 U 盘制作工具之后，依旧无果，甚至将 CentOS 安装好一次之后，排除是启动 U 盘的问题

最后打开 bios 发现是安全引导（Secure boot）没有关闭

解决：  
1. 进入 bios 界面，在 Boot 标签页安全引导 (Secure Boot) 并关闭
2. 进入到 Security 标签，找到 “选择一个用于执行的可信任 UEFI 文件 (Select an UEFI file as trusted for executing) ” 并敲击回车。
3. 在这里你可以看到你的硬盘，例如 HDD0。如果你有多块硬盘，我希望你记住你安装 Ubuntu 的那块。同样敲击回车。
4. 选择 `<EFI>` -> `<ubuntu>` -> `<shimx64.efi>`
5. 保存后重启解决

## 参考链接
<https://www.jianshu.com/p/54d9a3a695cc?tdsourcetag=s_pctim_aiomsg>