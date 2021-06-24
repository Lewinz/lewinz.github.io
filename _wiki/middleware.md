---
layout: wiki
title: 中间件部署文档
categories: 中间件
description: 中间件部署文档
keywords: 中间件, 部署
---


# **JDK**

### rpm方式
上传准备好的离线安装包到服务器 
安装  
`rpm -ivh jdk-8u181-linux-x64.rpm`

### 压缩包安装
上传准备好的压缩包到服务器  

切换到安装目录  
`tar -zvxf /mnt/package/jdk-8u181-linux-x64.tar.gz`  

修改环境变量  
`vim /etc/profile`  

添加以下内容  
```sh
export JAVA_HOME=/mnt/app/jdk
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib:$CLASSPATH
export JAVA_PATH=${JAVA_HOME}/bin:${JRE_HOME}/bin
export PATH=$PATH:${JAVA_PATH}
```

重新生效配置  
`source /etc/profile`
### 检查
输入`java -version`出现信息则说明安装成功

# **Postgresql**   
### 在线安装

1. 安装RPM  
`yum install`

2. 安装客户端(若安装服务器，则不需要)  
`yum install postgresql12`

3. 安装服务端  
`yum install postgresql12-server`
 
### 离线安装

**rpm包**  

准备好如下rpm安装包
>libicu-50.2-4.el7_7.x86_64.rpm  
>libxslt-1.1.28-5.el7.x86_64.rpm  
>postgresql12-libs-12.4-1PGDG.rhel7.x86_64.rpm  
>postgresql12-12.4-1PGDG.rhel7.x86_64.rpm  
>postgresql12-server-12.4-1PGDG.rhel7.x86_64.rpm  
>postgresql12-contrib-12.4-1PGDG.rhel7.x86_64.rpm  

**安装rpm包<按顺序执行>**
```sh
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

设置开机启动pg服务  
`systemctl enable postgresql-12`
 
启动pg服务  
`systemctl start postgresql-12`

修改PostgreSQL远程连接配置  
`vim /var/lib/pgsql/12/data/pg_hba.conf`

```sh
local   all             all                                     peer
host    all             all             127.0.0.1/32            trust
host    all             all             all                     md5
host    replication     all             127.0.0.1/32            trust
host    replication     all             all                     md5
```

修改配置  

`vim /var/lib/pgsql/12/data/postgresql.conf`

```sh
# 新增以下配置
# 放开远程连接ip限制
listen_addresses = '*'
# 配置pg_stat_statements
shared_preload_libraries = 'pg_stat_statements'
track_activity_query_size = 2048
# 配置流复制模式，允许debezium cdc组件实时抓取 
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

修改postgres账号密码  
使用postgres登入数据库  
`psql -h 127.0.0.1 -d postgres -U postgres`

修改postgres密码  
`alter user postgres with password 'postgres';`

加载pg_stat_statements  
`CREATE EXTENSION pg_stat_statements;`

**挂载外部数据盘 ps: mnt为任意挂载盘**  
停止pgsql   
`systemctl stop postgresql-12`


编辑`vim /usr/lib/systemd/system/postgresql-12.service` 

修改文件中PGDATA路径配置为外部数据盘相应文件夹地址  
配置由`Environment=PGDATA=/var/lib/pgsql/12/data/`  
变为`Environment=PGDATA=/mnt/pg/data/`  
 
将文件夹复制到外部数据盘  
`mv /var/lib/pgsql/12/data/ /mnt/pg/data/`

将数据目录文件夹所有者改为postgres用户, postgres用户组  
`chown postgres:postgres /mnt/pg/data/ -R`
 
访问权限改为 750或700  
`chmod 750 -R /mnt/pg/data/`  
执行 `systemctl daemon-reload`  
以后修改配置文件就在/mnt/pg/data下面修改

重启服务  
`systemctl restart postgresql-12`  

### Postgresql主从  

安装PG从库  
按照前面的安装操作，在另一台服务器安装从库pg。    
**！！！从库在部署时不能初始化数据库，否则会导致主从库序列号不一致。**

**配置步骤**  

