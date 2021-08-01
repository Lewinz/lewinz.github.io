---
layout: post
title: gorm 自动创建 database
categories: [gorm]
description: gorm 自动创建 database
keywords: gorm
---

## gorm 自动创建 database

### tag 约定

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
size	指定列大小, 默认值 255
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
grom 的约定中，一般将数据模型中的 id 字段映射为数据表的主键，如下面定义的 testmodel,id 为主键，testmodel 的 id 的数据类型为 string，如果 id 的数据类型为 int，则 grom 还会为该设置 auto_increment，使用 id 成为自增主键。  

示例：
```golang
type TestModel struct{
    ID   int
    Name string
}
```

当然，我们也可以自定义主键字段的名称，示例的 Post 结构体，我们设置了 PostId 字段为主键，如果我们定义了其他字段为主键，那么，就算结构体中仍有 ID 字段，GROM 框架也不会把 ID 字段当作主键了。

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

当我们使用结构体创建数据表时，数据表的名称默认为结构体的小写复数形式，如结构体 Post 对应的数据表名称为 posts,当然我们也可以自己指定结构体对应的数据表名称，而不是用默认的。

为结构体加上 TableName() 方法，通过这个方法可以返回自定义的数据表名，如下：

```golang
//指定 Post 结构体对应的数据表为 my_posts
func (p Post) TableName() string{
    return "my_posts"
}
```

#### 数据表前缀（全局）

除了指定数据表名外，我们也可以重写 gorm.DefaultTableNameHandler 这个变量，这样可以为所有数据表指定统一的数据表前缀，如下：

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

上面的结构体 Prize 中的 PrizeName 字段对应的数据表为 prize_name，但我们把 PrizeName 改为 Prizename 时，则对应的数据表字段名称为 prizename,这是为因为只分隔大写字段开头的单词。

当然，我们也可以为结构体的某个字段定义 tags 扩展信息,这样结构体字段到数据表字段的映规则就在 tags 中定义。

#### 时间点追踪

前面我们说过，如果结构体中有名称为 ID 字段，则 GORM 框架会把该字段作为数据表的主键，除此之外，如果结构体中有 CreatedAt,UpdatedAt,DeletedAt 这几个字段的话，则 GROM 框架也会作一些特殊处理，规则如下：

```
CreatedAt：新增数据表记录的时候，会自动写入这个字段。
UpdatedAt：更新数据表记录的时候，会自动更新这个字段。
DeletedAt：当执行软删除的时候，会自动更新这个字段，表示删除时间
```

#### gorm.Model

由于如果结构体中有 ID,CreatedAt,UpdatedAt,DeletedAt 这几个比较通用的字段，GORM 框架会自动处理这几个字段，所以如果我们结构体需要这几个字段时，我们可以直接在自定义结构体中嵌入 gorm.Model 结构体，gorm.Model 的结构体如下：

```golang
type Model struct {
    ID        uint `gorm:"primary_key"`
    CreatedAt time.Time
    UpdatedAt time.Time
    DeletedAt *time.Time `sql:"index"`
}
```

所以，如果我们在结构体 Prize 中嵌入 gorm.Model,如下：
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

我们这里所说的数据库迁移，即通过使用 GROM 提供的一系列方法，根据数据模型定义好的规则，进行创建、删除数据表等操作，也就是数据库的 DDL 操作。

GORM 提供对数据库进行 DDL 操作的方法，主要以下几类：

#### 数据表操作

```golang
//根据模型自动创建数据表
func (s *DB) AutoMigrate(values ...interface{}) *DB 
 
//根据模型创建数据表
func (s *DB) CreateTable(models ...interface{}) *DB
 
//删除数据表，相当于 drop table 语句
func (s *DB) DropTable(values ...interface{}) *DB 
 
//相当于 drop table if exsist 语句
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
    Id       int   //对应数据表的自增 id
    Username string
    Password string
    Email    string
    Phone    string
}
 
func main(){
    db.AutoMigrate(&Post{},&User{})//创建 posts 和 users 数据表
 
    db.CreateTable(&Post{})//创建 posts 数据表
    
    db.Set("gorm:table_options", "ENGINE=InnoDB").CreateTable(&Post{})//创建 posts 表时指存在引擎
    
    db.DropTable(&Post{},"users")//删除 posts 和 users 表数据表
    
    db.DropTableIfExists(&Post{},"users")//删除前会判断 posts 和 users 表是否存在
    
    //先判断 users 表是否存在，再删除 users 表
    if db.HasTable("users") {
        db.DropTable("users")
    }
    
    //删除数据表字段
    db.Model(&Post{}).DropColumn("id")
    
    //修改字段数据类型
    db.Model(&Post{}).ModifyColumn("id","varchar(255)")
    
    //建立 posts 与 users 表之间的外键关联
    db.Model(&Post{}).AddForeignKey("uid", "users(id)", "RESTRICT", "RESTRICT")
    
    //给 posts 表的 title 字段添加索引
    db.Model(&Post{}).AddIndex("index_title","title")
    
    //给 users 表的 phone 字段添加唯一索引
    db.Model(&User{}).AddUniqueIndex("index_phone","phone")
}
```