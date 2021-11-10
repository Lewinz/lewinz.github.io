---
layout: post
title: openstack 各服务重启命令
categories: [openstack, restart, cli]
description: openstack 各服务重启命令
keywords: openstack, restart, cli
---

### 重启 openstack 的整个服务
``` shell
openstack-service restart
```

### 重启 dashboard
``` shell
service httpd  restart 
service memcached restart
```
### 重启 ceilometer
#### cinder
``` shell
service mongod restart
```
#### controller
``` shell
service openstack-ceilometer-api restart  
service openstack-ceilometer-notification restart
service openstack-ceilometer-central restart
service openstack-ceilometer-collector restart
service openstack-ceilometer-alarm-evaluator restart
service openstack-ceilometer-alarm-notifier restart
```
#### compute
``` shell
service openstack-nova-compute restart
```
#### controller
``` shell
service openstack-glance-api restart
service openstack-glance-registry restart
Block Storage service
```
#### controller node
``` shell
service   openstack-cinder-api restart
service   openstack-cinder-scheduler restart
```
#### cinder
``` shell
service    openstack-cinder-volume  restart
```
### 重启 Fuel 服务
``` shell
docker restart fuel-core-6.1-nailgun
docker restart fuel-core-6.1-keystone
docker restart fuel-core-6.1-rsync
docker restart fuel-core-6.1-mcollective
docker restart fuel-core-6.1-ostf
docker restart fuel-core-6.1-astute
docker restart fuel-core-6.1-rsyslog
docker restart fuel-core-6.1-postgres
docker restart fuel-core-6.1-rabbitmq
docker restart fuel-core-6.1-nginx
docker restart fuel-core-6.1-cobbler
```
### Neutron 服务
#### 控制节点
``` shell
service openstack-nova-api restart
service openstack-nova-scheduler restart
service openstack-nova-conductor restart
service neutron-server restart
```
#### 网络节点
``` shell
service openvswitch restart
#（fuel 控制节点默认 stop）
service neutron-openvswitch-agent restart
#（fuel 控制节点默认 stop）
service neutron-l3-agent restart
#（fuel 控制节点默认 stop）
service neutron-dhcp-agent restart
#（fuel 控制节点默认 stop）
service neutron-metadata-agent restart
```
#### 计算节点
``` shell
service neutron-openvswitch-agent restart
service openvswitch restart
```
### 重启 cinder 服务
#### 控制节点
``` shell
service openstack-cinder-api restart
service openstack-cinder-scheduler restart
```
#### 存储节点
``` shell
service openstack-cinder-volume restart
```
### 重启 glance 服务
#### 控制节点
service openstack-glance-api restart
service openstack-glance-registry restart
### 重启 Swift 服务
#### 控制节点
``` shell
service openstack-swift-proxy restart
service memcached restart
```
#### 存储节点
``` shell
service openstack-swift-account restart
service openstack-swift-account-auditor restart
service openstack-swift-account-reaper restart
service openstack-swift-account-replicator restart
service openstack-swift-container restart
service openstack-swift-container-auditor restart
service openstack-swift-container-replicator restart
service openstack-swift-container-updater restart
service openstack-swift-object restart
service openstack-swift-object-auditor restart
service openstack-swift-object-replicator restart
service openstack-swift-object-updater restart
```
### 重启 Nova 服务
#### 控制节点
``` shell
service openstack-nova-api restart
service openstack-nova-cert restart
service openstack-nova-consoleauth restart
service openstack-nova-scheduler restart
service openstack-nova-conductor restart
service openstack-nova-novncproxy restart
```
#### 计算节点
``` shell
service libvirtd restart
service openstack-nova-compute restart
```