准备流复制角色 ps: 密码可更改  
在主库中创建进行流复制的角色  
```sh
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

`host replication repuser 从库ip/32 md5`
 
`vim postgresql.conf` 修改以下变量  
```sh
archive_mode = on
archive_command = '/bin/date'
max_wal_senders = 10
wal_keep_segments = 16
synchronous_standby_names = '*'
```
重启主库 使配置生效   

从库配置  
使用以下命令 在两边服务器中检查， 主从服务器连通性  
```sh
    ping ip
    telnet ip 5432
```

确认无误后，在从库服务器执行以下命令，备份主库数据
```sh
cd /mnt/pg/     打开从库pg的数据文件夹处  
pg_basebackup -R -D /mnt/pg/backup -Fp -Xs -v -P -h 主库ip -p 5432 -U repuser  
```

将原数据文件夹备份 再将backup改名为data  
更改data文件夹的用户组，权限 
```sh
chown postgres:postgres data -R
chmod 750 -R data
```

检查 postgresql.auto.conf 文件里是否包含如下内容：
```sh
primary_conninfo = 'user=repuser password=''1qazXSW@3edc'' host=主库IP port=5432 sslmode=prefer sslcompression=0 gssencmode=prefer krbsrvname=postgres target_session_attrs=any'
```

重启从库PG  
检查主从是否成功  

在主库中查询流复制信息  
`select * from pg_stat_replication;`
 
在主库中添加或修改数据，在从库中查看。 

# MySql

### 单节点安装

检查插件  
```sh
# 会与mysql冲突，如果预装了，先卸载
rpm -qa|grep mariadb
rpm -e --nodeps mariadb-xxxxxxxx.x86_64

# mysql安装依赖插件，如果没有，先安装
rpm -qa|grep libaio
yum install libaio
```

```sh
# 安装mysql
rpm -ivh mysql-community-common-5.7.22-1.el7.x86_64.rpm
rpm -ivh mysql-community-libs-5.7.22-1.el7.x86_64.rpm
rpm -ivh mysql-community-client-5.7.22-1.el7.x86_64.rpm
rpm -ivh mysql-community-server-5.7.22-1.el7.x86_64.rpm
```
安装如果报错`perl(Getopt::Long) is needed`  
执行`yum install perl`  

如果报错`warning: mysql-community-server-5.7.22-1.el7.x86_64.rpm: Header V3 DSA/SHA1 Signature, key ID 5072e1f5: NOKEY`  
解决办法：`在rpm命令后加上 --force --nodeps`

初始化数据库，自动创建data文件目录  
`mysqld --initialize-insecure --user=mysql`

将`/var/lib/mysql`文件所有用户修改为mysql
`chown mysql:mysql /var/lib/mysql -R`

修改mysql配置  
`vi /etc/my.cnf`
```sh
# 设置字符编码集 utf8mb4
[client] 
default-character-set = utf8mb4 
 
[mysql] 
default-character-set = utf8mb4 
 
[mysqld] 
character-set-client-handshake = FALSE 
character-set-server = utf8mb4 
collation-server = utf8mb4_general_ci 
init_connect='SET NAMES utf8mb4'

user=mysql
skip-name-resolve
max_connections=500
wait_timeout=31536000
interactive_timeout=31536000
```

启动mysql  
`systemctl start mysqld.service`

登陆mysql修改密码  
```sh
mysql -u root
 
update mysql.user set authentication_string = password('123456') where user = 'root' and host = 'localhost';
 
grant all privileges on *.* to 'root'@'%' identified by '123456' with grant option;
 
