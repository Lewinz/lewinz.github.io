---
layout: wiki
title: 中间件部署文档
categories: 中间件
description: 中间件部署文档
keywords: 中间件, 部署
---


# JDK

## rpm 方式
上传准备好的离线安装包到服务器 
安装  
`rpm -ivh jdk-8u181-linux-x64.rpm`

## 压缩包安装
上传准备好的压缩包到服务器  

切换到安装目录  
`tar -zvxf /mnt/package/jdk-8u181-linux-x64.tar.gz`  

修改环境变量  
`vim /etc/profile`  

添加以下内容  
``` shell
export JAVA_HOME=/mnt/app/jdk
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib:$CLASSPATH
export JAVA_PATH=${JAVA_HOME}/bin:${JRE_HOME}/bin
export PATH=$PATH:${JAVA_PATH}
```

重新生效配置  
`source /etc/profile`
## 检查
输入`java -version`出现信息则说明安装成功

# Postgresql  
## 在线安装

1. 安装 RPM  
`yum install`

2. 安装客户端 (若安装服务器，则不需要)  
`yum install postgresql12`

3. 安装服务端  
`yum install postgresql12-server`
 
## 离线安装

**rpm 包**  

准备好如下 rpm 安装包
>libicu-50.2-4.el7_7.x86_64.rpm  
>libxslt-1.1.28-5.el7.x86_64.rpm  
>postgresql12-libs-12.4-1PGDG.rhel7.x86_64.rpm  
>postgresql12-12.4-1PGDG.rhel7.x86_64.rpm  
>postgresql12-server-12.4-1PGDG.rhel7.x86_64.rpm  
>postgresql12-contrib-12.4-1PGDG.rhel7.x86_64.rpm  

**安装 rpm 包 <按顺序执行>**
``` shell
rpm -ivh libicu-50.2-4.el7_7.x86_64.rpm --force --nodeps
 
rpm -ivh libxslt-1.1.28-5.el7.x86_64.rpm --force --nodeps
 
rpm -ivh postgresql12-libs-12.4-1PGDG.rhel7.x86_64.rpm --force --nodeps
 
rpm -ivh postgresql12-12.4-1PGDG.rhel7.x86_64.rpm --force --nodeps
 
rpm -ivh postgresql12-server-12.4-1PGDG.rhel7.x86_64.rpm --force --nodeps
 
rpm -ivh postgresql12-contrib-12.4-1PGDG.rhel7.x86_64.rpm --force --nodeps
```

**配置步骤**

始化数据库并启用自动启动 （可选）  
初始化数据库  
`/usr/pgsql-12/bin/postgresql-12-setup initdb`

设置开机启动 pg 服务  
`systemctl enable postgresql-12`
 
启动 pg 服务  
`systemctl start postgresql-12`

修改 PostgreSQL 远程连接配置  
`vim /var/lib/pgsql/12/data/pg_hba.conf`

``` shell
local   all             all                                     peer
host    all             all             127.0.0.1/32            trust
host    all             all             all                     md5
host    replication     all             127.0.0.1/32            trust
host    replication     all             all                     md5
```

修改配置  

`vim /var/lib/pgsql/12/data/postgresql.conf`

``` shell
# 新增以下配置
# 放开远程连接 ip 限制
listen_addresses = '*'
# 配置 pg_stat_statements
shared_preload_libraries = 'pg_stat_statements'
track_activity_query_size = 2048
# 配置流复制模式，允许 debezium cdc 组件实时抓取 
wal_level = logical
max_wal_senders = 10
max_replication_slots = 10
pg_stat_statements.track = all

# 修改以下配置
max_wal_size = 1GB
min_wal_size = 100MB
```

重启服务  
`systemctl restart postgresql-12`

修改 postgres 账号密码  
使用 postgres 登入数据库  
`psql -h 127.0.0.1 -d postgres -U postgres`

修改 postgres 密码  
`alter user postgres with password 'postgres';`

加载 pg_stat_statements  
`CREATE EXTENSION pg_stat_statements;`

**挂载外部数据盘 ps: mnt 为任意挂载盘**  
停止 pgsql   
`systemctl stop postgresql-12`


编辑`vim /usr/lib/systemd/system/postgresql-12.service` 

修改文件中 PGDATA 路径配置为外部数据盘相应文件夹地址  
配置由`Environment=PGDATA=/var/lib/pgsql/12/data/`  
变为`Environment=PGDATA=/mnt/pg/data/`  
 
将文件夹复制到外部数据盘  
`mv /var/lib/pgsql/12/data/ /mnt/pg/data/`

将数据目录文件夹所有者改为 postgres 用户, postgres 用户组  
`chown postgres:postgres /mnt/pg/data/ -R`
 
访问权限改为 750 或 700  
`chmod 750 -R /mnt/pg/data/`  
执行 `systemctl daemon-reload`  
以后修改配置文件就在/mnt/pg/data 下面修改

重启服务  
`systemctl restart postgresql-12`  

## Postgresql 主从  

安装 PG 从库  
按照前面的安装操作，在另一台服务器安装从库 pg。    
**！！！从库在部署时不能初始化数据库，否则会导致主从库序列号不一致。**

**配置步骤**  

