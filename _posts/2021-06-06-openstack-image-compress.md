---
layout: post
title: openstack 制作标准镜像
categories: [openstack,镜像 ]
description: openstack 制作标准镜像
keywords: openstack,镜像
---

## 下载官方镜像
具体的镜像地址参考： [https://docs.openstack.org/image-guide/obtain-images.html](https://docs.openstack.org/image-guide/obtain-images.html)

## 需要对镜像做的改动：
- 配置支持密码登录
- 配置支持系统盘自动扩容
- 配置支持使用 root 登录
- 配置支持监控 qemu-guest-agent
- 配置系统日志，开启 nova console log

## Ubuntu | CentOS

### Ubuntu
#### 设置密码和 root 登录
修改 cloud-init config
```sh
vi /etc/cloud/cloud.cfg

# 修改内容：
disable_root: false
# 新增内容：
ssh_pwauth: true
```

修改 sshd_config
```sh
vi /etc/ssh/sshd_config
# 修改的内容：
PasswordAuthentication yes
PermitRootLogin yes
```
重启 sshd 服务： `service sshd restart` 
#### 设置系统盘自动扩容
Ubuntu 的官方 Cloud 镜像自带这个功能，所以不需要操作
#### 配置监控
```sh
sudo apt-get update -y

sudo apt-get install -y qemu-guest-agent
```
#### 配置开启 nova console log
```sh
vi /etc/default/grub

修改的内容：
GRUB_CMDLINE_LINUX_DEFAULT 加上 console=tty0 console=ttyS0,115200
```

### CentOS
#### cloud-init
允许 root 远程登录
``` sh
yum install cloud-init

# 修改 /etc/cloud/cloud.cfg
vi /etc/cloud/cloud.cfg
disable_root: 0
ssh_pwauth:   1

# 文件末尾添加
user: root
```

#### sshd_config
``` sh
vi /etc/ssh/sshd_config

# 修改
PasswordAuthentication yes # 注意不能有多个
PermitRootLogin yes

# 重启 sshd 服务
service sshd restart
```

### 配置支持系统盘自动扩容
CentOS7 官方镜像自带自动扩容，无需修改  
CentOS6 参考[文档](https://ykfq.github.io/openstack/create-centos6-image-for-openstack) 修改

### qemu-guest-agent(监控)
虚机内操作
```sh
yum install qemu-guest-agent

vi /etc/sysconfig/qemu-ga
# 新增
FSFREEZE_HOOK_ENABLE=1
# 注释
BLACKLIST_RPC="guest-file-open,guest-file-close,guest-file-read,guest-file-write,guest-file-seek,guest-file-flush"
==>
# BLACKLIST_RPC="guest-file-open,guest-file-close,guest-file-read,guest-file-write,guest-file-seek,guest-file-flush"
```
镜像文件操作  
修改镜像元数据，新增
`hw_qemu_guest_agent=yes`

### 开启 nova console log 日志输出支持
#### centos6
``` sh
vim /etc/grub.conf
console=tty0 console=ttyS0,115200n8 #追加到 kernel 行末尾

#注：/etc/grub.conf /boot/grub/menu.lst 都是指向 /boot/grub/grub.conf 的软链。
```

#### centos7
``` sh
vim /etc/default/grub
删除 rhgb quiet 并追加 console=tty0 console=ttyS0,115200n8
``` 

#### 重新生成 grub.cfg 文件
`grub2-mkconfig -o /boot/grub2/grub.cfg`

### 镜像文件格式转换与压缩
#### openstack 命令行客户端鉴权
``` sh
source auth.sh

# auth.sh 为鉴权文件，例如
export OS_USERNAME=openstack_username
export OS_PASSWORD=openstack_password
export OS_TENANT_NAME=name
export OS_AUTH_URL=https://qvm-wz.qiniu.com:5000/v3
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_DOMAIN_NAME=Default
export OS_IDENTITY_API_VERSION=3
export OS_PROJECT_ID=project_id
```
#### 镜像格式转换
``` sh
# -f 转换前格式
# -O 转换后格式
qemu-img convert -f qcow2 -O raw test.qcow2 test.raw
```
#### 压缩镜像文件
``` sh
# -- -18.5G 在原有镜像文件基础上压缩 18.5G
# CentOS 镜像可能会出现压缩后转换格式 size 变很小，解决办法是压缩时减少压缩大小
qemu-img resize test.raw -- -18.5G
```
#### 重新上传镜像文件
``` sh
glance image-create --name Ubuntu --disk-format raw --container-format bare --property os_type="linux" --property os_distro="UbuntuServer16.04-64" --file test.qcow2
```