flush privileges;
```

退出mysql后，设置开机自启，重启mysql
```sh
systemctl stop mysqld.service
systemctl enable mysqld.service
systemctl list-unit-files | grep mysqld
systemctl start mysqld.service
```

linux上Mysql登录
```sh
# 使用root登录，-p代表有密码
mysql -u root -h 127.0.0.1 -p
```

### 主从模式安装

**主节点修改**  

`vim /etc/my.cnf`

```sh
## 设置server_id，一般设置为IP,注意要唯一
server-id=125107
## 复制过滤：也就是指定哪个数据库不用同步（mysql库一般不同步）
binlog-ignore-db=mysql
binlog-ignore-db=sys
binlog-ignore-db=information_schema
binlog-ignore-db=performance_schem
## 开启二进制日志功能，可以随便取，最好有含义（关键就是这里了）
log-bin=mysql-bin
## 为每个session 分配的内存，在事务过程中用来存储二进制日志的缓存
binlog_cache_size=16M
## 主从复制的格式（mixed,statement,row，默认格式是statement）
binlog_format=mixed
## 二进制日志自动删除/过期的天数。默认值为0，表示不自动删除。
expire_logs_days=30
## 跳过主从复制中遇到的所有错误或指定类型的错误，避免slave端复制中断。
## 如：1062错误是指一些主键重复，1032错误是因为主从数据库数据不一致
slave_skip_errors=1062
## 控制binlog的写入频率。每执行多少次事务写入一次
## 这个参数性能消耗很大，但可减小MySQL崩溃造成的损失，为0表示不控制
sync_binlog = 1
innodb_flush_log_at_trx_commit = 1
```

重启主节点服务  
```sh
systemctl restart mysqld.service
```

创建同步用户  
```sh
# 创建给从节点使用
CREATE USER 'slave'@'%' IDENTIFIED BY 'a123456';

GRANT REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'slave'@'%';
```

**从节点修改**  

`vim /etc/my.cnf`

```sh
## 设置server_id，一般设置为IP,注意要唯一
server-id=125111
## 复制过滤：也就是指定哪个数据库不用同步（mysql库一般不同步）
binlog-ignore-db=mysql
binlog-ignore-db=sys
binlog-ignore-db=information_schema
binlog-ignore-db=performance_schem
## 开启二进制日志功能，以备Slave作为其它Slave的Master时使用
#log-bin=mysql-slave1-bin
## 为每个session 分配的内存，在事务过程中用来存储二进制日志的缓存
binlog_cache_size=16M
## 主从复制的格式（mixed,statement,row，默认格式是statement）
binlog_format=mixed
## 二进制日志自动删除/过期的天数。默认值为0，表示不自动删除。
expire_logs_days=30
## 跳过主从复制中遇到的所有错误或指定类型的错误，避免slave端复制中断。
## 如：1062错误是指一些主键重复，1032错误是因为主从数据库数据不一致
slave_skip_errors=1062
## relay_log配置中继日志
relay_log=mysql-relay-bin
## log_slave_updates表示slave将复制事件写进自己的二进制日志
## 主要为了作为其他的master
#log_slave_updates=ON
## 防止改变数据(除了特殊的线程)
#read_only=1(为了使备机随时转正，所以这里允许写)
## MySQL主从复制的时候，当Master和Slave之间的网络中断，但是Master和Slave无法察觉的情况下（比如防火墙或者路由问题）。Slave会等待slave_net_timeout设置的秒数后，才能认为网络出现故障，然后才会重连并且追赶这段时间主库的数据,默认60
slave-net-timeout = 20                    
## 如果启用，此变量将在服务器启动后立即启用自动中继日志恢复。
relay_log_recovery = ON
## 该变量确定从站在中继日志中的位置是写入FILE还是写入表
relay_log_info_repository = TABLE
```

重启从节点服务  
```sh
systemctl restart mysqld.service
```

登录从节点mysql
```sh
# 命令信息需要修改，在主节点执行命令 show master status\G; 获取
change master to master_host='47.117.136.250', master_user='slave', master_password='a123456', master_port=3306, master_log_file='mysql-bin.000002', master_log_pos=154, master_connect_retry=30,master_heartbeat_period=10;
```

开始主从复制  

`start slave;`

查看主从同步状态  
`show slave status\G;`

Slave_IO_Running/Slave_SQL_Running 两个属性都为YES即为复制成功

若主从复制信息设置错误，可通过下列命令重置信息  
```sh
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
**1593** 在mysql中使用 `show variables like '%server_%';` 查看 server_id/server_uuid 是否配置生效，server_id配置在my.cnf中，server_uuid在auto.cnf中。  

# Nginx

### 安装相关依赖
`yum -y install gcc gcc-c++ pcre pcre-devel zlib zlib-devel openssl openssl-devel gd gd-devel`  