准备流复制角色 ps: 密码可更改  
在主库中创建进行流复制的角色  
``` shell
CREATE ROLE repuser WITH
  LOGIN
  REPLICATION
  CONNECTION LIMIT 5
  PASSWORD '1qazXSW@3edc';
```

修改主库配置  
打开主库配置文件夹  
`cd /mnt/pg/data`
 
`vim pg_hba.conf` 在最后一行添加  

`host replication repuser 从库 ip/32 md5`
 
`vim postgresql.conf` 修改以下变量  
``` shell
archive_mode = on
archive_command = '/bin/date'
max_wal_senders = 10
wal_keep_segments = 16
synchronous_standby_names = '*'
```
重启主库 使配置生效   

从库配置  
使用以下命令 在两边服务器中检查， 主从服务器连通性  
``` shell
    ping ip
    telnet ip 5432
```

确认无误后，在从库服务器执行以下命令，备份主库数据
``` shell
cd /mnt/pg/     打开从库 pg 的数据文件夹处  
pg_basebackup -R -D /mnt/pg/backup -Fp -Xs -v -P -h 主库 ip -p 5432 -U repuser  
```

将原数据文件夹备份 再将 backup 改名为 data  
更改 data 文件夹的用户组，权限 
``` shell
chown postgres:postgres data -R
chmod 750 -R data
```

检查 postgresql.auto.conf 文件里是否包含如下内容：
``` shell
primary_conninfo = 'user=repuser password=''1qazXSW@3edc'' host=主库 IP port=5432 sslmode=prefer sslcompression=0 gssencmode=prefer krbsrvname=postgres target_session_attrs=any'
```

重启从库 PG  
检查主从是否成功  

在主库中查询流复制信息  
`select * from pg_stat_replication;`
 
在主库中添加或修改数据，在从库中查看。 

# MySql

## 单节点安装

检查插件  
``` shell
# 会与 mysql 冲突，如果预装了，先卸载
rpm -qa|grep mariadb
rpm -e --nodeps mariadb-xxxxxxxx.x86_64

# mysql 安装依赖插件，如果没有，先安装
rpm -qa|grep libaio
yum install libaio
```

``` shell
# 安装 mysql
rpm -ivh mysql-community-common-5.7.22-1.el7.x86_64.rpm
rpm -ivh mysql-community-libs-5.7.22-1.el7.x86_64.rpm
rpm -ivh mysql-community-client-5.7.22-1.el7.x86_64.rpm
rpm -ivh mysql-community-server-5.7.22-1.el7.x86_64.rpm
```
> 安装如果报错`perl(Getopt::Long) is needed`  
> 执行`yum install perl`  

> 如果报错`warning: mysql-community-server-5.7.22-1.el7.x86_64.rpm: Header V3 DSA/SHA1 Signature, key ID 5072e1f5: NOKEY`  
> 解决办法：`在 rpm 命令后加上 --force --nodeps`

``` shell
# 初始化数据库，自动创建 data 文件目录
mysqld --initialize-insecure --user=mysql

# 将`/var/lib/mysql`文件所有用户修改为 mysql
chown mysql:mysql /var/lib/mysql -R
```
``` shell
# 替换 mysql 配置
vi /etc/my.cnf

[mysqld]
# 全局配置
port                                          = 3306                            # 端口
datadir                                       = /var/lib/mysql                  # 数据存放目录
socket                                        = /var/lib/mysql/mysql.sock       # 同 host 连接文件
pid-file                                      = /var/lib/mysql/mysqld.pid       # pid 存储文件
user                                          = mysql                           # 账号配置
default_storage_engine                        = InnoDB                          # 默认引擎
symbolic-links                                = 0                               # 禁用文件软连接
character-set-client-handshake                = FALSE                           # 在客户端字符集和服务端字符集不同的时候将拒绝连接到服务端执行任何操作
character-set-server                          = utf8mb4                         # 默认字符集
collation-server                              = utf8mb4_general_ci              # 默认字符集编码
init_connect                                  = 'SET NAMES utf8mb4'             # 执行第一次查询之前执行此语句，统一字符集
wait_timeout                                  = 31536000                        # 连接空闲超时时间，秒
interactive_timeout                           = 31536000                        # 连接空闲超时时间，秒

skip_name_resolve                                         # 关闭域名解析，避免dns导致的请求失败(关闭后要使用IP访问mysql)，mysql 会做正向和反向dns查询。一旦失败就会拒绝连接。


# innodb 引擎配置
innodb_buffer_pool_size = 256M                            # 引擎缓冲池大小，根据实际调整
innodb_log_file_size    = 50M                             # 引擎日志文件大小，根据实际调整
innodb_file_per_table   = 1                               # 开启独立表空间，开启后每个表都有自已独立的表空间，每个表的数据和索引都会存在自已的表空间中
innodb_flush_method     = O_DIRECT                        # innodb 使用 O_DIRECT 打开数据文件，使用 fsync () 刷写数据文件跟 redo log

# LOGGING
log-error               = /var/log/mysql-error.log        # 错误日志存放位置
slow_query_log          = ON                              # 开启慢日志
slow_query_log_file     = /var/log/mysql-slow.log         # 慢日志存放位置
long_query_time         = 3                               # 慢日志阈值，秒

# OTHER
tmp_table_size          = 32M                             # 临时表内存缓存大小
max_heap_table_size     = 32M                             # MEMORY 内存引擎的表大小
max_connections         = 100                             # 最大连接数
thread_cache_size       = 50                              # 线程池缓存大小
table_open_cache        = 10                              # 表文件描述符的缓存大小
open_files_limit        = 65535                           # 文件描述符限制数量

[client]
default-character-set   = utf8mb4                         # 默认字符集

[mysql]
default-character-set   = utf8mb4                         # 默认字符集
```

