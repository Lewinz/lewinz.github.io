---
layout: post
title: 分布式事务解决方案-seate
categories: 分布式
description: 阿里巴巴开源项目-seate
keywords: 分布式, 事务
---


Seata 是一款开源的分布式事务解决方案，致力于提供高性能和简单易用的分布式事务服务。Seata 将为用户提供了 **AT**、**TCC**、**SAGA** 和 **XA** 事务模式，为用户打造一站式的分布式解决方案。  

[**官方文档**](http://seata.io/zh-cn/docs/overview/what-is-seata.html)

- - - 
# AT 模式
**Transaction Coordinator (TC)**： 事务协调器，维护全局事务的运行状态，负责协调并驱动全局事务的提交或回滚。  

**Transaction Manager (TM)**： 控制全局事务的边界，负责开启一个全局事务，并最终发起全局提交或全局回滚的决议。  

**Resource Manager (RM)**： 控制分支事务，负责分支注册、状态汇报，并接收事务协调器的指令，驱动分支（本地）事务的提交和回滚。  

三种角色关系如图：  

![image_TC_TM_RM](https://cdn.jsdelivr.net/gh/Lewinz/lewinz.github.io@master/images/posts/seate_TC_TM_RM.png)

- - -
**使用**  

TC：为分布式服务之外的中间件，需要独立部署。数据源支持 Nacos、MySQL 等多种，常用 Mysql。使用 Mysql 需要新建 seate 表结构，分别为 global_table、branch_table、lock_table 

TM：为全局事务发起、提交、回滚角色，通俗来讲分布式事务业务起点就是 TM。

RM：具体微服务业务系统，每个系统分别管理各自的子事务，各业务系统需要添加同一表结构 undo_log，用于事务回滚。

**两阶段提交协议的演变：**

> 一阶段：业务数据和回滚日志记录在同一个本地事务中提交，释放本地锁和连接资源。
>
> 二阶段：  
>> 提交异步化，非常快速地完成。  
>> 回滚通过一阶段的回滚日志进行反向补偿。

# TCC 模式

>**AT 模式（参考链接 TBD）基于 支持本地 ACID 事务 的 关系型数据库：**  

>> 一阶段 prepare 行为：在本地事务中，一并提交业务数据更新和相应回滚日志记录。  
>> 二阶段 commit 行为：马上成功结束，自动 异步批量清理回滚日志。  
>> 二阶段 rollback 行为：通过回滚日志，自动 生成补偿操作，完成数据回滚。  

>**相应的，TCC 模式，不依赖于底层数据资源的事务支持：** 
>> 一阶段 prepare 行为：调用 自定义 的 prepare 逻辑。  
>> 二阶段 commit 行为：调用 自定义 的 commit 逻辑。  
>> 二阶段 rollback 行为：调用 自定义 的 rollback 逻辑。  
> 所谓 TCC 模式，是指支持把 自定义 的分支事务纳入到全局事务的管理中。  

# SAGA 模式

Saga 模式是 SEATA 提供的长事务解决方案，在 Saga 模式中，业务流程中每个参与者都提交本地事务，当出现某一个参与者失败则补偿前面已经成功的参与者，一阶段正向服务和二阶段补偿服务都由业务开发实现。  

**适用场景：**  
业务流程长、业务流程多  
参与者包含其它公司或遗留系统服务，无法提供 TCC 模式要求的三个接口  

**优势：**  
一阶段提交本地事务，无锁，高性能  
事件驱动架构，参与者可异步执行，高吞吐  
补偿服务易于实现  

**缺点：**  
不保证隔离性  