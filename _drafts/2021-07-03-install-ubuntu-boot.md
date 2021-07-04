---
layout: post
title: 物理机安装 Ubuntu
categories: [install,ubuntu]
description: 物理机安装 Ubuntu
keywords: install,ubuntu
---
## 出现问题
### 安装 Ubuntu 后无可引导设备
我的安装设备是弘基笔记本，具体型号忘记了（多年前使用的电脑，一直闲置），安装完 Ubuntu 之后出现无可引导设备（no bootable device found）错误

![ubuntui_install](https://cdn.jsdelivr.net/gh/Lewinz/lewinz.github.io@master/images/posts/ubuntui_install_1.jpg)

起初怀疑是制作启动 U 盘有问题（最开始是在 mac 上用命令行写到 U 盘上的），但换了好几次 U 盘制作工具之后，依旧无果，甚至将 CentOS 安装好一次之后，排除是启动 U 盘的问题

最后打开 bios 发现是安全引导（Secure boot）没有关闭

解决：  
1. 进入 bios 界面，在 Boot 标签页安全引导 (Secure Boot) 并关闭
2. 进入到 Security 标签，找到 “选择一个用于执行的可信任 UEFI 文件 (Select an UEFI file as trusted for executing) ” 并敲击回车。
3. 在这里你可以看到你的硬盘，例如 HDD0。如果你有多块硬盘，我希望你记住你安装 Ubuntu 的那块。同样敲击回车。
4. 选择 `<EFI>` -> `<ubuntu>` -> `<shimx64.efi>`
5. 保存后重启解决

## 参考链接
https://www.jianshu.com/p/54d9a3a695cc?tdsourcetag=s_pctim_aiomsg