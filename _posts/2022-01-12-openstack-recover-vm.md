---
layout: post
title: OpenStack 实例宕机恢复
categories: [OpenStack, recover, instance]
description: OpenStack 实例宕机恢复
keywords: OpenStack, recover, instance
---

## 环境信息
- OS --> CentOS7.2-1511
- OpenStack --> Mikata
- Ceph -->j 版
虽然 OpenStack 自带有迁移和疏散机制，但并不一定保证 100% 成功，本文基于疏散失败的情况，来恢复实例。  
起因客户那边，物理机系统盘故障，导致数据全部丢失，最开始想到的方法是疏散，直接在 dashboard 或者控制节点终端执行疏散命令

`nova evacuate  node09`

然而事情并没有到此结束，有 2 台虚拟机，一直疏散不了，没办法，数据最重要，由于 nova，glance，cinder 使用的都是 ceph，即数据都存放在 ceph 中，这样恢复起来就有了可能，  
具体步骤如下：  

## 重建 libvirt.xml 文件
首先查看一下拟机信息
`nova show 6bb3bc65-91c3-486c-b853-2a5c879a5395`

具体信息如下：
``` shell
+--------------------------------------+----------------------------------------------------------+
| Property                             | Value                                                    |
+--------------------------------------+----------------------------------------------------------+
|  network                             | 192.168.10.25, xx.xx.xx.xx                              |
| OS-DCF:diskConfig                    | AUTO                                                     |
| OS-EXT-AZ:availability_zone          | nova                                                     |
| OS-EXT-SRV-ATTR:host                 | node03                                                   |
| OS-EXT-SRV-ATTR:hypervisor_hostname  | node03                                                   |
| OS-EXT-SRV-ATTR:instance_name        | instance-000001ae                                        |
| OS-EXT-STS:power_state               | 1                                                        |
| OS-EXT-STS:task_state                | -                                                        |
| OS-EXT-STS:vm_state                  | active                                                   |
| OS-SRV-USG:launched_at               | 2017-06-23T06:03:20.000000                               |
| OS-SRV-USG:terminated_at             | -                                                        |
| accessIPv4                           |                                                          |
| accessIPv6                           |                                                          |
| config_drive                         |                                                          |
| created                              | 2017-06-23T06:01:44Z                                     |
| flavor                               | yunwei01 (d76ea4cd-0c46-423c-9a2a-0ab15c5a1b0a)          |
| hostId                               | 7081812a3c654f417fc545300ecd03252d8ac4bf992b54272bcfee61 |
| id                                   | 6bb3bc65-91c3-486c-b853-2a5c879a5395                     |
| image                                | centos65 (0488b591-1755-4da0-abe9-8e5c2a6931b5)          |
| key_name                             | -                                                        |
| metadata                             | {}                                                       |
| name                                 | rhea-2                                                   |
| os-extended-volumes:volumes_attached | [{"id": "7cd2936a-b4ef-4e85-8159-eace4c1b7981"}]         |
| progress                             | 0                                                        |
| security_groups                      | default                                                  |
| status                               | ACTIVE                                                   |
| tenant_id                            | 290f841df5644709847a51d6604a228f                         |
| updated                              | 2017-12-21T06:46:13Z                                     |
| user_id                              | d158732076c740aead5286969273faea                         |
+-------------------------------------+----------------------------------------------------------+
```
记录这些信息，后面有大用

在平台上，找一个 flavor 和 image 相同的实例，找到此实例 uuid

查看该实例信息
`nova show uuid`

进入到相应节点，拷贝 libvirt.xml 和 console.log
``` shell
cd /var/lib/nova/instance/$uuid/
cp * /mnt/bakrecovery
cd /mnt/bakrecovery
more libvirt.xml
```