### 安装nginx

解压ngnix安装包 `tar -zxvf nginx-1.17.3.tar.gz`  

进入解压缩目录,编译监控插件  
`./configure --add-module=/mnt/nginx/nginx-module-vts`  

```sh
make
make install
```

启动  
`cd /usr/local/nginx/sbin`  
./nginx  

### 配置
```sh
server {
    listen 8000;
    server_name info;
    client_max_body_size 20M;
    root /data/info/99-admin-web/info-web;
 
    include /etc/nginx/default.d/*.conf;
 
    location /{
        root /data/info/99-admin-web;
        index index.html index.htm;
    }
 
    location /info-api/ {
        proxy_pass http://gateway地址:8080/;
    }
    #添加监控接口/status/：
    location /status {             
        vhost_traffic_status_display;             
        vhost_traffic_status_display_format html;         
    }
}
```

# Nacos

### 单机部署

**通过源码或者发行包获取**  

从 Github 上下载源码方式
```sh
git clone https://github.com/alibaba/nacos.git
cd nacos/
mvn -Prelease-nacos clean install -U  
ls -al distribution/target/
cd distribution/target/nacos-server-$version/nacos/bin
```

下载编译后压缩包方式  

```sh
https://github.com/alibaba/nacos/releases
unzip nacos-server-$version.zip 或者 tar -xvf nacos-server-$version.tar.gz
cd nacos/bin
```

**单机启动**
`sh startup.sh -m standalone`

关闭服务
`sh shutdown.sh`

### 集群部署

安装数据库，版本要求：5.6.5+  

初始化mysql数据库，数据库初始化文件：nacos-mysql.sql  

修改conf/application.properties文件，添加mysql数据源的url、用户名和密码。  

```sh
spring.datasource.platform=mysql
db.num=1
db.url.0=jdbc:mysql://127.0.0.1:3306/nacos?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true
db.user=user_name
db.password=pwd
```

修改conf目录下cluster.conf文件  
```sh
xxx.xxx.xxx.16:8848
xxx.xxx.xxx.17:8848
xxx.xxx.xxx.18:8848
```

配置mysql数据库  

无参模式启动startup.sh脚本  
`sh startup.sh`

# Kafka

### zookeeper单节点安装（可使用kafka自带）

**解压zookeeper压缩包（必须是带"bin"的压缩包，否则启动会报错）**
```sh
cd /mnt/zookeeper/
tar -zxvf apache-zookeeper-3.6.2-bin.tar.gz
```

**修改配置文件**  
```sh
cd /mnt/zookeeper/apache-zookeeper-3.6.2-bin/conf
cp zoo_sample.cfg zoo.cfg
vi zoo.cfg
```

**配置数据路径和日志路径**
```sh
dataDir=/mnt/data/zookeeper
dataLogDir=/mnt/logs/zookeeper

# 自动清理日志<非必须> 3.4之后版本可配置
autopurge.snapRetainCount=3
autopurge.purgeInterval=1
```

**配置环境变量**
```sh
export ZOOKEEPER_INSTALL=/mnt/zookeeper/apache-zookeeper-3.6.2-bin/
export PATH=$PATH:$ZOOKEEPER_INSTALL/bin
```

**启动zookeeper服务**

`/mnt/zookeeper/apache-zookeeper-3.6.2-bin/bin/zkServer.sh start`  

**启动成功**  
```sh
ZooKeeper JMX enabled by default
Using config: /usr/local/zookeeper-3.4.13/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED
```

### zookeeper集群安装

多台服务器zookeeper单节点安装完毕之后

在zoo.cfg配置的data目录下新增myid文件
`	echo 1 > /mnt/software/zookeeper/data/myid`

`vim /mnt/software/zookeeper/conf/zoo.cfg`

```sh
# ip:port:port<port:port为选举leader使用，默认2888:3888，如果是一台服务器部署集群，使用多个不同端口即可>
server.1=xx.xx.xx.xx:2888:3888
server.2=xx.xx.xx.xx:2888:3888
server.3=xx.xx.xx.xx:2888:2888
```

启动服务。

