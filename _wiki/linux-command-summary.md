---
layout: wiki
title: Linux 命令备忘
categories: Linux
description: Linux 命令备忘
keywords: Linux, 命令， 备忘
---

### 常用
切换 shell  
```
# 查看所有终端类型
cat /etc/shells

# 切换
chsh -s /bin/sh
```

设置用户密码  
```
passwd 用户名
```


### 文件/文件夹
```
    mkdir 新建文件夹  
    rmdir 删除文件夹 
    vim --> i --> esc --> :wq!  修改文件并保存 
    路 pwd 查看文件夹具体径
```

### tar
```
-c 创建新的文档。
-v 显示详细的 tar 处理的文件信息
-f 要操作的文件名
-z 调用 gunzip 操作
-x 解压文件
-r 表示增加文件，把要增加的文件追加在压缩文件的末尾
-t 表示查看文件，查看文件中的文件内容

常用：
    tar -zvxf 解压.gz 文件
```

### rpm
```
rpm -ivh (--badreloc 强制指定安装目录)(--relocate 指定安装目录) 安装  
rpm -Uvh 升级  
rpm -e 反安装  
rpm -qpi 查询安装包详细信息  
rpm -qf 查询文件属于哪个安装包  
rpm -qpl 查询软件会向系统写入哪些文件  
rpm -qa | grep XXX(moudle name) 查看安装包是否被安装  
```

### 查看 java 进程
```
jps
    -q：只输出进程 ID
    -m：输出传入 main 方法的参数
    -l：输出完全的包名，应用主类名，jar 的完全路径名
    -v：输出 jvm 参数
    -V：输出通过 flag 文件传递到 JVM 中的参数
```

### whereis 查找文件
```
whereis
    -b 　只查找二进制文件。
    -B<目录> 　只在设置的目录下查找二进制文件。
    -f 　不显示文件名前的路径名称。
    -m 　只查找说明文件。
    -M<目录> 　只在设置的目录下查找说明文件。
    -s 　只查找原始代码文件。
    -S<目录> 　只在设置的目录下查找原始代码文件。
    -u 　查找不包含指定类型的文件。
```

### mysql 命令
启动  
```
service mysql start
systemctl start mysqld.service
```

关闭  
```
service mysql stop
systemctl stop mysqld.service
```

重启  
```
service mysql restart
systemctl restart mysqld.service
```

登录
`mysql -u 用户名 -p 用户密码`

切换数据库
`use mysql;`

导入 sql 文件在 mysql 执行
`mysql -u root -p dbname < filename.sql`

其他命令：<https://www.cnblogs.com/roadone/p/9834997.html>

### postgresql 命令

#### 登入数据库
```
# -h 连接 IP -d 用户名 -U 数据库名
psql -h 127.0.0.1 -d postgres -U postgres
```

#### 通过命令行  
```
# 重新设置用户（dif）密码
\password dlf

# 导入 sql 在数据库中（exampledb）执行    ps：数据库外执行
psql exampledb < user.sql

# 导入 sql 在当前数据库中执行
\i aaa.sql

# 列出数据库名
\l

# 切换数据库
\c 数据库名

# 列出表名
\d 数据库

# 查看所有表
\dt

# 得到表结构
\d 表名

# 显示字符集
\encoding

# 显示所有用户
\du

# 查看所有存储过程
\df （\df+ name）


```

#### 通过 SQL 语句  
```
# 列出数据库名
select datname from pg_database;

# 列出表名
select tablename from pg_tables where tablename not like 'pg%' and tablename not like 'sql_%' order by tablename;

# 得到当前 db 中所有表的信息（这里 pg_tables 是系统视图）
select * from pg_tables;

# 得到所有用户自定义表的名字
# 这里"tablename"字段是表的名字
# "schemaname"是 schema 的名字
# 用户自定义的表，如果未经特殊处理，默认都是放在名为 public 的 schema 下
select tablename from pg_tables where schemaname='public';

# 创建 schema
create schema myschema;

# 创建用户
create user $username with password '*****';

# 将数据库所有权限授权给用户
grant all privileges on database $databasename to $username;

# 将 schema 所有权限授权给用户
grant all privileges on schema $schemaname to $username;

# 授权用户 schemaname 下所有表查询权限
grant select on all tables in schema $schemaname to $username;

# 将$schemaname 以后新建的表默认授权 select 给$username
alter default privileges in schema $schemaname grant select on tables to $username;

# 通用 sql
*创建数据库：
create database [数据库名 ];

*删除数据库：
drop database [数据库名 ]; 

*创建表：
create table ([字段名 1] [类型 1] ;,[字段名 2] [类型 2],......<,primary key (字段名 m,字段名 n,...)>;);

*在表中插入数据：
insert into 表名 ([字段名 m],[字段名 n],......) values ([列 m 的值 ],[列 n 的值 ],......);

*重命名一个表：
alter table [表名 A] rename to [表名 B];

*在已有的表里添加字段：
alter table [表名 ] add column [字段名 ] [类型 ];

*删除表中的字段：
alter table [表名 ] drop column [字段名 ];

*重命名一个字段： 
alter table [表名 ] rename column [字段名 A] to [字段名 B];

*给一个字段设置缺省值： 
alter table [表名 ] alter column [字段名 ] set default [新的默认值 ];

*去除缺省值： 
alter table [表名 ] alter column [字段名 ] drop default;

*可以使用 pg_dump 和 pg_dumpall 来完成。比如备份 sales 数据库：
pg_dump drupal>/opt/Postgresql/backup/1.bak

```