信息如下：
``` xml
<domain type="kvm">
  <uuid>6bb3bc65-91c3-486c-b853-2a5c879a5395</uuid>
  <name>instance-000001ae</name>
  <memory>16777216</memory>
  <vcpu>16</vcpu>
  <metadata>
    <nova:instance xmlns:nova="http://openstack.org/xmlns/libvirt/nova/1.0">
      <nova:package version="13.1.0-1.el7"/>
      <nova:name>rhea-2</nova:name>
      <nova:creationTime>2017-12-21 06:46:11</nova:creationTime>
      <nova:flavor name="yunwei01">
        <nova:memory>16384</nova:memory>
        <nova:disk>40</nova:disk>
        <nova:swap>2048</nova:swap>
        <nova:ephemeral>0</nova:ephemeral>
        <nova:vcpus>16</nova:vcpus>
      </nova:flavor>
      <nova:owner>
        <nova:user uuid="3eb5de8526c14da495488f1a264915f6">admin</nova:user>
        <nova:project uuid="10d0b378c82d4b5da2288ef852ca5bd1">admin</nova:project>
      </nova:owner>
      <nova:root type="image" uuid="0488b591-1755-4da0-abe9-8e5c2a6931b5"/>
    </nova:instance>
  </metadata>
  <sysinfo type="smbios">
    <system>
      <entry name="manufacturer">Fedora Project</entry>
      <entry name="product">OpenStack Nova</entry>
      <entry name="version">13.1.0-1.el7</entry>
      <entry name="serial">d3c7a080-2534-4b5f-9d86-9cd152acc671</entry>
      <entry name="uuid">6bb3bc65-91c3-486c-b853-2a5c879a5395</entry>
      <entry name="family">Virtual Machine</entry>
    </system>
  </sysinfo>
  <os>
    <type>hvm</type>
    <boot dev="hd"/>
    <smbios mode="sysinfo"/>
  </os>
  <features>
    <acpi/>
    <apic/>
  </features>
  <cputune>
    <shares>16384</shares>
  </cputune>
  <clock offset="utc">
    <timer name="pit" tickpolicy="delay"/>
    <timer name="rtc" tickpolicy="catchup"/>
    <timer name="hpet" present="no"/>
  </clock>
  <cpu mode="host-model" match="exact">
    <topology sockets="16" cores="1" threads="1"/>
  </cpu>
  <devices>
    <disk type="network" device="disk">
      <driver type="raw" cache="writeback"/>
      <source protocol="rbd" name="vms/6bb3bc65-91c3-486c-b853-2a5c879a5395_disk">
        <host name="172.16.2.5" port="6789"/>
        <host name="172.16.2.6" port="6789"/>
        <host name="172.16.2.7" port="6789"/>
      </source>
      <auth username="cinder">
        <secret type="ceph" uuid="14fdf1bb-44d7-40ad-a98a-16fe7e65115b"/>
      </auth>
      <target bus="virtio" dev="vda"/>
    </disk>
    <disk type="network" device="disk">
      <driver type="raw" cache="writeback"/>
      <source protocol="rbd" name="vms/6bb3bc65-91c3-486c-b853-2a5c879a5395_disk.swap">
        <host name="172.16.2.5" port="6789"/>
        <host name="172.16.2.6" port="6789"/>
        <host name="172.16.2.7" port="6789"/>
      </source>
      <auth username="cinder">
        <secret type="ceph" uuid="14fdf1bb-44d7-40ad-a98a-16fe7e65115b"/>
      </auth>
      <target bus="virtio" dev="vdb"/>
    </disk>
    <disk type="network" device="disk">
      <driver name="qemu" type="raw" cache="writeback"/>
      <source protocol="rbd" name="volumes/volume-7cd2936a-b4ef-4e85-8159-eace4c1b7981">
        <host name="172.16.2.5" port="6789"/>
        <host name="172.16.2.6" port="6789"/>
        <host name="172.16.2.7" port="6789"/>
      </source>
      <auth username="cinder">
        <secret type="ceph" uuid="14fdf1bb-44d7-40ad-a98a-16fe7e65115b"/>
      </auth>
      <target bus="virtio" dev="vdc"/>
      <serial>7cd2936a-b4ef-4e85-8159-eace4c1b7981</serial>
    </disk>
    <interface type="bridge">
      <mac address="fa:16:3e:34:e3:fe"/>
      <model type="virtio"/>
      <source bridge="qbrff441fbc-bb"/>
      <target dev="tapff441fbc-bb"/>
    </interface>
    <serial type="file">
      <source path="/var/lib/nova/instances/6bb3bc65-91c3-486c-b853-2a5c879a5395/console.log"/>
    </serial>
    <serial type="pty"/>
    <input type="tablet" bus="usb"/>
    <graphics type="vnc" autoport="yes" keymap="en-us" listen="0.0.0.0"/>
    <video>
      <model type="cirrus"/>
    </video>
    <memballoon model="virtio">
      <stats period="10"/>
    </memballoon>
  </devices>
</domain>
```
需要修改如下部分：