### kafka单节点部署

解压kafka压缩包  
```sh
cd /mnt/tools

tar -zxvf kafka_2.13-2.6.0.tgz
mv kafka_2.13-2.6.0 /mnt/software/kafka

cd /mnt/software/kafka
mkdir zookeeper
mkdir log
mkdir log/zookeeper
mkdir log/kafka
```

修改zookeeper配置  
```sh
vim config/zookeeper.properties
dataDir= /mnt/software/kafka/zookeeper
```

修改kafka配置 
```sh
vim config/server.properties
#修改这几项
log.dirs=/mnt/software/kafka/log/kafka  #日志存放路径
listeners=PLAINTEXT://10.17.64.110:9092 #IP填本机地址 外网/内网可访问地址
zookeeper.connect=10.17.64.110:2181  #IP填zookeeper安装地址
```

先启动zookeeper  
```sh
sh bin/zookeeper-server-start.sh -daemon  config/zookeeper.properties
sh bin/kafka-server-start.sh -daemon config/server.properties
```
 
创建 Topic  
```sh
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
 
启动先zookeeper 停止先kafka  

### kafka集群部署
复制kafka文件夹到其它服务器  

在/mnt/kafka/zookeeper下添加myid文件，写入服务broker.id属性值  
例如 echo 0 > myid  

修改zookeeper配置文件  
`config/zookeeper.properties`

```sh
#注释掉
#maxClientCnxns=0

#设置连接参数，添加如下配置
tickTime=2000　　　　#为zk的基本时间单元，毫秒
initLimit=10　　　　 #Leader-Follower初始通信时限 tickTime*10
syncLimit=5　　　　　#Leader-Follower同步通信时限 tickTime*5
 
#设置broker Id的服务地址 id值与服务器IP对应
server.0=10.17.64.110:2888:3888
server.1=10.17.64.111:2888:3888
server.2=10.17.64.112:2888:3888
```

修改kafka配置文件 config/server.properties  

```sh
broker.id=0  # kafka实例的唯一标识，用整数表示，使用不同的整数值将集群中的kafka实例区分即可

# topic 在当前broker上的分片个数，与broker保持一致
num.partitions=3
zookeeper.connect=node1:2181,node2:2181,node3:2181
 
1. broker.id：同一个集群中，每台机器均不能一样
2. zookeeper.connect：有几台zookeeper服务器，设置为几台台，必须全部加进去
3. listeners：在配置集群的时候，必须设置，不然以后的操作会报找不到leader的错误
```

配置完成后先启动zookeeper 保证服务都启动后在启动kafka  

# ES集群搭建

解压安装包

安装rpm  
`rpm -ivh elasticsearch-7.8.0-x86_64.rpm`

更新es配置  
`vim /etc/elasticsearch/elasticsearch.yml`

```sh
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
///本机ip以及端口
# Set the bind address to a specific IP (IPv4 or IPv6):
#
network.host: 10.0.209.47
network.bind_host: 10.0.209.47
network.publish_host: 10.0.209.47
#
# Set a custom port for HTTP:
#
http.port: 9200
///集群ip配置以及主节点数（计算方式：total number of master-eligible nodes / 2 + 1）
# Pass an initial list of hosts to perform discovery when new node is started:
# The default list of hosts is ["127.0.0.1", "[::1]"]
#
discovery.zen.ping.unicast.hosts: ["10.0.209.47", "10.0.209.44","10.0.209.43"]
#
# Prevent the "split brain" by configuring the majority of nodes (total number of master-eligible nodes / 2 + 1):
#
discovery.zen.minimum_master_nodes: 2
```

设置jvm参数  
`vim etc/elasticsearch/jvm.options  （内存允许 设置31g）`

```sh
-Xms31g
-Xmx31g
```

配置密码 （公网环境需要设置，客户服务器视情况而定  内网访问可不需要密码）  

执行以下命令输入ElasticSearch的密码

`/usr/share/elasticsearch/bin/elasticsearch-setup-passwords interactive`

配置开机启动
```sh
systemctl enable elasticsearch.service
systemctl start elasticsearch
systemctl status elasticsearch
```

# Redis

### 单机搭建

解压
```sh
tar –zxvf redis-5.0.5.tar.gz

