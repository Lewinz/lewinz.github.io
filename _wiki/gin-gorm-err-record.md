---
layout: wiki
title: gin/gorm 报错记录
categories: [gin,gorm]
description: gin/gorm 报错记录
keywords: gin,gorm
---

## gin

#### gin listen tcp: address 8000: missing port in address
报错原因：gin绑定的端口号格式错误，必须为"ip:port"，一般为":port"，故容易忽略  
解决方案：检查一下端口绑定格式是否正确  

## gorm

#### 映射表名为结构体蛇形复数
```
// 全局禁用表名复数
// 如果设置为true,`User`的默认表名为`user`,使用`TableName`设置的表名不受影响

db.SingularTable(true)
```

#### create自动设置createdAt、updatedAt
gorm在create方法时，会自动设置创建时间与更新时间为now()，源码如下：

``` golang
// updateTimeStampForCreateCallback will set `CreatedAt`, `UpdatedAt` when creating
func updateTimeStampForCreateCallback(scope *Scope) {
    if !scope.HasError() {
        now := scope.db.nowFunc()

        if createdAtField, ok := scope.FieldByName("CreatedAt"); ok {
            if createdAtField.IsBlank {
                createdAtField.Set(now)
            }
        }

        if updatedAtField, ok := scope.FieldByName("UpdatedAt"); ok {
            if updatedAtField.IsBlank {
                updatedAtField.Set(now)
            }
        }
    }
}
```

当发现gorm无法自动设置时间时，检查：  
1、字段名是否正确  
2、结构体字段类型是否为time.Time 或其他时间类型  
3、数据库字段是否正确  
4、结构体实例时是否传值（非空情况下才会自动设置）  

#### 主键设置自增，注意点

1、如果结构体中主键为ID，表字段一定要为id，否则gorm在映射的时候会严格按照蛇形，**主键也不例外**，尽管有主键声明。