启动 mysql  
`systemctl start mysqld.service`

登陆 mysql 修改密码  
``` shell
mysql -u root
 
update mysql.user set authentication_string = password('123456') where user = 'root' and host = 'localhost';
 
grant all privileges on *.* to 'root'@'%' identified by '123456' with grant option;
 
flush privileges;
```

退出 mysql 后，设置开机自启，重启 mysql
``` shell
systemctl stop mysqld.service
systemctl enable mysqld.service
systemctl list-unit-files | grep mysqld
systemctl start mysqld.service
```

linux 上 Mysql 登录
``` shell
# 使用 root 登录，-p 代表有密码
mysql -u root -h 127.0.0.1 -p
```

## 主从模式安装

**主节点修改**  

`vim /etc/my.cnf`

``` shell
## 设置 server_id，一般设置为 IP,注意要唯一
server-id=125107
## 复制过滤：也就是指定哪个数据库不用同步（mysql 库一般不同步）
binlog-ignore-db=mysql
binlog-ignore-db=sys
binlog-ignore-db=information_schema
binlog-ignore-db=performance_schem
## 开启二进制日志功能，可以随便取，最好有含义（关键就是这里了）
log-bin=mysql-bin
## 为每个 session 分配的内存，在事务过程中用来存储二进制日志的缓存
binlog_cache_size=16M
## 主从复制的格式（mixed,statement,row，默认格式是 statement）
binlog_format=mixed
## 二进制日志自动删除/过期的天数。默认值为 0，表示不自动删除。
expire_logs_days=30
## 跳过主从复制中遇到的所有错误或指定类型的错误，避免 slave 端复制中断。
## 如：1062 错误是指一些主键重复，1032 错误是因为主从数据库数据不一致
slave_skip_errors=1062
## 控制 binlog 的写入频率。每执行多少次事务写入一次
## 这个参数性能消耗很大，但可减小 MySQL 崩溃造成的损失，为 0 表示不控制
sync_binlog = 1
innodb_flush_log_at_trx_commit = 1
```

重启主节点服务  
``` shell
systemctl restart mysqld.service
```

创建同步用户  
``` shell
# 创建给从节点使用
CREATE USER 'slave'@'%' IDENTIFIED BY 'a123456';

GRANT REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'slave'@'%';
```

**从节点修改**  

`vim /etc/my.cnf`

``` shell
## 设置 server_id，一般设置为 IP,注意要唯一
server-id=125111
## 复制过滤：也就是指定哪个数据库不用同步（mysql 库一般不同步）
binlog-ignore-db=mysql
binlog-ignore-db=sys
binlog-ignore-db=information_schema
binlog-ignore-db=performance_schem
## 开启二进制日志功能，以备 Slave 作为其它 Slave 的 Master 时使用
#log-bin=mysql-slave1-bin
## 为每个 session 分配的内存，在事务过程中用来存储二进制日志的缓存
binlog_cache_size=16M
## 主从复制的格式（mixed,statement,row，默认格式是 statement）
binlog_format=mixed
## 二进制日志自动删除/过期的天数。默认值为 0，表示不自动删除。
expire_logs_days=30
## 跳过主从复制中遇到的所有错误或指定类型的错误，避免 slave 端复制中断。
## 如：1062 错误是指一些主键重复，1032 错误是因为主从数据库数据不一致
slave_skip_errors=1062
## relay_log 配置中继日志
relay_log=mysql-relay-bin
## log_slave_updates 表示 slave 将复制事件写进自己的二进制日志
## 主要为了作为其他的 master
#log_slave_updates=ON
## 防止改变数据 (除了特殊的线程)
#read_only=1(为了使备机随时转正，所以这里允许写)
## MySQL 主从复制的时候，当 Master 和 Slave 之间的网络中断，但是 Master 和 Slave 无法察觉的情况下（比如防火墙或者路由问题）。Slave 会等待 slave_net_timeout 设置的秒数后，才能认为网络出现故障，然后才会重连并且追赶这段时间主库的数据,默认 60
slave-net-timeout = 20                    
## 如果启用，此变量将在服务器启动后立即启用自动中继日志恢复。
relay_log_recovery = ON
## 该变量确定从站在中继日志中的位置是写入 FILE 还是写入表
relay_log_info_repository = TABLE
```

重启从节点服务  
``` shell
systemctl restart mysqld.service
```

登录从节点 mysql
``` shell
# 命令信息需要修改，在主节点执行命令 show master status\G; 获取
change master to master_host='47.117.136.250', master_user='slave', master_password='a123456', master_port=3306, master_log_file='mysql-bin.000002', master_log_pos=154, master_connect_retry=30,master_heartbeat_period=10;
```

开始主从复制  

`start slave;`

查看主从同步状态  
`show slave status\G;`

Slave_IO_Running/Slave_SQL_Running 两个属性都为 YES 即为复制成功