cd /mnt/software/redis/

<sudo> make MALLOC=libc && make install
```

如果在此步报错  
cc: 错误：../deps/hiredis/libhiredis.a：没有那个文件或目录  
cc: 错误：../deps/lua/src/liblua.a：没有那个文件或目录  

切换至deps/执行  

`<sudo> make lua hiredis linenoise`  

再执行`<sudo> make MALLOC=libc && make install`即可  

```sh
cd src
./redis-server
```

### Redis配置成服务

修改配置文件redis.conf  
`vim /mnt/software/redis/redis.conf`

```sh
# 三个参数都是修改
bind 0.0.0.0
logfile /mnt/software/redis/logs/redis.log
dir /mnt/software/redis/data
```

将配置文件拷贝到/etc/init.d目录下  

`cp /mnt/software/redis/redis.conf /etc/redis/redis.conf`

将redis启动脚本复制到/etc/init.d目录下  

`cp /mnt/software/redis/utils/redis_init_script /etc/init.d/redis`

修改启动脚本  

`vim /etc/init.d/redis`

```sh
# 修改参数，文件名为刚刚复制到此目录的 conf 文件
CONF="/etc/redis/6379.conf"
```

redis文件添加到 systemctl

`systemctl enable redis`

重启Redis  
```sh
<sudo> systemctl stop redis
<sudo> systemctl start redis
```

### Redis配置主从复制  

1）各结点启动redis 服务  

`sudo systemctl start redis`  

2）登陆从结点，查看主从复制情况  

`/data/redis/redis-5.0.5/src/redis-cli info Replication`

3）设置开机自启  

`$ sudo /sbin/chkconfig redis on`  

从redis149上配置文件6379.conf添加  
```sh
slaveof 10.16.212.224 6379
slave-read-only yes
```

4）验证  

执行redis-cli 连接到redis-service  

执行info Replication命令查看状态  

看到从节点上有  

master_host: 10.16.212.224以及slave_read_only:1字样即为配置成功  

### 集群部署    

暂时先参考此文档：https://www.jianshu.com/p/8045b92fafb2
 
# 监控系统
### Prometheus

节点规划
```sh
ip
port
x.x.x.x
9090
```

解压缩事先下载好的Prometheus二进制包到/user/local/prometheus目录下  

```sh
tar -zxvf prometheus-2.14.0.linux-amd64.tar.gz -C /opt/software/
cd /opt/software/
mv prometheus-2.14.0.linux-amd64/ prometheus
```
 
验证版本信息，确认安装包没有问题  
```sh
cd prometheus/
./prometheus –version
```
 
备份配置文件，后续修改prometheus.yml配置文件来收集监控信息  
`cp prometheus.yml prometheus.yml-bak`
 
将Prometheus配置为系统服务  
`vim /etc/systemd/system/prometheus.service`

```sh
Ps: ExecStart参数为现在服务安装路径 storage.tsdb.path为数据存储路径 可修改
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

重新加载systemd系统  
```sh
systemctl daemon-reload 
systemctl start prometheus 
systemctl enable prometheus.service
```

检查服务状态  
`systemctl status prometheus`
 
浏览器中打开 http://x.x.x.x:9090/若能看到内容，安装成功  

### Grafana安装
节点规划
```sh
ip
port
x.x.x.x
3000
```
 
执行rpm安装事前下载好的rpm安装包  
`rpm -Uvh grafana-6.4.4-1.x86_64.rpm`

重新加载systemd系统  
```sh
systemctl daemon-reload 
systemctl start grafana-server.service
systemctl enable grafana-server.service
```
 
检查服务状态  
`systemctl status grafana-server.service`

浏览器中打开 http://xxx.xxx.xxx.xxx:3000/若能看到登录页面，安装成功
 
### Kafka-exporter  ps:适用于未加权限的kafka
节点规划
```sh
ip
port
x.x.x.x
9308
```
 
