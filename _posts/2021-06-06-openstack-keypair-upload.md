---
layout: post
title: openstack 秘钥上传
categories: [openstack,秘钥]
description: openstack 秘钥
keywords: openstack,秘钥
---

``` sh
nova help | grep key
    flavor-key          Set or unset extra_spec for a flavor.
    keypair-add         Create a new key pair for use with instances.
    keypair-delete      Delete keypair given by its name.
    keypair-list        Print a list of keypairs for a user
    keypair-show        Show details about the given keypair.
```

#### 上传秘钥
``` sh
# /root/.ssh/id_rsa.pub 文件位置
# terrykey 秘钥名
nova keypair-add --pub-key /root/.ssh/id_rsa.pub terrykey
```

#### 查看秘钥列表
``` sh
nova keypair-list

+----------+-------------------------------------------------+
| Name     | Fingerprint                                     |
+----------+-------------------------------------------------+
| terrykey | 94:b8:9c:2a:31:8c:2c:87:7f:f5:80:24:23:73:f8:e9 |
+----------+-------------------------------------------------+
```