### 远程拷贝 

#### 复制文件  

将本地文件拷贝到远程  
```
# scp 文件名 用户名@计算机 IP 或者计算机名称:远程路径
# 本地 192.168.1.8 客户端

scp /root/install.* root@192.168.1.12:/usr/local/src
```

从远程将文件拷回本地  
```
# scp 用户名@计算机 IP 或者计算机名称:文件名  本地路径
# 本地 192.168.1.8 客户端取远程服务器 12、11 上的文件

scp root@192.168.1.12:/usr/local/src/*.log /root/
```
#### 复制文件夹  

将本地文件夹拷贝到远程  
```
# scp -r 目录名 用户名@计算机 IP 或者计算机名称:远程路径
# test1 为源目录，test2 为目标目录，zhidao@192.168.0.1 为远程服务器的用户名和 ip 地址

scp -r /home/test1 zhidao@192.168.0.1:/home/test2
```

从远程将文件夹拷回本地  
```
# scp -r 用户名@计算机 IP 或者计算机名称:目录名 本地路径
# zhidao@192.168.0.1 为远程服务器的用户名和 ip 地址，test1 为源目录，test2 为目标目录

scp  -r zhidao@192.168.0.1:/home/test2 /home/test1
```

#### 防火墙操作

查看防火墙状态  
```
systemctl status firewalld

active(running)：开启状态，正在运行中

inactive(dead)：关闭状态，未在运行
```

开启防火墙  

`systemctl start firewalld`没有任何提示，表示开启成功

关闭防火墙

`systemctl stop firewalld`

添加端口
```
# 添加端口 8089（--permanent 表示永久生效，没有此参数重启后失效）
firewall-cmd --zone=public --add-port=8089/udp --permanent
# 这个命令必须运行，才能加载成功
firewall-cmd --reload
```

查看端口开启情况  

`firewall-cmd --zone=public --query-port=8089/udp`

删除防火墙 8086 端口
```
firewall-cmd --zone=public --remove-port=8086/tcp --permanent

firewall-cmd --reload
```

### tree 命令
``` shell
brew install tree

tree 
-a 显示所有文件和目录。
-A 使用 ASNI 绘图字符显示树状图而非以 ASCII 字符组合。
-C 在文件和目录清单加上色彩，便于区分各种类型。
-d 显示目录名称而非内容。
-D 列出文件或目录的更改时间。
-f 在每个文件或目录之前，显示完整的相对路径名称。
-F 在执行文件，目录，Socket，符号连接，管道名称名称，各自加上”*”,”/”,”=”,”@”,”|”号。
-g 列出文件或目录的所属群组名称，没有对应的名称时，则显示群组识别码。
-i 不以阶梯状列出文件或目录名称。
-I<范本样式> 不显示符合范本样式的文件或目录名称。
-l 如遇到性质为符号连接的目录，直接列出该连接所指向的原始目录。
-n 不在文件和目录清单加上色彩。
-N 直接列出文件和目录名称，包括控制字符。
-p 列出权限标示。
-P <范本样式> 只显示符合范本样式的文件或目录名称。
-q 用”?”号取代控制字符，列出文件和目录名称。
-s 列出文件或目录大小。
-t 用文件和目录的更改时间排序。
-u 列出文件或目录的拥有者名称，没有对应的名称时，则显示用户识别码。
-x 将范围局限在现行的文件系统中，若指定目录下的某些子目录，其存放于另一个文件系统上，则将该子目录予以排除在寻找范围外。
```