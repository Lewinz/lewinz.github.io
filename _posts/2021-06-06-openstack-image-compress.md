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
``` shell
vi /etc/cloud/cloud.cfg

# 修改内容：
disable_root: false
# 新增内容：
ssh_pwauth: true
```

修改 sshd_config
``` shell
vi /etc/ssh/sshd_config
# 修改的内容
PasswordAuthentication yes
PermitRootLogin yes
UseDNS no # 关闭 DNS 反查，提高 ssh 连接速度
GSSAPIAuthentication no # 关闭 gssapi 验证方式，提高 ssh 连接速度

vi /etc/hosts.allow
# 新增的内容
sshd:all # 防止 client 轮询导致的 IP 封禁
```
重启 sshd 服务： `service sshd restart` 
#### 设置系统盘自动扩容
Ubuntu 的官方 Cloud 镜像自带这个功能，所以不需要操作
#### 配置监控
``` shell
sudo apt-get update -y

sudo apt-get install -y qemu-guest-agent
```
#### 配置开启 nova console log
``` shell
vi /etc/default/grub

修改的内容：
GRUB_CMDLINE_LINUX_DEFAULT 加上 console=tty0 console=ttyS0,115200
```

#### 设置 DNS
##### Ubuntu 18.04
``` shell
vi /etc/systemd/resolved.conf

[Resolve]
DNS=114.114.114.114 114.114.115.115 223.5.5.5 223.6.6.6
#FallbackDNS=
#Domains=
LLMNR=no
#MulticastDNS=no
#DNSSEC=no
#Cache=yes
#DNSStubListener=yes
```

#### 设置系统时间
`timedatectl set-timezone Asia/Shanghai`

### CentOS
#### cloud-init
允许 root 远程登录
``` shell
yum install cloud-init

# 修改 /etc/cloud/cloud.cfg
vi /etc/cloud/cloud.cfg
disable_root: 0
ssh_pwauth:   1

# 文件末尾添加
user: root
```

#### sshd_config
``` shell
vi /etc/ssh/sshd_config

# 修改
PasswordAuthentication yes # 注意不能有多个
PermitRootLogin yes
UseDNS no # 关闭 DNS 反查，提高 ssh 连接速度
GSSAPIAuthentication no # 关闭 gssapi 验证方式，提高 ssh 连接速度

vi /etc/hosts.allow

# 新增的内容：
sshd:all # 防止 client 轮询导致的 IP 封禁

# 重启 sshd 服务
service sshd restart
```

#### 配置支持系统盘自动扩容
CentOS7 官方镜像自带自动扩容，无需修改  
CentOS6 参考[文档](https://ykfq.github.io/openstack/create-centos6-image-for-openstack) 修改

#### qemu-guest-agent(监控)
虚机内操作
``` shell
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

#### 设置系统时间
`timedatectl set-timezone Asia/Shanghai`

#### 开启 nova console log 日志输出支持
##### centos6
``` shell
vim /etc/grub.conf
console=tty0 console=ttyS0,115200n8 #追加到 kernel 行末尾

#注：/etc/grub.conf /boot/grub/menu.lst 都是指向 /boot/grub/grub.conf 的软链。
```

##### centos7
``` shell
vim /etc/default/grub
删除 rhgb quiet 并追加 console=tty0 console=ttyS0,115200n8
``` 

#### 重新生成 grub.cfg 文件
`grub2-mkconfig -o /boot/grub2/grub.cfg`

### 镜像文件格式转换与压缩
#### openstack 命令行客户端鉴权
``` shell
source auth.sh

# auth.sh 为鉴权文件，例如
export OS_USERNAME=openstack_username
export OS_PASSWORD=openstack_password
export OS_TENANT_NAME=name
export OS_AUTH_URL=https://xxxxxx:5000/v3
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_DOMAIN_NAME=Default
export OS_IDENTITY_API_VERSION=3
export OS_PROJECT_ID=project_id
```
#### 镜像格式转换
``` shell
# -f 转换前格式
# -O 转换后格式
qemu-img convert -f qcow2 -O raw test.qcow2 test.raw
```
#### 压缩镜像文件
``` shell
# -- -18.5G 在原有镜像文件基础上压缩 18.5G
# CentOS 镜像可能会出现压缩后转换格式 size 变很小，解决办法是压缩时减少压缩大小
qemu-img resize -f raw --shrink centos.raw -- -5G
```
#### 重新上传镜像文件
``` shell
glance image-create --name Ubuntu --disk-format qcow2 --container-format bare --property os_type="linux" --property os_distro="UbuntuServer16.04-64" --file test.qcow2
```

### 参考
[参考](https://github.com/guojy8993/blogs/blob/master/OpenStack%E9%95%9C%E5%83%8F(%E5%9F%BA%E4%BA%8ECentOS7)%E7%9A%84%E5%88%B6%E4%BD%9C%E4%B8%8E%E8%AF%B4%E6%98%8E)