若主从复制信息设置错误，可通过下列命令重置信息  
``` shell
# 停止已经启动的绑定
stop slave;
# 重置绑定
reset master;
# 启动复制
start slave;
```

**常见错误：**  

查看错误日志
`tail -f -n 1000 /var/log/mysqld.log`

errNum:   
**1236** 复制信息有误，重新设置。  

errNum:   
**1593** 在 mysql 中使用 `show variables like '%server_%';` 查看 server_id/server_uuid 是否配置生效，server_id 配置在 my.cnf 中，server_uuid 在 auto.cnf 中。  

# Nginx

## 安装相关依赖
`yum -y install gcc gcc-c++ pcre pcre-devel zlib zlib-devel openssl openssl-devel gd gd-devel`  

## 安装 nginx
``` shell
# 下载tar包
wget http://nginx.org/download/nginx-1.17.3.tar.gz
tar -xvf nginx-1.17.3.tar.gz
cd nginx-1.17.3

./configure --prefix=/usr/local/nginx --with-http_stub_status_module --with-http_ssl_module --with-http_v2_module
# prefix 安装目录
# with-http_stub_status_module 监控状态
# with-http_ssl_module 开启 ssl
# with-http_v2_module 开启 http2

make
make install
```

## nginx 配置
生成 diffie-hellman 4096 位加密秘钥
``` shell
cd /etc/nginx/

openssl dhparam -out dhparam.pem 2048
```

全局配置
``` shell
# /usr/local/nginx/conf/nginx.conf
user                 root;
pid                  /var/run/nginx.pid;
worker_processes     auto;
worker_rlimit_nofile 65535;

events {
    multi_accept       on;
    worker_connections 65535;
}

http {
    charset                utf-8;
    sendfile               on;
    tcp_nopush             on;
    tcp_nodelay            on;
    server_tokens          off;
    log_not_found          off;
    types_hash_max_size    2048;
    types_hash_bucket_size 64;
    client_max_body_size   16M;

    # MIME
    include                mime.types;
    default_type           application/octet-stream;

    # Logging
    access_log             /var/log/nginx/access.log;
    error_log              /var/log/nginx/error.log warn;

    # SSL
    ssl_session_timeout    1d;
    ssl_session_cache      shared:SSL:10m;
    ssl_session_tickets    off;

    # Diffie-Hellman parameter for DHE ciphersuites
    ssl_dhparam            /etc/nginx/dhparam.pem;

    # Mozilla Intermediate configuration
    ssl_protocols          TLSv1.2 TLSv1.3;
    ssl_ciphers            ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384;

    # OCSP Stapling
    ssl_stapling           on;
    ssl_stapling_verify    on;
    resolver               1.1.1.1 1.0.0.1 8.8.8.8 8.8.4.4 208.67.222.222 208.67.220.220 valid=60s;
    resolver_timeout       2s;

    # Connection header for WebSocket reverse proxy
    map $http_upgrade $connection_upgrade {
        default upgrade;
        ""      close;
    }

    # Load configs
    include /etc/nginx/conf.d/*.conf;
}
```

详细配置
``` shell
# /etc/nginx/conf.d/tracehabit.conf
server {
    listen              443 ssl http2;
    listen              [::]:443 ssl http2;
    server_name         tracehabit;

    # SSL
    ssl_certificate     /root/.acme.sh/tracehabit.com.csr;
    ssl_certificate_key /root/.acme.sh/tracehabit.com.key;

    # security
    # include             nginxconfig.io/security.conf;

    # logging
    access_log          /var/log/nginx/tracehabit.access.log;
    error_log           /var/log/nginx/tracehabit.error.log warn;

    proxy_set_header Authorization $http_authorization;
    proxy_pass_header  Authorization;

    # reverse proxy
    location / {
        proxy_pass http://127.0.0.1:9090;
       # include    nginxconfig.io/proxy.conf;
    }

    # additional config
    #include nginxconfig.io/general.conf;
}

# HTTP redirect
server {
    listen      80;
    listen      [::]:80;
    server_name tracehabit;
    return      301 https://tracehabit$request_uri;
}
```

## nginx 启停
``` shell
# 检查配置文件是否正确
/usr/local/nginx/sbin/nginx -t  

# 启动
/usr/local/nginx/sbin/nginx

# 关闭
/usr/local/nginx/sbin/nginx -s stop

# 重启
/usr/local/nginx/sbin/nginx -s reload
```

# Nacos

## 单机部署

**通过源码或者发行包获取**  

从 Github 上下载源码方式
``` shell
git clone https://github.com/alibaba/nacos.git
cd nacos/
mvn -Prelease-nacos clean install -U  
ls -al distribution/target/
cd distribution/target/nacos-server-$version/nacos/bin
```

下载编译后压缩包方式  

``` shell
https://github.com/alibaba/nacos/releases
unzip nacos-server-$version.zip 或者 tar -xvf nacos-server-$version.tar.gz
cd nacos/bin
```

**单机启动**
`sh startup.sh -m standalone`

关闭服务
`sh shutdown.sh`

## 集群部署

安装数据库，版本要求：5.6.5+  

初始化 mysql 数据库，数据库初始化文件：nacos-mysql.sql  

修改 conf/application.properties 文件，添加 mysql 数据源的 url、用户名和密码。  

``` shell
spring.datasource.platform=mysql
db.num=1
db.url.0=jdbc:mysql://127.0.0.1:3306/nacos?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true
db.user=user_name
db.password=pwd
```