1. name 和 uuid  
``` xml
<uuid>6bb3bc65-91c3-486c-b853-2a5c879a5395</uuid>
<name>instance-000001ae</name>
```
前面记录信息，刚好派上用场

2. name 和创建时间  
``` xml
<nova:name>rhea-2</nova:name>
<nova:creationTime>2017-12-21 06:46:11</nova:creationTime>
```
这个 name 是创建实例的时候，我们起的名字，上面的那个，是随机生成的名字

3. 修改相应 uuid（包括 image，project，user）
``` xml
<nova:user uuid="3eb5de8526c14da495488f1a264915f6">admin</nova:user>
<nova:project uuid="10d0b378c82d4b5da2288ef852ca5bd1">admin</nova:project>
</nova:owner>
<nova:root type="image" uuid="0488b591-1755-4da0-abe9-8e5c2a6931b5"/>
```

3. 修改虚拟机 uuid，和 serial，serial 通过比对发现，同一个计算节点这个号相同
``` xml
<entry name="serial">d3c7a080-2534-4b5f-9d86-9cd152acc671</entry>
<entry name="uuid">6bb3bc65-91c3-486c-b853-2a5c879a5395</entry>
<entry name="family">Virtual Machine</entry>
```

4. 块设备 id，格式为虚拟机 uuid_disk,uuid_swap
``` xml
<source protocol="rbd" name="vms/6bb3bc65-91c3-486c-b853-2a5c879a5395_disk">
<source protocol="rbd" name="vms/6bb3bc65-91c3-486c-b853-2a5c879a5395_disk.swap">
```

5. 通过 cinder 划分的卷 id
``` xml
<target bus="virtio" dev="vdc"/>
<serial>7cd2936a-b4ef-4e85-8159-eace4c1b7981</serial>
</disk>
```

6. 虚拟机网卡 mac 地址，还有端口，这里，mac 地址可通过 dashboard 查看，admin 账户下，管理员–> 网络–>（网络）uuid–> 端口
``` xml
<interface type="bridge">
<mac address="fa:16:3e:34:e3:fe"/>
<model type="virtio"/>
<source bridge="qbrff441fbc-bb"/>
<target dev="tapff441fbc-bb"/>
```

7. console 日志路径
``` xml
<source path="/var/lib/nova/instances/6bb3bc65-91c3-486c-b853-2a5c879a5395/console.log"/>
```
8. 对比修改完后，一定要仔细确认，以免出现问题
``` shell
mkdir -p /var/lib/nova/instance/$uuid
cp libvirt.xml  /var/lib/nova/instance/$uuid
cp console.log /var/lib/nova/instance/$uuid
```

## 修改权限
``` shell
chown nova.nova /var/lib/nova/instance/$uuid/xml
chown qemu.qemu /var/lib/nova/instance/$uuid/console.log
```

## 在数据库中修改节点信息
``` shell
update instances set host='node10',node='node10' where uuid='6bb3bc65-91c3-486c-b853-2a5c879a5395';
```

## 重启相应节点 nova-coompute 服务
``` shell
systemctl restart openstack-nova-compute
```

## 启动实例，启动后网络可能不通，这时可以通过迁移来解决，

[博客链接](https://www.its404.com/article/H_haow/80287192)