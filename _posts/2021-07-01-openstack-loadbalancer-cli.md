---
layout: post
title: openstack loadbalancer 命令行
categories: [openstack,命令行,loadbalancer]
description: openstack loadbalancer 命令行
keywords: openstack,命令行,loadbalancer
---

## 鉴权文件格式
``` sh
export OS_USERNAME=admin
export OS_PASSWORD=adminPassword
export OS_TENANT_NAME=admin
export OS_AUTH_URL=https://xxxxxxxx:port/v3
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_DOMAIN_NAME=Default
export OS_IDENTITY_API_VERSION=3
export OS_PROJECT_ID=xxxxxxxxxxxxxxxxx
```

## CLI
### 查询环境中有的子网和安全组
``` sh
[lookback@LookdeMacBook-Pro ~/OpenStack]$ openstack network list
+--------------------------------------+------------+--------------------------------------+
| ID                                   | Name       | Subnets                              |
+--------------------------------------+------------+--------------------------------------+
| 0d3f4ed0-6eaf-4526-989d-6666898e3721 | lbaas-mgmt | 8f2ead4f-ff07-422f-815e-ff4dd8ede705 |
| 24333c1d-001b-4898-9c30-994a20b57cb1 | NET-A      | ec936eaa-ca78-45fb-9b02-561681a966dd |
| 58ba4366-ecc5-46f6-8898-8b8e743797d6 | NET-B      | e0531da2-b031-4e0a-9303-33f10d9c3aec |
| 67ca0cc2-68f9-4aee-b059-4666e2721dfa | NET-C      | 3fb199b8-8375-41b5-aebe-e6684bb0fe57 |
+--------------------------------------+------------+--------------------------------------+
[lookback@LookdeMacBook-Pro ~/OpenStack]$ openstack security group list 
+--------------------------------------+-----------------------------------------+------------------------------------+----------------------------------+------+
| ID                                   | Name                                    | Description                        | Project                          | Tags |
+--------------------------------------+-----------------------------------------+------------------------------------+----------------------------------+------+
| 1327a82f-6974-45d0-9361-6029b2b7f1fe | lb-26a6049b-f5d0-4ac4-b64f-b3e6671ae6e0 |                                    | fe1eada6191c45bca8cf2eb1b1a1eb47 | []   |
| 52f0eed5-f398-427c-a4f6-2c6d7c8b88e4 | default                                 | Default security group             | 8561e762e5ce4b79ab5757f9e393a44b | []   |
| a657ba7b-f2ff-4e10-b986-fe84243a0a16 | RAX-TEST                                | RAX-TEST                           | 5df4e0c1c68647ef800e7904d455b58d | []   |
| ac3f43c5-fe9d-49bb-befd-c42b2f100150 | octavia_sec_grp                         | security group for octavia amphora | fe1eada6191c45bca8cf2eb1b1a1eb47 | []   |
| d9386462-0eae-43c1-b815-f999fa3cd833 | 放行进出公网                            | 对30段网络的公网进出全放行         | 5df4e0c1c68647ef800e7904d455b58d | []   |
| ee287eef-0ad2-4d78-97dd-1fef7544879e | default                                 | Default security group             | fe1eada6191c45bca8cf2eb1b1a1eb47 | []   |
| f0e116a5-35a4-47c6-baeb-06aa15cfdfed | rax-test-20200310                       | rax-test-20200310                  | 5df4e0c1c68647ef800e7904d455b58d | []   |
| f0e28f3a-167a-4134-93d7-0ef001c72f6d | default                                 | Default security group             | 5df4e0c1c68647ef800e7904d455b58d | []   |
+--------------------------------------+-----------------------------------------+------------------------------------+----------------------------------+------+
[lookback@LookdeMacBook-Pro ~/OpenStack]$
```