修改 conf 目录下 cluster.conf 文件  
``` shell
xxx.xxx.xxx.16:8848
xxx.xxx.xxx.17:8848
xxx.xxx.xxx.18:8848
```

配置 mysql 数据库  

无参模式启动 startup.sh 脚本  
`sh startup.sh`

# Kafka

## zookeeper 单节点安装（可使用 kafka 自带）

**解压 zookeeper 压缩包（必须是带"bin"的压缩包，否则启动会报错）**
``` shell
cd /mnt/zookeeper/
tar -zxvf apache-zookeeper-3.6.2-bin.tar.gz
```

**修改配置文件**  
``` shell
cd /mnt/zookeeper/apache-zookeeper-3.6.2-bin/conf
cp zoo_sample.cfg zoo.cfg
vi zoo.cfg
```

**配置数据路径和日志路径**
``` shell
dataDir=/mnt/data/zookeeper
dataLogDir=/mnt/logs/zookeeper

# 自动清理日志 <非必须> 3.4 之后版本可配置
autopurge.snapRetainCount=3
autopurge.purgeInterval=1
```

**配置环境变量**
``` shell
export ZOOKEEPER_INSTALL=/mnt/zookeeper/apache-zookeeper-3.6.2-bin/
export PATH=$PATH:$ZOOKEEPER_INSTALL/bin
```

**启动 zookeeper 服务**

`/mnt/zookeeper/apache-zookeeper-3.6.2-bin/bin/zkServer.sh start`  

**启动成功**  
``` shell
ZooKeeper JMX enabled by default
Using config: /usr/local/zookeeper-3.4.13/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED
```

## zookeeper 集群安装

多台服务器 zookeeper 单节点安装完毕之后

在 zoo.cfg 配置的 data 目录下新增 myid 文件
`	echo 1 > /mnt/software/zookeeper/data/myid`

`vim /mnt/software/zookeeper/conf/zoo.cfg`

``` shell
# ip:port:port<port:port 为选举 leader 使用，默认 2888:3888，如果是一台服务器部署集群，使用多个不同端口即可>
server.1=xx.xx.xx.xx:2888:3888
server.2=xx.xx.xx.xx:2888:3888
server.3=xx.xx.xx.xx:2888:2888
```

启动服务。

# kafka

## 单节点部署

解压 kafka 压缩包  
``` shell
cd /mnt/tools

tar -zxvf kafka_2.13-2.6.0.tgz
mv kafka_2.13-2.6.0 /mnt/software/kafka

cd /mnt/software/kafka
mkdir zookeeper
mkdir log
mkdir log/zookeeper
mkdir log/kafka
```

修改 zookeeper 配置  
``` shell
vim config/zookeeper.properties
dataDir= /mnt/software/kafka/zookeeper
```

修改 kafka 配置 
``` shell
vim config/server.properties
#修改这几项
log.dirs=/mnt/software/kafka/log/kafka  #日志存放路径
listeners=PLAINTEXT://10.17.64.110:9092 #IP 填本机地址 外网/内网可访问地址
zookeeper.connect=10.17.64.110:2181  #IP 填 zookeeper 安装地址
```

先启动 zookeeper  
``` shell
sh bin/zookeeper-server-start.sh -daemon  config/zookeeper.properties
sh bin/kafka-server-start.sh -daemon config/server.properties
```
 
创建 Topic  
``` shell
bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic test
```

查看 topic 列表  

`bin/kafka-topics.sh --list --zookeeper localhost:2181`
 
查看描述 topics 信息  

`bin/kafka-topics.sh --describe --zookeeper localhost:2181 --topic test`
 
启动生产者  

`bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test`
 
启动消费者 

``
bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test --from-beginning
--from-beginning 表示从起始位读取
``
 
启动先 zookeeper 停止先 kafka  

## 集群部署
复制 kafka 文件夹到其它服务器  

在/mnt/kafka/zookeeper 下添加 myid 文件，写入服务 broker.id 属性值  
例如 echo 0 > myid  

修改 zookeeper 配置文件  
`config/zookeeper.properties`

``` shell
#注释掉
#maxClientCnxns=0

#设置连接参数，添加如下配置
tickTime=2000　　　　#为 zk 的基本时间单元，毫秒
initLimit=10　　　　 #Leader-Follower 初始通信时限 tickTime*10
syncLimit=5　　　　　#Leader-Follower 同步通信时限 tickTime*5
 
#设置 broker Id 的服务地址 id 值与服务器 IP 对应
server.0=10.17.64.110:2888:3888
server.1=10.17.64.111:2888:3888
server.2=10.17.64.112:2888:3888
```

修改 kafka 配置文件 config/server.properties  

``` shell
broker.id=0  # kafka 实例的唯一标识，用整数表示，使用不同的整数值将集群中的 kafka 实例区分即可

# topic 在当前 broker 上的分片个数，与 broker 保持一致
num.partitions=3
zookeeper.connect=node1:2181,node2:2181,node3:2181
 
1. broker.id：同一个集群中，每台机器均不能一样
2. zookeeper.connect：有几台 zookeeper 服务器，设置为几台台，必须全部加进去
3. listeners：在配置集群的时候，必须设置，不然以后的操作会报找不到 leader 的错误
```

