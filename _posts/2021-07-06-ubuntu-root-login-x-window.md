---
layout: post
title: Ubuntu 20.04 设置 root 登录图形界面
categories: [ubuntu,root,login]
description: Ubuntu 20.04 设置 root 登录图形界面
keywords: ubuntu,root,login
---

Ubuntu 默认是关闭 root 账户登录图形界面的

### 修改 root 账号密码
`sudo passwd root`

### 修改配置文件
#### 修改 50-ubuntu.conf
执行 `sudo vim /usr/share/lightdm/lightdm.conf.d/50-ubuntu.conf` 把配置改为如下所示
``` sh
[Seat:*]
user-session=ubuntu
greeter-show-manual-login= true
```
#### 修改 gdm-autologin 和 gdm-password
执行 `sudo vim /etc/pam.d/gdm-autologin` 注释掉 `auth required pam_succeed_if.so user != root quiet_success` 这一行 (第三行左右)
``` sh
#%PAM-1.0
auth    requisite       pam_nologin.so
#auth   required        pam_succeed_if.so user != root quiet_success
auth    optional        pam_gdm.so
auth    optional        pam_gnome_keyring.so
auth    required        pam_permit.so
```
执行 `sudo vim /etc/pam.d/gdm-password` 注释掉 `auth required pam_succeed_if.so user != root quiet_success` 这一行 (第三行左右)
``` sh
#%PAM-1.0
auth    requisite       pam_nologin.so
#auth   required        pam_succeed_if.so user != root quiet_success
@include common-auth
auth    optional        pam_gnome_keyring.so
@include common-account
```
#### 修改 /root/.profile 文件
执行 `sudo vim/root/.profile` 修改配置文件如下
``` sh
# ~/.profile: executed by Bourne-compatible login shells.

if [ "$BASH" ]; then
  if [ -f ~/.bashrc ]; then
    . ~/.bashrc
  fi
fi
tty -s && mesg n || true
mesg n || true
```

### 重启生效，使用 root 登录