### 创建一个在子网 NET-B 上负载均衡 IP 172.25.255.202 名字为 test2
``` sh
[lookback@LookdeMacBook-Pro ~/OpenStack]$ openstack loadbalancer create --name test2 --vip-subnet-id e0531da2-b031-4e0a-9303-33f10d9c3aec --vip-address 172.25.255.202
+---------------------+--------------------------------------+
| Field               | Value                                |
+---------------------+--------------------------------------+
| admin_state_up      | True                                 |
| availability_zone   |                                      |
| created_at          | 2020-03-22T16:46:45                  |
| description         |                                      |
| flavor_id           | None                                 |
| id                  | 26a6049b-f5d0-4ac4-b64f-b3e6671ae6e0 |
| listeners           |                                      |
| name                | test2                                |
| operating_status    | OFFLINE                              |
| pools               |                                      |
| project_id          | 5df4e0c1c68647ef800e7904d455b58d     |
| provider            | amphora                              |
| provisioning_status | PENDING_CREATE                       |
| updated_at          | None                                 |
| vip_address         | 172.25.255.202                       |
| vip_network_id      | 58ba4366-ecc5-46f6-8898-8b8e743797d6 |
| vip_port_id         | abe786ea-7c9a-4937-816d-edd996dde5a5 |
| vip_qos_policy_id   | None                                 |
| vip_subnet_id       | e0531da2-b031-4e0a-9303-33f10d9c3aec |
+---------------------+--------------------------------------+
[lookback@LookdeMacBook-Pro ~/OpenStack]$
```
### 给 vip port 的安全组添加 icmpk 可以 ping 的规则
``` sh
openstack loadbalancer list
loadbalancer_id=26a6049b-f5d0-4ac4-b64f-b3e6671ae6e0
[lookback@LookdeMacBook-Pro ~/OpenStack]$ openstack security group rule create --protocol icmp --remote-ip 0.0.0.0/0 $(openstack port show $(openstack loadbalancer show $loadbalancer_id | awk '$2=="vip_port_id"{print $4}') | awk '$2=="security_group_ids"{print $4}')
+-------------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Field             | Value                                                                                                                                                            |
+-------------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| created_at        | 2020-03-23T08:45:58Z                                                                                                                                             |
| description       |                                                                                                                                                                  |
| direction         | ingress                                                                                                                                                          |
| ether_type        | IPv4                                                                                                                                                             |
| id                | 3bdb7e7c-dc72-46e9-b9d2-0481bbad42a4                                                                                                                             |
| location          | cloud='', project.domain_id='default', project.domain_name=, project.id='5df4e0c1c68647ef800e7904d455b58d', project.name='admin', region_name='RegionOne', zone= |
| name              | None                                                                                                                                                             |
| port_range_max    | None                                                                                                                                                             |
| port_range_min    | None                                                                                                                                                             |
| project_id        | 5df4e0c1c68647ef800e7904d455b58d                                                                                                                                 |
| protocol          | icmp                                                                                                                                                             |
| remote_group_id   | None                                                                                                                                                             |
| remote_ip_prefix  | 0.0.0.0/0                                                                                                                                                        |
| revision_number   | 0                                                                                                                                                                |
| security_group_id | d9386462-0eae-43c1-b815-f999fa3cd833                                                                                                                             |
| tags              | []                                                                                                                                                               |
| updated_at        | 2020-03-23T08:45:58Z                                                                                                                                             |
+-------------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------+
[lookback@LookdeMacBook-Pro ~/OpenStack]$
```
### 给 vip 添加安全组
``` sh
[lookback@LookdeMacBook-Pro ~/OpenStack]$ openstack port set --security-group d9386462-0eae-43c1-b815-f999fa3cd833 abe786ea-7c9a-4937-816d-edd996dde5a5
[lookback@LookdeMacBook-Pro ~/OpenStack]$ 
[lookback@LookdeMacBook-Pro ~/OpenStack]$ 
[lookback@LookdeMacBook-Pro ~/OpenStack]$ openstack port show abe786ea-7c9a-4937-816d-edd996dde5a5
+-------------------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Field                   | Value                                                                                                                                                            |
+-------------------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| admin_state_up          | DOWN                                                                                                                                                             |
| allowed_address_pairs   |                                                                                                                                                                  |
| binding_host_id         |                                                                                                                                                                  |
| binding_profile         |                                                                                                                                                                  |
| binding_vif_details     |                                                                                                                                                                  |
| binding_vif_type        | unbound                                                                                                                                                          |
| binding_vnic_type       | normal                                                                                                                                                           |
| created_at              | 2020-03-22T16:46:45Z                                                                                                                                             |
| data_plane_status       | None                                                                                                                                                             |
| description             |                                                                                                                                                                  |
| device_id               | lb-26a6049b-f5d0-4ac4-b64f-b3e6671ae6e0                                                                                                                          |
| device_owner            | Octavia                                                                                                                                                          |
| dns_assignment          | fqdn='host-172-25-255-202.openstack.local.', hostname='host-172-25-255-202', ip_address='172.25.255.202'                                                         |
| dns_domain              |                                                                                                                                                                  |
| dns_name                |                                                                                                                                                                  |
| extra_dhcp_opts         |                                                                                                                                                                  |
| fixed_ips               | ip_address='172.25.255.202', subnet_id='e0531da2-b031-4e0a-9303-33f10d9c3aec'                                                                                    |
| id                      | abe786ea-7c9a-4937-816d-edd996dde5a5                                                                                                                             |
| location                | cloud='', project.domain_id='default', project.domain_name=, project.id='5df4e0c1c68647ef800e7904d455b58d', project.name='admin', region_name='RegionOne', zone= |
| mac_address             | fa:16:3e:e2:f5:62                                                                                                                                                |
| name                    | octavia-lb-26a6049b-f5d0-4ac4-b64f-b3e6671ae6e0                                                                                                                  |
| network_id              | 58ba4366-ecc5-46f6-8898-8b8e743797d6                                                                                                                             |
| port_security_enabled   | True                                                                                                                                                             |
| project_id              | 5df4e0c1c68647ef800e7904d455b58d                                                                                                                                 |
| propagate_uplink_status | None                                                                                                                                                             |
| qos_policy_id           | None                                                                                                                                                             |
| resource_request        | None                                                                                                                                                             |
| revision_number         | 3                                                                                                                                                                |
| security_group_ids      | 1327a82f-6974-45d0-9361-6029b2b7f1fe, d9386462-0eae-43c1-b815-f999fa3cd833                                                                                       |
| status                  | DOWN                                                                                                                                                             |
| tags                    |                                                                                                                                                                  |
| trunk_details           | None                                                                                                                                                             |
| updated_at              | 2020-03-22T17:10:56Z                                                                                                                                             |
+-------------------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------+
[lookback@LookdeMacBook-Pro ~/OpenStack]$ 
[lookback@LookdeMacBook-Pro ~/OpenStack]$ openstack port unset --security-group 1327a82f-6974-45d0-9361-6029b2b7f1fe abe786ea-7c9a-4937-816d-edd996dde5a5
[lookback@LookdeMacBook-Pro ~/OpenStack]$ openstack port show abe786ea-7c9a-4937-816d-edd996dde5a5                                                       
+-------------------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Field                   | Value                                                                                                                                                            |
+-------------------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| admin_state_up          | DOWN                                                                                                                                                             |
| allowed_address_pairs   |                                                                                                                                                                  |
| binding_host_id         |                                                                                                                                                                  |
| binding_profile         |                                                                                                                                                                  |
| binding_vif_details     |                                                                                                                                                                  |
| binding_vif_type        | unbound                                                                                                                                                          |
| binding_vnic_type       | normal                                                                                                                                                           |
| created_at              | 2020-03-22T16:46:45Z                                                                                                                                             |
| data_plane_status       | None                                                                                                                                                             |
| description             |                                                                                                                                                                  |
| device_id               | lb-26a6049b-f5d0-4ac4-b64f-b3e6671ae6e0                                                                                                                          |
| device_owner            | Octavia                                                                                                                                                          |
| dns_assignment          | fqdn='host-172-25-255-202.openstack.local.', hostname='host-172-25-255-202', ip_address='172.25.255.202'                                                         |
| dns_domain              |                                                                                                                                                                  |
| dns_name                |                                                                                                                                                                  |
| extra_dhcp_opts         |                                                                                                                                                                  |
| fixed_ips               | ip_address='172.25.255.202', subnet_id='e0531da2-b031-4e0a-9303-33f10d9c3aec'                                                                                    |
| id                      | abe786ea-7c9a-4937-816d-edd996dde5a5                                                                                                                             |
| location                | cloud='', project.domain_id='default', project.domain_name=, project.id='5df4e0c1c68647ef800e7904d455b58d', project.name='admin', region_name='RegionOne', zone= |
| mac_address             | fa:16:3e:e2:f5:62                                                                                                                                                |
| name                    | octavia-lb-26a6049b-f5d0-4ac4-b64f-b3e6671ae6e0                                                                                                                  |
| network_id              | 58ba4366-ecc5-46f6-8898-8b8e743797d6                                                                                                                             |
| port_security_enabled   | True                                                                                                                                                             |
| project_id              | 5df4e0c1c68647ef800e7904d455b58d                                                                                                                                 |
| propagate_uplink_status | None                                                                                                                                                             |
| qos_policy_id           | None                                                                                                                                                             |
| resource_request        | None                                                                                                                                                             |
| revision_number         | 4                                                                                                                                                                |
| security_group_ids      | d9386462-0eae-43c1-b815-f999fa3cd833                                                                                                                             |
| status                  | DOWN                                                                                                                                                             |
| tags                    |                                                                                                                                                                  |
| trunk_details           | None                                                                                                                                                             |
| updated_at              | 2020-03-22T17:16:41Z                                                                                                                                             |
+-------------------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------+
[lookback@LookdeMacBook-Pro ~/OpenStack]$
```
### 创建一个 listener 监控器
``` sh
[lookback@LookdeMacBook-Pro ~/OpenStack]$ openstack loadbalancer listener create --name listener-test2 --protocol TCP --protocol-port 4200 test2
+-----------------------------+--------------------------------------+
| Field                       | Value                                |
+-----------------------------+--------------------------------------+
| admin_state_up              | True                                 |
| connection_limit            | -1                                   |
| created_at                  | 2020-03-22T16:50:42                  |
| default_pool_id             | None                                 |
| default_tls_container_ref   | None                                 |
| description                 |                                      |
| id                          | 81d6ca4f-c6b0-4674-9eef-80511d0f97be |
| insert_headers              | None                                 |
| l7policies                  |                                      |
| loadbalancers               | 26a6049b-f5d0-4ac4-b64f-b3e6671ae6e0 |
| name                        | listener-test2                       |
| operating_status            | OFFLINE                              |
| project_id                  | 5df4e0c1c68647ef800e7904d455b58d     |
| protocol                    | TCP                                  |
| protocol_port               | 4200                                 |
| provisioning_status         | PENDING_CREATE                       |
| sni_container_refs          | []                                   |
| timeout_client_data         | 50000                                |
| timeout_member_connect      | 5000                                 |
| timeout_member_data         | 50000                                |
| timeout_tcp_inspect         | 0                                    |
| updated_at                  | None                                 |
| client_ca_tls_container_ref | None                                 |
| client_authentication       | NONE                                 |
| client_crl_container_ref    | None                                 |
| allowed_cidrs               |                                      |
+-----------------------------+--------------------------------------+
[lookback@LookdeMacBook-Pro ~/OpenStack]$
```
### 创建一个资源池
``` sh
[lookback@LookdeMacBook-Pro ~/OpenStack]$ openstack loadbalancer pool create --name pool-test2 --lb-algorithm SOURCE_IP --listener listener-test2 --protocol TCP
+----------------------+--------------------------------------+
| Field                | Value                                |
+----------------------+--------------------------------------+
| admin_state_up       | True                                 |
| created_at           | 2020-03-22T16:54:22                  |
| description          |                                      |
| healthmonitor_id     |                                      |
| id                   | 6850d049-2d41-4e47-bf4b-f930fb09784a |
| lb_algorithm         | SOURCE_IP                            |
| listeners            | 81d6ca4f-c6b0-4674-9eef-80511d0f97be |
| loadbalancers        | 26a6049b-f5d0-4ac4-b64f-b3e6671ae6e0 |
| members              |                                      |
| name                 | pool-test2                           |
| operating_status     | OFFLINE                              |
| project_id           | 5df4e0c1c68647ef800e7904d455b58d     |
| protocol             | TCP                                  |
| provisioning_status  | PENDING_CREATE                       |
| session_persistence  | None                                 |
| updated_at           | None                                 |
| tls_container_ref    | None                                 |
| ca_tls_container_ref | None                                 |
| crl_container_ref    | None                                 |
| tls_enabled          | False                                |
+----------------------+--------------------------------------+
[lookback@LookdeMacBook-Pro ~/OpenStack]$
```
### 添加监控监控
``` sh
[lookback@LookdeMacBook-Pro ~/OpenStack]$ openstack loadbalancer healthmonitor create --delay 3 --type TCP --max-retries 3 --timeout 3 --name healthmonitor-test2 pool-test2
+---------------------+--------------------------------------+
| Field               | Value                                |
+---------------------+--------------------------------------+
| project_id          | 5df4e0c1c68647ef800e7904d455b58d     |
| name                | healthmonitor-test2                  |
| admin_state_up      | True                                 |
| pools               | 6850d049-2d41-4e47-bf4b-f930fb09784a |
| created_at          | 2020-03-22T18:07:29                  |
| provisioning_status | PENDING_CREATE                       |
| updated_at          | None                                 |
| delay               | 3                                    |
| expected_codes      | None                                 |
| max_retries         | 3                                    |
| http_method         | None                                 |
| timeout             | 3                                    |
| max_retries_down    | 3                                    |
| url_path            | None                                 |
| type                | TCP                                  |
| id                  | 7a901bb8-eb8a-490b-977d-6c756c3d9bc9 |
| operating_status    | OFFLINE                              |
| http_version        | None                                 |
| domain_name         | None                                 |
+---------------------+--------------------------------------+
[lookback@LookdeMacBook-Pro ~/OpenStack]$
```
### 给负载均衡器添加后端成员
``` sh
[lookback@LookdeMacBook-Pro ~/OpenStack]$ openstack loadbalancer member create --subnet-id e0531da2-b031-4e0a-9303-33f10d9c3aec --protocol-port 4200 --address 172.25.106.17 --name member1-test2 pool-test2 
+---------------------+--------------------------------------+
| Field               | Value                                |
+---------------------+--------------------------------------+
| address             | 172.25.106.17                        |
| admin_state_up      | True                                 |
| created_at          | 2020-03-22T17:30:28                  |
| id                  | b573666b-6d9e-423a-94cc-e6e4a4067bf0 |
| name                | member1-test2                        |
| operating_status    | NO_MONITOR                           |
| project_id          | 5df4e0c1c68647ef800e7904d455b58d     |
| protocol_port       | 4200                                 |
| provisioning_status | PENDING_CREATE                       |
| subnet_id           | e0531da2-b031-4e0a-9303-33f10d9c3aec |
| updated_at          | None                                 |
| weight              | 1                                    |
| monitor_port        | None                                 |
| monitor_address     | None                                 |
| backup              | False                                |
+---------------------+--------------------------------------+
[lookback@LookdeMacBook-Pro ~/OpenStack]$ openstack loadbalancer member create --subnet-id e0531da2-b031-4e0a-9303-33f10d9c3aec --protocol-port 4200 --address 172.25.106.18 --name member2-test2 pool-test2  
+---------------------+--------------------------------------+
| Field               | Value                                |
+---------------------+--------------------------------------+
| address             | 172.25.106.18                        |
| admin_state_up      | True                                 |
| created_at          | 2020-03-22T17:30:41                  |
| id                  | 93455ca9-b501-4509-a4bd-1fba59775363 |
| name                | member2-test2                        |
| operating_status    | NO_MONITOR                           |
| project_id          | 5df4e0c1c68647ef800e7904d455b58d     |
| protocol_port       | 4200                                 |
| provisioning_status | PENDING_CREATE                       |
| subnet_id           | e0531da2-b031-4e0a-9303-33f10d9c3aec |
| updated_at          | None                                 |
| weight              | 1                                    |
| monitor_port        | None                                 |
| monitor_address     | None                                 |
| backup              | False                                |
+---------------------+--------------------------------------+
[lookback@LookdeMacBook-Pro ~/OpenStack]$
```
### 查看
``` sh
[lookback@LookdeMacBook-Pro ~/OpenStack]$ openstack loadbalancer amphora list
+--------------------------------------+--------------------------------------+-----------+--------+----------------+----------------+
| id                                   | loadbalancer_id                      | status    | role   | lb_network_ip  | ha_ip          |
+--------------------------------------+--------------------------------------+-----------+--------+----------------+----------------+
| 8bbd6b3e-0610-40f3-b243-13fc0e0abb07 | 26a6049b-f5d0-4ac4-b64f-b3e6671ae6e0 | ALLOCATED | BACKUP | 172.29.255.113 | 172.25.255.202 |
| e3402547-f9a7-4c2d-ab31-e259af17c356 | 26a6049b-f5d0-4ac4-b64f-b3e6671ae6e0 | ALLOCATED | MASTER | 172.29.255.67  | 172.25.255.202 |
+--------------------------------------+--------------------------------------+-----------+--------+----------------+----------------+
[lookback@LookdeMacBook-Pro ~/OpenStack]$
[lookback@LookdeMacBook-Pro ~/OpenStack]$
[lookback@LookdeMacBook-Pro ~/OpenStack]$
[lookback@LookdeMacBook-Pro ~/OpenStack]$ openstack loadbalancer member list pool-test2
+--------------------------------------+---------------+----------------------------------+---------------------+---------------+---------------+------------------+--------+
| id                                   | name          | project_id                       | provisioning_status | address       | protocol_port | operating_status | weight |
+--------------------------------------+---------------+----------------------------------+---------------------+---------------+---------------+------------------+--------+
| b573666b-6d9e-423a-94cc-e6e4a4067bf0 | member1-test2 | 5df4e0c1c68647ef800e7904d455b58d | ACTIVE              | 172.25.106.17 |          4200 | NO_MONITOR       |      1 |
| 93455ca9-b501-4509-a4bd-1fba59775363 | member2-test2 | 5df4e0c1c68647ef800e7904d455b58d | ACTIVE              | 172.25.106.18 |          4200 | NO_MONITOR       |      1 |
+--------------------------------------+---------------+----------------------------------+---------------------+---------------+---------------+------------------+--------+
[lookback@LookdeMacBook-Pro ~/OpenStack]$
[lookback@LookdeMacBook-Pro ~/OpenStack]$ openstack loadbalancer list
+--------------------------------------+-------+----------------------------------+----------------+---------------------+----------+
| id                                   | name  | project_id                       | vip_address    | provisioning_status | provider |
+--------------------------------------+-------+----------------------------------+----------------+---------------------+----------+
| 26a6049b-f5d0-4ac4-b64f-b3e6671ae6e0 | test2 | 5df4e0c1c68647ef800e7904d455b58d | 172.25.255.202 | ACTIVE              | amphora  |
+--------------------------------------+-------+----------------------------------+----------------+---------------------+----------+
[lookback@LookdeMacBook-Pro ~/OpenStack]$ openstack loadbalancer show 26a6049b-f5d0-4ac4-b64f-b3e6671ae6e0          
+---------------------+--------------------------------------+
| Field               | Value                                |
+---------------------+--------------------------------------+
| admin_state_up      | True                                 |
| availability_zone   |                                      |
| created_at          | 2020-03-22T16:46:45                  |
| description         |                                      |
| flavor_id           | None                                 |
| id                  | 26a6049b-f5d0-4ac4-b64f-b3e6671ae6e0 |
| listeners           | 81d6ca4f-c6b0-4674-9eef-80511d0f97be |
| name                | test2                                |
| operating_status    | ONLINE                               |
| pools               | 6850d049-2d41-4e47-bf4b-f930fb09784a |
| project_id          | 5df4e0c1c68647ef800e7904d455b58d     |
| provider            | amphora                              |
| provisioning_status | ACTIVE                               |
| updated_at          | 2020-03-22T18:07:30                  |
| vip_address         | 172.25.255.202                       |
| vip_network_id      | 58ba4366-ecc5-46f6-8898-8b8e743797d6 |
| vip_port_id         | abe786ea-7c9a-4937-816d-edd996dde5a5 |
| vip_qos_policy_id   | None                                 |
| vip_subnet_id       | e0531da2-b031-4e0a-9303-33f10d9c3aec |
+---------------------+--------------------------------------+
[lookback@LookdeMacBook-Pro ~/OpenStack]$ openstack loadbalancer listener show 81d6ca4f-c6b0-4674-9eef-80511d0f97be
+-----------------------------+--------------------------------------+
| Field                       | Value                                |
+-----------------------------+--------------------------------------+
| admin_state_up              | True                                 |
| connection_limit            | -1                                   |
| created_at                  | 2020-03-22T16:50:42                  |
| default_pool_id             | 6850d049-2d41-4e47-bf4b-f930fb09784a |
| default_tls_container_ref   | None                                 |
| description                 |                                      |
| id                          | 81d6ca4f-c6b0-4674-9eef-80511d0f97be |
| insert_headers              | None                                 |
| l7policies                  |                                      |
| loadbalancers               | 26a6049b-f5d0-4ac4-b64f-b3e6671ae6e0 |
| name                        | listener-test2                       |
| operating_status            | ONLINE                               |
| project_id                  | 5df4e0c1c68647ef800e7904d455b58d     |
| protocol                    | TCP                                  |
| protocol_port               | 4200                                 |
| provisioning_status         | ACTIVE                               |
| sni_container_refs          | []                                   |
| timeout_client_data         | 50000                                |
| timeout_member_connect      | 5000                                 |
| timeout_member_data         | 50000                                |
| timeout_tcp_inspect         | 0                                    |
| updated_at                  | 2020-03-22T18:07:30                  |
| client_ca_tls_container_ref | None                                 |
| client_authentication       | NONE                                 |
| client_crl_container_ref    | None                                 |
| allowed_cidrs               |                                      |
+-----------------------------+--------------------------------------+
[lookback@LookdeMacBook-Pro ~/OpenStack]$ openstack loadbalancer pool show 6850d049-2d41-4e47-bf4b-f930fb09784a
+----------------------+--------------------------------------+
| Field                | Value                                |
+----------------------+--------------------------------------+
| admin_state_up       | True                                 |
| created_at           | 2020-03-22T16:54:22                  |
| description          |                                      |
| healthmonitor_id     | 7a901bb8-eb8a-490b-977d-6c756c3d9bc9 |
| id                   | 6850d049-2d41-4e47-bf4b-f930fb09784a |
| lb_algorithm         | SOURCE_IP                            |
| listeners            | 81d6ca4f-c6b0-4674-9eef-80511d0f97be |
| loadbalancers        | 26a6049b-f5d0-4ac4-b64f-b3e6671ae6e0 |
| members              | b573666b-6d9e-423a-94cc-e6e4a4067bf0 |
|                      | 93455ca9-b501-4509-a4bd-1fba59775363 |
| name                 | pool-test2                           |
| operating_status     | ONLINE                               |
| project_id           | 5df4e0c1c68647ef800e7904d455b58d     |
| protocol             | TCP                                  |
| provisioning_status  | ACTIVE                               |
| session_persistence  | None                                 |
| updated_at           | 2020-03-22T18:07:30                  |
| tls_container_ref    | None                                 |
| ca_tls_container_ref | None                                 |
| crl_container_ref    | None                                 |
| tls_enabled          | False                                |
+----------------------+--------------------------------------+
[lookback@LookdeMacBook-Pro ~/OpenStack]$ openstack loadbalancer healthmonitor show 7a901bb8-eb8a-490b-977d-6c756c3d9bc9
+---------------------+--------------------------------------+
| Field               | Value                                |
+---------------------+--------------------------------------+
| project_id          | 5df4e0c1c68647ef800e7904d455b58d     |
| name                | healthmonitor-test2                  |
| admin_state_up      | True                                 |
| pools               | 6850d049-2d41-4e47-bf4b-f930fb09784a |
| created_at          | 2020-03-22T18:07:29                  |
| provisioning_status | ACTIVE                               |
| updated_at          | 2020-03-22T18:07:30                  |
| delay               | 3                                    |
| expected_codes      | None                                 |
| max_retries         | 3                                    |
| http_method         | None                                 |
| timeout             | 3                                    |
| max_retries_down    | 3                                    |
| url_path            | None                                 |
| type                | TCP                                  |
| id                  | 7a901bb8-eb8a-490b-977d-6c756c3d9bc9 |
| operating_status    | ONLINE                               |
| http_version        | None                                 |
| domain_name         | None                                 |
+---------------------+--------------------------------------+
[lookback@LookdeMacBook-Pro ~/OpenStack]$ openstack loadbalancer member show 6850d049-2d41-4e47-bf4b-f930fb09784a b573666b-6d9e-423a-94cc-e6e4a4067bf0
+---------------------+--------------------------------------+
| Field               | Value                                |
+---------------------+--------------------------------------+
| address             | 172.25.106.17                        |
| admin_state_up      | True                                 |
| created_at          | 2020-03-22T17:30:28                  |
| id                  | b573666b-6d9e-423a-94cc-e6e4a4067bf0 |
| name                | member1-test2                        |
| operating_status    | ONLINE                               |
| project_id          | 5df4e0c1c68647ef800e7904d455b58d     |
| protocol_port       | 4200                                 |
| provisioning_status | ACTIVE                               |
| subnet_id           | e0531da2-b031-4e0a-9303-33f10d9c3aec |
| updated_at          | 2020-03-22T18:07:35                  |
| weight              | 1                                    |
| monitor_port        | None                                 |
| monitor_address     | None                                 |
| backup              | False                                |
+---------------------+--------------------------------------+
[lookback@LookdeMacBook-Pro ~/OpenStack]$
```
### 测试可用性
``` sh
[lookback@LookdeMacBook-Pro ~/OpenStack]$ telnet 172.25.255.202 4200
Trying 172.25.255.202...
Connected to 172.25.255.202.
Escape character is '^]'.
^]
telnet> quit
Connection closed.
[lookback@LookdeMacBook-Pro ~/OpenStack]$
[lookback@LookdeMacBook-Pro ~/OpenStack]$
[lookback@LookdeMacBook-Pro ~/OpenStack]$
[lookback@LookdeMacBook-Pro ~/OpenStack]$ curl -Lks -vv http://172.25.255.202:4200
*   Trying 172.25.255.202...
* TCP_NODELAY set
* Connected to 172.25.255.202 (172.25.255.202) port 4200 (#0)
> GET / HTTP/1.1
> Host: 172.25.255.202:4200
> User-Agent: curl/7.64.1
> Accept: */*
> 
< HTTP/1.1 200 OK
< content-type: application/json
< content-length: 330
< 
{
  "ok" : true,
  "status" : 200,
  "name" : "crate_node_2",
  "cluster_name" : "dsmnjdc_history_bet_cluster",
  "version" : {
    "number" : "4.0.3",
    "build_hash" : "1b7058fa16f6dfc833beb1b4fe2263c105c202f3",
    "build_timestamp" : "2019-08-09T01:20:38Z",
    "build_snapshot" : false,
    "lucene_version" : "8.0.0"
  }
}
* Connection #0 to host 172.25.255.202 left intact
* Closing connection 0
[lookback@LookdeMacBook-Pro ~/OpenStack]$
```