配置完成后先启动 zookeeper 保证服务都启动后在启动 kafka  

# ES 集群搭建

解压安装包

安装 rpm  
`rpm -ivh elasticsearch-7.8.0-x86_64.rpm`

更新 es 配置  
`vim /etc/elasticsearch/elasticsearch.yml`

``` shell
# Use a descriptive name for your cluster:
#
cluster.name: elasticsearch
///节点名称
# Use a descriptive name for the node:
#
node.name: 'es-node1'
///数据以及日志存放路径
# Path to directory where to store the data (separate multiple locations by comma):
#
path.data: /data/elasticsearch/data
#
# Path to log files:
#
path.logs: /data/elasticsearch/logs
///本机 ip 以及端口
# Set the bind address to a specific IP (IPv4 or IPv6):
#
network.host: 10.0.209.47
network.bind_host: 10.0.209.47
network.publish_host: 10.0.209.47
#
# Set a custom port for HTTP:
#
http.port: 9200
///集群 ip 配置以及主节点数（计算方式：total number of master-eligible nodes / 2 + 1）
# Pass an initial list of hosts to perform discovery when new node is started:
# The default list of hosts is ["127.0.0.1", "[::1]"]
#
discovery.zen.ping.unicast.hosts: ["10.0.209.47", "10.0.209.44","10.0.209.43"]
#
# Prevent the "split brain" by configuring the majority of nodes (total number of master-eligible nodes / 2 + 1):
#
discovery.zen.minimum_master_nodes: 2
```

设置 jvm 参数  
`vim etc/elasticsearch/jvm.options  （内存允许 设置 31g）`

``` shell
-Xms31g
-Xmx31g
```

配置密码 （公网环境需要设置，客户服务器视情况而定  内网访问可不需要密码）  

执行以下命令输入 ElasticSearch 的密码

`/usr/share/elasticsearch/bin/elasticsearch-setup-passwords interactive`

配置开机启动
``` shell
systemctl enable elasticsearch.service
systemctl start elasticsearch
systemctl status elasticsearch
```

# Redis
## 单节点部署
下载安装包
``` shell
# 创建目录 /mnt/software
cd /mnt/software

wget http://download.redis.io/releases/redis-5.0.5.tar.gz

tar -zxvf redis-5.0.5.tar.gz

mv redis-5.0.5 redis
```

编译安装包
``` shell
cd redis

make MALLOC=libc && make install
```
> 如果在此步报错  
> cc: 错误：../deps/hiredis/libhiredis.a：没有那个文件或目录  
> cc: 错误：../deps/lua/src/liblua.a：没有那个文件或目录  
> 切换至 deps/执行  
> `make lua hiredis linenoise`  
> 再执行`make MALLOC=libc && make install`即可  

启动 redis 服务
``` shell
/mnt/software/redis/src/redis-server
```

### Redis 配置成服务
配置文件
``` shell
# 拷贝配置
cp /mnt/software/redis/redis.conf /etc/redis/redis.conf
cp /mnt/software/redis/utils/redis_init_script /etc/init.d/redis

# 后台启动
daemonize yes
```

`vi /etc/redis/redis.conf`
``` shell
# 去掉 bind 配置
bind 127.0.0.1
# 日志存放位置
logfile /mnt/software/redis/logs/redis.log
# 数据文件存放位置
dir /mnt/software/redis/data

# 密码设置
requirepass redis_password
```

vi /etc/init.d/redis
``` shell
# 修改
CONF="/etc/redis/redis.conf"
```

加入 systemctl
``` shell
systemctl enable redis

# 启停
systemctl stop redis
systemctl start redis
```

## Redis 配置主从复制  

1）各结点启动 redis 服务  

`sudo systemctl start redis`  

2）登陆从结点，查看主从复制情况  

`/data/redis/redis-5.0.5/src/redis-cli info Replication`

3）设置开机自启  

`$ sudo /sbin/chkconfig redis on`  

从 redis149 上配置文件 6379.conf 添加  
``` shell
slaveof 10.16.212.224 6379
slave-read-only yes
```

4）验证  

执行 redis-cli 连接到 redis-service  

执行 info Replication 命令查看状态  

看到从节点上有  

master_host: 10.16.212.224 以及 slave_read_only:1 字样即为配置成功  

## 集群部署
1. 先部署一个单节点 Redis，将 /mnt/softwore/redis 拷贝到其他机器上
> 如果是一台机器上部署，可以拷贝在同一文件夹，并修改文件夹名称为占用端口号，便于区分

2. 修改 redis.conf 配置

``` shell
# 后台启动
daemonize yes

# 端口（每个实例都不一样，按需修改）
port 8001

# 数据文件存放位置，也可以直接使用 ./ 存放在当前目录下，便于区分
dir /usr/local/redis-cluster/8001/

# 启动集群模式
cluster-enabled yes

# 集群节点信息文件，每个实例配置不重复即可（每个实例都不一样，按需修改）
cluster-config-file nodes-8001.conf

# 删除 IP 绑定信息
bind 127.0.0.1

# 关闭保护模式
protected-mode no

# 开启数据持久化
appendonly yes

# 密码
requirepass redis_password

# 主从复制密码
masterauth redis_password
```

3. 启动所有实例

``` shell
/mnt/software/redis/src/redis-server /mnt/software/redis/redis.conf
```

