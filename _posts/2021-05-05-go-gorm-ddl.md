---
layout: post
title: gorm自动创建database
categories: [gorm]
description: gorm自动创建database
keywords: gorm
---

## gorm自动创建database

### tag约定

``` golang
示例：
type Post struct {
    PostId    int    `gorm:"primary_key;auto_increment"`
    Uid       int    `gorm:"type:int;not null"`
    Title     string `gorm:"type:varchar(255);not null"`
    Content   string `gorm:"type:text;not null"`
    Type      uint8  `gorm:"type:tinyint;default 1;not null"`
    CreatedAt time.Time
    UpdatedAt time.Time
    DeletedAt time.Time
}

column	指定列名
type	指定列数据类型
size	指定列大小, 默认值255
primary_key	将列指定为主键
unique	将列指定为唯一
default	指定列默认值
precision   指定列精度
not null    将列指定为非 null
auto_increment  指定列是否为自增类型
index   创建具有或不带名称的索引, 如果多个索引同名则创建复合索引
unique_index    和 index 类似，只不过创建的是唯一索引
embedded    将结构设置为嵌入
embedded_prefix 设置嵌入结构的前缀
-   忽略此字段
```
#### 主键
grom的约定中，一般将数据模型中的id字段映射为数据表的主键，如下面定义的testmodel,id为主键，testmodel的id的数据类型为string，如果id的数据类型为int，则grom还会为该设置auto_increment，使用id成为自增主键。  

示例：
```golang
type TestModel struct{
    ID   int
    Name string
}
```

当然，我们也可以自定义主键字段的名称，示例的Post结构体，我们设置了PostId字段为主键，如果我们定义了其他字段为主键，那么，就算结构体中仍有ID字段，GROM框架也不会把ID字段当作主键了。

``` golang
type Post struct {
    ID        int
    PostId    int    `gorm:"primary_key;auto_increment"`
    Uid       int    `gorm:"type:int;not null"`
    Title     string `gorm:"type:varchar(255);not null"`
    Content   string `gorm:"type:text;not null"`
    Type      uint8  `gorm:"type:tinyint;default 1;not null"`
    CreatedAt time.Time
    UpdatedAt time.Time
    DeletedAt time.Time
}
```

#### 数据表映射规则

当我们使用结构体创建数据表时，数据表的名称默认为结构体的小写复数形式，如结构体Post对应的数据表名称为posts,当然我们也可以自己指定结构体对应的数据表名称，而不是用默认的。

为结构体加上TableName()方法，通过这个方法可以返回自定义的数据表名，如下：

```golang
//指定Post结构体对应的数据表为my_posts
func (p Post) TableName() string{
    return "my_posts"
}
```

#### 数据表前缀（全局）

除了指定数据表名外，我们也可以重写gorm.DefaultTableNameHandler这个变量，这样可以为所有数据表指定统一的数据表前缀，如下：

```golang
gorm.DefaultTableNameHandler = func (db *gorm.DB, defaultTableName string) string  {
    return "tb_" + defaultTableName;
}
```

#### 字段映射规则

结构体到数据表的名称映射规则为结构体名称的复数，而结构体的字段到数据表字段的默认映射规则是用下划线分隔每个大写字母开头的单词，如下：

```golang
type Prize struct {
	ID int
	PrizeName string
}
```

上面的结构体Prize中的PrizeName字段对应的数据表为prize_name，但我们把PrizeName改为Prizename时，则对应的数据表字段名称为prizename,这是为因为只分隔大写字段开头的单词。

当然，我们也可以为结构体的某个字段定义tags扩展信息,这样结构体字段到数据表字段的映规则就在tags中定义。

#### 时间点追踪

前面我们说过，如果结构体中有名称为ID字段，则GORM框架会把该字段作为数据表的主键，除此之外，如果结构体中有CreatedAt,UpdatedAt,DeletedAt这几个字段的话，则GROM框架也会作一些特殊处理，规则如下：

