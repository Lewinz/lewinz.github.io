---
layout: wiki
title: mysql记录
categories: mysql
description: mysql记录
keywords: mysql
---

# 相关操作SQL
``` sql
# 建表
# unsigned 无符号，即非负数
# auto_increment 自增
# primary key关键字用于定义列为主键。 您可以使用多列来定义主键，列间以逗号分隔。

create table if not exists runoob_tbl(
   runoob_id int unsigned auto_increment (primary key),
   runoob_title varchar(100) default '0' not null,
   runoob_author varchar(40) not null,
   submission_date date,
   primary key ( runoob_id,runoob_title )
)engine=innodb default charset=utf8;


# 索引
create index indexname on mytable(username(length));
drop index [indexname] on mytable; 

主键索引
该语句添加一个主键，这意味着索引值必须是唯一的，且不能为null。
alter table tbl_name add primary key (column_list)

唯一索引
这条语句创建索引的值必须是唯一的（除了null外，null可能会出现多次）。
alter table tbl_name add unique index_name (column_list)

普通索引
添加普通索引，索引值可出现多次。
alter table tbl_name add index index_name (column_list)

全文索引
该语句指定了索引为 fulltext ，用于全文索引。
alter table tbl_name add fulltext index_name (column_list)

命令行下显示查看索引
show index from table_name; \g

添加字段
alter table student add age int(4);

修改字段（修改字段名）
alter table 表名 change [column] 旧字段名 新字段名 新数据类型;

修改字段（修改字段类型）
alter table 表名 modify [column] 字段名 新数据类型 新类型长度  新默认值  新注释;
alter table table1 modify  column column1  decimal(10,1) DEFAULT NULL COMMENT '注释';

修改表名
alter table 旧表名 rename to 新表名;

修改注释
alter table 表名 comment '新注释'

在指定位置插入新字段
alter table 表名 add [column] 字段名 字段类型 是否可为空 comment '注释' after 指定某字段;

删除字段
alter table 表名 drop [column] 字段名;
```

# 字段类型

|   类型    |   大小    |   描述    |
|   ----    |   ----    |   ----    |
|char[length]   |   length字节  |   定长字段，长度为0-255个字节 |
|varchar[length] |   string长度+1字节   |   变长字段，64k大小的行采用行[溢出模式](https://blog.csdn.net/weixin_42469000/article/details/113199404)  |
|tinytext   |   string长度+1字节    |   字符串，长度为0-255个字节   |
|text   |   string长度+2字节    |   字符串，最大长度为0-65535个字节 |
|mediumtext |   string长度+3字节    |   字符串，组嗲长度为16777215个字节    |
|longtext   |    string长度+4字节   |    字符串，最大长度为4194967295个字节  |
|tinyint[length]   |    1字节   |    范围:-128~127或0~255    |
|smallint[length]   |    2字节   |       |
|mediumint[length]   |    3字节   |       |
|int[length]   |    4字节   |       |
|bigint[length]   |    8字节   |       |
|float   |    4字节   |       |
|double[length,decimals]   |    8字节   |    运行固定的小数点    |
|decimal[length,decimals]   |    length+1字节或length+2字节   |       |
|date   |    3字节   |    采用YYYY-MM-DD格式  |
|datetime   |    8字节   |    采用YYYY-MM-DD HH:mm:SS格式 |
|timestamp   |    4字节   |    采用YYYYMMDDHHmmSS格式  |
|time   |    3字节   |    采用HH:MM:SS格式    |
|enum   |    1或2字节   |    枚举类型,[详细描述](https://blog.csdn.net/u011442682/article/details/79078199)    |
|set   |    1、2、3、4或8字节   |    与enum一样，只不过每一列可以具有多个可能的值,[详细描述](https://www.cnblogs.com/wtsgtc/p/10387007.html)    |
|blob   |       |    是text的一个变体。允许存储二进制文件，还可用于某些加密数据。    |