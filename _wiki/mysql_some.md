---
layout: wiki
title: mysql 记录
categories: mysql
description: mysql 记录
keywords: mysql
---

# 相关操作 SQL
``` sql
# 建表
# unsigned 无符号，即非负数
# auto_increment 自增
# primary key 关键字用于定义列为主键。 您可以使用多列来定义主键，列间以逗号分隔。

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
该语句添加一个主键，这意味着索引值必须是唯一的，且不能为 null。
alter table tbl_name add primary key (column_list)

唯一索引
这条语句创建索引的值必须是唯一的（除了 null 外，null 可能会出现多次）。
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
alter table table1 modify  column column1  decimal(10,1) DEFAULT NULL COMMENT ' 注释 ';

修改表名
alter table 旧表名 rename to 新表名;

修改注释
alter table 表名 comment ' 新注释 '

在指定位置插入新字段
alter table 表名 add [column] 字段名 字段类型 是否可为空 comment ' 注释 ' after 指定某字段;

删除字段
alter table 表名 drop [column] 字段名;
```

# 字段类型

|   类型    |   大小    |   描述    |
|   ----    |   ----    |   ----    |
|char[length]   |   length 字节  |   定长字段，长度为 0-255 个字节 |
|varchar[length] |   string 长度+1 字节   |   变长字段，64k 大小的行采用行[溢出模式](https://blog.csdn.net/weixin_42469000/article/details/113199404)  |
|tinytext   |   string 长度+1 字节    |   字符串，长度为 0-255 个字节   |
|text   |   string 长度+2 字节    |   字符串，最大长度为 0-65535 个字节 |
|mediumtext |   string 长度+3 字节    |   字符串，组嗲长度为 16777215 个字节    |
|longtext   |    string 长度+4 字节   |    字符串，最大长度为 4194967295 个字节  |
|tinyint[length]   |    1 字节   |    范围:-128~127 或 0~255    |
|smallint[length]   |    2 字节   |       |
|mediumint[length]   |    3 字节   |       |
|int[length]   |    4 字节   |       |
|bigint[length]   |    8 字节   |       |
|float   |    4 字节   |       |
|double[length,decimals]   |    8 字节   |    运行固定的小数点    |
|decimal[length,decimals]   |    length+1 字节或 length+2 字节   |       |
|date   |    3 字节   |    采用 YYYY-MM-DD 格式  |
|datetime   |    8 字节   |    采用 YYYY-MM-DD HH:mm:SS 格式 |
|timestamp   |    4 字节   |    采用 YYYYMMDDHHmmSS 格式  |
|time   |    3 字节   |    采用 HH:MM:SS 格式    |
|enum   |    1 或 2 字节   |    枚举类型,[详细描述](https://blog.csdn.net/u011442682/article/details/79078199)    |
|set   |    1、2、3、4 或 8 字节   |    与 enum 一样，只不过每一列可以具有多个可能的值,[详细描述](https://www.cnblogs.com/wtsgtc/p/10387007.html)    |
|blob   |       |    是 text 的一个变体。允许存储二进制文件，还可用于某些加密数据。    |