```
CreatedAt：新增数据表记录的时候，会自动写入这个字段。
UpdatedAt：更新数据表记录的时候，会自动更新这个字段。
DeletedAt：当执行软删除的时候，会自动更新这个字段，表示删除时间
```

#### gorm.Model

由于如果结构体中有ID,CreatedAt,UpdatedAt,DeletedAt这几个比较通用的字段，GORM框架会自动处理这几个字段，所以如果我们结构体需要这几个字段时，我们可以直接在自定义结构体中嵌入gorm.Model结构体，gorm.Model的结构体如下：

```golang
type Model struct {
    ID        uint `gorm:"primary_key"`
    CreatedAt time.Time
    UpdatedAt time.Time
    DeletedAt *time.Time `sql:"index"`
}
```

所以，如果我们在结构体Prize中嵌入gorm.Model,如下：
``` golang
type Prize struct{
    gorm.Model
    Name string
}
```
这样就等同于：
```golang
type Prize struct {
	ID        uint `gorm:"primary_key"`
	Name      string
	CreatedAt time.Time
	UpdatedAt time.Time
	DeletedAt *time.Time `sql:"index"`
}
```

### 数据库迁移

我们这里所说的数据库迁移，即通过使用GROM提供的一系列方法，根据数据模型定义好的规则，进行创建、删除数据表等操作，也就是数据库的DDL操作。

GORM提供对数据库进行DDL操作的方法，主要以下几类：

#### 数据表操作

```golang
//根据模型自动创建数据表
func (s *DB) AutoMigrate(values ...interface{}) *DB 
 
//根据模型创建数据表
func (s *DB) CreateTable(models ...interface{}) *DB
 
//删除数据表，相当于drop table语句
func (s *DB) DropTable(values ...interface{}) *DB 
 
//相当于drop table if exsist 语句
func (s *DB) DropTableIfExists(values ...interface{}) *DB 
 
//根据模型判断数据表是否存在
func (s *DB) HasTable(value interface{}) bool
```

#### 列操作

```golang
//删除数据表字段
func (s *DB) DropColumn(column string) *DB
 
//修改数据表字段的数据类型
func (s *DB) ModifyColumn(column string, typ string) *DB
```

#### 索引操作

```golang
//添加外键
func (s *DB) AddForeignKey(field string, dest string, onDelete string, onUpdate string) *DB
 
//给数据表字段添加索引
func (s *DB) AddIndex(indexName string, columns ...string) *DB
 
//给数据表字段添加唯一索引
func (s *DB) AddUniqueIndex(indexName string, columns ...string) *DB
```

### 数据迁移简单代码示例

```golang
type User struct {
    Id       int   //对应数据表的自增id
    Username string
    Password string
    Email    string
    Phone    string
}
 
func main(){
    db.AutoMigrate(&Post{},&User{})//创建posts和users数据表
 
    db.CreateTable(&Post{})//创建posts数据表
    
    db.Set("gorm:table_options", "ENGINE=InnoDB").CreateTable(&Post{})//创建posts表时指存在引擎
    
    db.DropTable(&Post{},"users")//删除posts和users表数据表
    
    db.DropTableIfExists(&Post{},"users")//删除前会判断posts和users表是否存在
    
    //先判断users表是否存在，再删除users表
    if db.HasTable("users") {
        db.DropTable("users")
    }
    
    //删除数据表字段
    db.Model(&Post{}).DropColumn("id")
    
    //修改字段数据类型
    db.Model(&Post{}).ModifyColumn("id","varchar(255)")
    
    //建立posts与users表之间的外键关联
    db.Model(&Post{}).AddForeignKey("uid", "users(id)", "RESTRICT", "RESTRICT")
    
    //给posts表的title字段添加索引
    db.Model(&Post{}).AddIndex("index_title","title")
    
    //给users表的phone字段添加唯一索引
    db.Model(&User{}).AddUniqueIndex("index_phone","phone")
}
```