4. 创建集群（使用 redis-cli）

``` shell
/mnt/software/redis/src/redis-cli -a xxx --cluster create --cluster-replicas 1 192.168.5.100:8001 192.168.5.100:8002 192.168.5.100:8003 192.168.5.100:8004 192.168.5.100:8005 192.168.5.100:8006

-a 密码
```

5. 验证

``` shell
# 连接任意节点
./redis-cli -c -a xxx -h 192.168.5.100 -p 8001

-c 表示集群
-h IP
-p 端口
-a 密码

# 查看集群信息与节点列表
cluster info（查看集群信息）
cluster nodes（查看节点列表）
```

6. 关闭集群

``` shell
# 需要逐个关闭节点
/mnt/software/redis/src/redis-cli -a xxx -c -h 192.168.0.60 -p 8001 shutdown
```
 
# 监控系统
## Prometheus

节点规划
``` shell
ip
port
x.x.x.x
9090
```

解压缩事先下载好的 Prometheus 二进制包到/user/local/prometheus 目录下  

``` shell
tar -zxvf prometheus-2.14.0.linux-amd64.tar.gz -C /opt/software/
cd /opt/software/
mv prometheus-2.14.0.linux-amd64/ prometheus
```
 
验证版本信息，确认安装包没有问题  
``` shell
cd prometheus/
./prometheus –version
```
 
备份配置文件，后续修改 prometheus.yml 配置文件来收集监控信息  
`cp prometheus.yml prometheus.yml-bak`
 
将 Prometheus 配置为系统服务  
`vim /etc/systemd/system/prometheus.service`

``` shell
Ps: ExecStart 参数为现在服务安装路径 storage.tsdb.path 为数据存储路径 可修改
[Unit]
Description=Prometheus Server
Documentation=https://prometheus.io/docs/introduction/overview/
After=network-online.target
[Service]
User=root
Restart=on-failure
ExecStart=/opt/software/prometheus/prometheus \
 --config.file=/opt/software/prometheus/prometheus.yml \
 --storage.tsdb.path=/opt/software/prometheus/data
[Install]
WantedBy=multi-user.target
```

重新加载 systemd 系统  
``` shell
systemctl daemon-reload 
systemctl start prometheus 
systemctl enable prometheus.service
```

检查服务状态  
`systemctl status prometheus`
 
浏览器中打开 http://x.x.x.x:9090/若能看到内容，安装成功  

### Grafana 安装
节点规划
``` shell
ip
port
x.x.x.x
3000
```
 
执行 rpm 安装事前下载好的 rpm 安装包  
`rpm -Uvh grafana-6.4.4-1.x86_64.rpm`

重新加载 systemd 系统  
``` shell
systemctl daemon-reload 
systemctl start grafana-server.service
systemctl enable grafana-server.service
```
 
检查服务状态  
`systemctl status grafana-server.service`

浏览器中打开 http://xxx.xxx.xxx.xxx:3000/若能看到登录页面，安装成功
 
### Kafka-exporter  ps:适用于未加权限的 kafka
节点规划
``` shell
ip
port
x.x.x.x
9308
```
 
解压缩事先下载好的 kafka-exporter 二进制包到/user/local/kafka-exporter 目录下  
``` shell
tar -zxvf kafka_exporter-1.2.0.linux-amd64.tar.gz -C /opt/software/
cd /opt/software/
mv kafka_exporter-1.2.0.linux-amd64/ kafka-exporter
```
 
编辑 host 文件，适配大数据平台  
`vim/etc/hosts`

若 kafka 集群有域名地址  
添加如下内容  
`x.x.x.x cdh-xxx`
 
把 kafka-exporter 配置为系统服务  
`vim /etc/systemd/system/kafka-exporter.service`

Ps: ExecStart 参数为现在服务安装路径 kafka.server 参数为要监控的 kafka 地址，可多个。  
``` shell
 [Unit]
Description=Kafka exporter
After=network-online.target
[Service]
User=root
Restart=on-failure
ExecStart=/opt/software/kafka-exporter/kafka_exporter \
--kafka.server=cdh-xxx:9092
[Install]
WantedBy=multi-user.target
```

重新加载 systemd 系统  
``` shell
systemctl daemon-reload 
systemctl start kafka-exporter 
systemctl enable kafka-exporter.service
```

检查服务状态  
`systemctl status kafka-exporter`

### Kafka-minion ps:适用于加权限的 kafka
节点规划
``` shell
ip
port
x.x.x.x
9101
```

解压缩事先下载好的 kafka-minion 二进制包到/user/local/kafka-minion 目录下  
``` shell
tar -zxvf kafka_minion-1.0.2.tar.gz -C /root/software/
cd /opt/software/
```
 
编辑 host 文件，适配大数据平台  
`vim/etc/hosts`

若 kafka 集群有域名地址  
添加如下内容  
`x.x.x.x cdh-xxx`

配置 kafka-minion 启动脚本  
`vi run.sh`