解压缩事先下载好的kafka-exporter二进制包到/user/local/kafka-exporter目录下  
```sh
tar -zxvf kafka_exporter-1.2.0.linux-amd64.tar.gz -C /opt/software/
cd /opt/software/
mv kafka_exporter-1.2.0.linux-amd64/ kafka-exporter
```
 
编辑host文件，适配大数据平台  
`vim/etc/hosts`

若kafka集群有域名地址  
添加如下内容  
`x.x.x.x cdh-xxx`
 
把kafka-exporter配置为系统服务  
`vim /etc/systemd/system/kafka-exporter.service`

Ps: ExecStart参数为现在服务安装路径 kafka.server参数为要监控的kafka地址，可多个。  
```sh
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

重新加载systemd系统  
```sh
systemctl daemon-reload 
systemctl start kafka-exporter 
systemctl enable kafka-exporter.service
```

检查服务状态  
`systemctl status kafka-exporter`

### Kafka-minion ps:适用于加权限的kafka
节点规划
```sh
ip
port
x.x.x.x
9101
```

解压缩事先下载好的kafka-minion二进制包到/user/local/kafka-minion目录下  
```sh
tar -zxvf kafka_minion-1.0.2.tar.gz -C /root/software/
cd /opt/software/
```
 
编辑host文件，适配大数据平台  
`vim/etc/hosts`

若kafka集群有域名地址  
添加如下内容  
`x.x.x.x cdh-xxx`

配置kafka-minion启动脚本  
`vi run.sh`

```sh
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

把kafka-minion配置为系统服务  
`vim /etc/systemd/system/kafka-minion.service`

```sh
Ps: ExecStart参数为现在服务安装路径
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

重新加载systemd系统  
```sh
systemctl daemon-reload 
systemctl start kafka-minion
systemctl enable kafka-minion.service
```
 
检查服务状态  
`systemctl status kafka-minion`
 
### Node-exporter安装
节点规划
```sh
ip
port
localhost
9100
```

解压缩事先下载好的Node-exporter二进制包到/user/local/node-exporter目录下  
```sh
tar -zxvf node_exporter-0.18.1.linux-amd64.tar.gz -C /opt/software
cd /opt/software/
mv node_exporter-0.18.1.linux-arm64/ node-exporter
```
 
把Node-exporter配置为系统服务  
`vim /etc/systemd/system/node-exporter.service`

```sh
Ps: ExecStart参数为现在服务安装路径
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
 
重新加载systemd系统  
```sh
systemctl daemon-reload 
systemctl start node-exporter 
systemctl enable node-exporter.service
```
 
检查服务状态  
`systemctl status node-exporter`
 
浏览器中打开 http://{ip-address}:9100/metrics若能看到相应的指标列表，则安装成功  

【注】：操作系统指标需要被监控的每一台服务器（虚机）上都需要安装.

### Postgres-exporter安装
节点规划
```sh
ip
port
localhost
9187
```
 
解压缩事先下载好的Postgres-exporter二进制包到/user/local/postgres-exporter目录下  
```sh
tar -zxvf postgres_exporter_v0.8.0_linux-amd64.tar.gz -C /opt/software
cd /opt/software
mv postgres_exporter_v0.8.0_linux-amd64/ postgres-exporter
```

把Postgres-exporter配置为系统服务  
`vim /etc/systemd/system/postgres_exporter.service`

```sh
Ps: ExecStart参数为现在服务安装路径
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

重新加载systemd系统  
```sh
systemctl daemon-reload 
systemctl start postgres-exporter
systemctl enable postgres-exporter.service
```
 
检查服务状态  
`systemctl status postgres-exporter.service`

Prometheus配置  
更新Prometheus配置文件  
`vim /opt/software/prometheus/prometheus.yml`

更新配置文件，内容如下：
```sh
Ps:
Prometheus 配置无需更改
node 配置需要按照实际服务器地址填写 可多个
spring boot配置需要按照服务所在服务器ip 端口填写
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

Grafana配置  
首先以默认的的管理员账号登录admin/admin
 
配置Prometheus数据源  

Grafana菜单栏选择Data Sources  

点击[Add]按钮后选择“Prometheus”  

配置Prometheus服务器的信息，如下图所示  

【注】localhost更换为Prometheus服务的IP地址。
