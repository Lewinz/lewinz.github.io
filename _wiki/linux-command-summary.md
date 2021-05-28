---
layout: wiki
title: Linux命令备忘
categories: Linux
description: Linux命令备忘
keywords: Linux, 命令， 备忘
---

### 常用
切换shell  
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
    路pwd 查看文件夹具体径
```

### tar
```
-c 创建新的文档。
-v 显示详细的tar处理的文件信息
-f 要操作的文件名
-z 调用gunzip操作
-x 解压文件
-r 表示增加文件，把要增加的文件追加在压缩文件的末尾
-t 表示查看文件，查看文件中的文件内容

常用：
    tar -zvxf 解压.gz文件
```

### rpm
```
rpm -ivh (--badreloc 强制指定安装目录)(--relocate 指定安装目录)安装  
rpm -Uvh 升级  
rpm -e 反安装  
rpm -qpi 查询安装包详细信息  
rpm -qf 查询文件属于哪个安装包  
rpm -qpl 查询软件会向系统写入哪些文件  
rpm -qa | grep XXX(moudle name) 查看安装包是否被安装  
```

### 查看java进程
```
jps
    -q：只输出进程 ID
    -m：输出传入 main 方法的参数
    -l：输出完全的包名，应用主类名，jar的完全路径名
    -v：输出jvm参数
    -V：输出通过flag文件传递到JVM中的参数
```

### whereis查找文件
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

### mysql命令
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
`mysql -u用户名 -p用户密码`

切换数据库
`use mysql;`

导入sql文件在mysql执行
`mysql -u root -p dbname < filename.sql`

其他命令：<https://www.cnblogs.com/roadone/p/9834997.html>

### postgresql命令

#### 登入数据库
```
# -h 连接IP -d 用户名 -U 数据库名
psql -h 127.0.0.1 -d postgres -U postgres
```

#### 通过命令行  
```
# 重新设置用户（dif）密码
\password dlf

# 导入sql在数据库中（exampledb）执行    ps：数据库外执行
psql exampledb < user.sql

# 导入sql在当前数据库中执行
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

#### 通过SQL语句  
```
# 列出数据库名
select datname from pg_database;

# 列出表名
select tablename from pg_tables where tablename not like 'pg%' and tablename not like 'sql_%' order by tablename;

# 得到当前db中所有表的信息（这里pg_tables是系统视图）
select * from pg_tables;

# 得到所有用户自定义表的名字
# 这里"tablename"字段是表的名字
# "schemaname"是schema的名字
# 用户自定义的表，如果未经特殊处理，默认都是放在名为public的schema下
select tablename from pg_tables where schemaname='public';

# 创建schema
create schema myschema;

# 创建用户
create user $username with password '*****';

# 将数据库所有权限授权给用户
grant all privileges on database $databasename to $username;

# 将schema所有权限授权给用户
grant all privileges on schema $schemaname to $username;

# 授权用户schemaname下所有表查询权限
grant select on all tables in schema $schemaname to $username;

# 将$schemaname以后新建的表默认授权select给$username
alter default privileges in schema $schemaname grant select on tables to $username;

# 通用sql
*创建数据库：
create database [数据库名];

*删除数据库：
drop database [数据库名]; 

*创建表：
create table ([字段名1] [类型1] ;,[字段名2] [类型2],......<,primary key (字段名m,字段名n,...)>;);

*在表中插入数据：
insert into 表名 ([字段名m],[字段名n],......) values ([列m的值],[列n的值],......);

*重命名一个表：
alter table [表名A] rename to [表名B];

*在已有的表里添加字段：
alter table [表名] add column [字段名] [类型];

*删除表中的字段：
alter table [表名] drop column [字段名];

*重命名一个字段： 
alter table [表名] rename column [字段名A] to [字段名B];

*给一个字段设置缺省值： 
alter table [表名] alter column [字段名] set default [新的默认值];

*去除缺省值： 
alter table [表名] alter column [字段名] drop default;

*可以使用pg_dump和pg_dumpall来完成。比如备份sales数据库：
pg_dump drupal>/opt/Postgresql/backup/1.bak

```

### 远程拷贝 

#### 复制文件  

将本地文件拷贝到远程  
```
# scp 文件名 用户名@计算机IP或者计算机名称:远程路径
# 本地192.168.1.8客户端

scp /root/install.* root@192.168.1.12:/usr/local/src
```

从远程将文件拷回本地  
```
# scp 用户名@计算机IP或者计算机名称:文件名  本地路径
# 本地192.168.1.8客户端取远程服务器12、11上的文件

scp root@192.168.1.12:/usr/local/src/*.log /root/
```
#### 复制文件夹  

将本地文件夹拷贝到远程  
```
# scp -r 目录名 用户名@计算机IP或者计算机名称:远程路径
# test1为源目录，test2为目标目录，zhidao@192.168.0.1为远程服务器的用户名和ip地址

scp -r /home/test1 zhidao@192.168.0.1:/home/test2
```

从远程将文件夹拷回本地  
```
# scp -r 用户名@计算机IP或者计算机名称:目录名 本地路径
# zhidao@192.168.0.1为远程服务器的用户名和ip地址，test1为源目录，test2为目标目录

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
# 添加端口8089（--permanent表示永久生效，没有此参数重启后失效）
firewall-cmd --zone=public --add-port=8089/udp --permanent
# 这个命令必须运行，才能加载成功
firewall-cmd --reload
```

查看端口开启情况  

`firewall-cmd --zone=public --query-port=8089/udp`

删除防火墙8086端口
```
firewall-cmd --zone=public --remove-port=8086/tcp --permanent

firewall-cmd --reload
```