``` shell
#!/bin/bash
export KAFKA_BROKERS=dbjf-cmh12:9092
export VERSION=1.0.2
export KAFKA_VERSION=2.0.0
export TELEMETRY_PORT=9101 
 
export KAFKA_SASL_ENABLED=true
export KAFKA_SASL_MECHANISM=GSSAPI
export KAFKA_SASL_GSSAPI_AUTH_TYPE=KEYTAB_AUTH
export KAFKA_SASL_GSSAPI_KEY_TAB_PATH=/data/rec.keytab
export KAFKA_SASL_GSSAPI_KERBEROS_CONFIG_PATH=/etc/krb5.conf
export KAFKA_SASL_GSSAPI_SERVICE_NAME=kafka
export KAFKA_SASL_GSSAPI_USERNAME=rec
export KAFKA_SASL_GSSAPI_REALM=ZXJTKDC
```
`nohup /root/software/kafka-minion/main &`

把 kafka-minion 配置为系统服务  
`vim /etc/systemd/system/kafka-minion.service`

``` shell
Ps: ExecStart 参数为现在服务安装路径
 [Unit]
Description=Kafka minion
After=network-online.target
[Service]
User=root
Restart=on-failure
ExecStart=/mnt/software/kafka-minion/run.sh
[Install]
WantedBy=multi-user.target
```

重新加载 systemd 系统  
``` shell
systemctl daemon-reload 
systemctl start kafka-minion
systemctl enable kafka-minion.service
```
 
检查服务状态  
`systemctl status kafka-minion`
 
### Node-exporter 安装
节点规划
``` shell
ip
port
localhost
9100
```

解压缩事先下载好的 Node-exporter 二进制包到/user/local/node-exporter 目录下  
``` shell
tar -zxvf node_exporter-0.18.1.linux-amd64.tar.gz -C /opt/software
cd /opt/software/
mv node_exporter-0.18.1.linux-arm64/ node-exporter
```
 
把 Node-exporter 配置为系统服务  
`vim /etc/systemd/system/node-exporter.service`

``` shell
Ps: ExecStart 参数为现在服务安装路径
 [Unit]
Description=Node exporter
After=network-online.target
[Service]
User=root
Restart=on-failure
ExecStart=/opt/software/node-exporter/node_exporter 
[Install]
WantedBy=multi-user.target
```
 
重新加载 systemd 系统  
``` shell
systemctl daemon-reload 
systemctl start node-exporter 
systemctl enable node-exporter.service
```
 
检查服务状态  
`systemctl status node-exporter`
 
浏览器中打开 http://{ip-address}:9100/metrics 若能看到相应的指标列表，则安装成功  

【注】：操作系统指标需要被监控的每一台服务器（虚机）上都需要安装.

### Postgres-exporter 安装
节点规划
``` shell
ip
port
localhost
9187
```
 
解压缩事先下载好的 Postgres-exporter 二进制包到/user/local/postgres-exporter 目录下  
``` shell
tar -zxvf postgres_exporter_v0.8.0_linux-amd64.tar.gz -C /opt/software
cd /opt/software
mv postgres_exporter_v0.8.0_linux-amd64/ postgres-exporter
```

把 Postgres-exporter 配置为系统服务  
`vim /etc/systemd/system/postgres_exporter.service`

``` shell
Ps: ExecStart 参数为现在服务安装路径
[Unit]
Description=Prometheus PostgreSQL Exporter
After=network.target
[Service]
Type=simple
Restart=always
User=postgres
Group=postgres
Environment=DATA_SOURCE_NAME="user=postgres host=xxx.xxx.xxx.xxx password='postgres' port=5432 dbname=postgres sslmode=disable"
ExecStart=/opt/software/postgres-exporter/postgres_exporter
[Install]
WantedBy=multi-user.target 
```

重新加载 systemd 系统  
``` shell
systemctl daemon-reload 
systemctl start postgres-exporter
systemctl enable postgres-exporter.service
```
 
检查服务状态  
`systemctl status postgres-exporter.service`

Prometheus 配置  
更新 Prometheus 配置文件  
`vim /opt/software/prometheus/prometheus.yml`

更新配置文件，内容如下：
``` shell
Ps:
Prometheus 配置无需更改
node 配置需要按照实际服务器地址填写 可多个
spring boot 配置需要按照服务所在服务器 ip 端口填写
Postgres 若是安装在同一台机器则不需要改
scrape_configs:
  - job_name: "prometheus"
    static_configs:
      - targets: ["localhost:9090"]
 
  - job_name: "kafka"
    static_configs:
      - targets: ["localhost:9308"]
 
  - job_name: "kafka-minion"
    static_configs:
      - targets: ["localhost:9101"]
 
  - job_name: "node"
    static_configs:
    - targets: ["localhost:9100"]
 
  - job_name: "springboot"
    metrics_path: /management/prometheus
    scrape_interval: 5s
    static_configs:
      - targets:
          - localhost:7020
          - localhost:7030
          - localhost:7100
          - localhost:7110
          - localhost:7120
          - localhost:7200
          - localhost:7350
          - localhost:7410
          - localhost:7430
          - localhost:7500
          - localhost:7510
          - localhost:7590
          - localhost:7600
          - localhost:8080
  
  - job_name: "postgres"
    static_configs:
      - targets: ["localhost:9187"] 
```

Grafana 配置  
首先以默认的的管理员账号登录 admin/admin
 
配置 Prometheus 数据源  

Grafana 菜单栏选择 Data Sources  

点击[Add] 按钮后选择“Prometheus”  

配置 Prometheus 服务器的信息，如下图所示  

【注】localhost 更换为 Prometheus 服务的 IP 地址。
