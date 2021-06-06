---
layout: post
title: golang Json Tag 详细用法
categories: [golang,json,tag]
description: golang Json Tag 详细用法
keywords: golang,json,tag
---

Go中使用json很多，玩法也很多，整理了一下go中关于json的tag使用
包含了一些遇到的问题与解决方案

> A field declaration may be followed by an optional string literal tag, which becomes an attribute for all the fields in the corresponding field declaration. The tags are made visible through a reflection interface but are otherwise ignored.

官方的解释是这个标签信息可以通过反射获取，自定义一些规则，比如常见的db json
Tag是一个以反引号 `包围， 空格分隔的 k:v 对，常用于数据解析，关系映射等

Go中数据类型与Json支持的类型对应关系

``` golang
bool                    ==> JSON booleans
float64                 ==> JSON numbers
string                  ==> JSON strings
[]interface{}           ==> JSON arrays
map[string]interface{}  ==> JSON objects
nil                     ==> JSON null
```

``` golang
package main

import (
	"encoding/json"
	"fmt"
	"time"
)

type User struct {
	Age      uint8     `json:"age"`
	Name     string    `json:"name"`
	Address  string    `json:"address,omitempty"`
	Phone    []string  `json:"phone"`
	Skill    []string  `json:"skill"`
	Birthday time.Time `json:"birthday,omitempty"`
	Password string    `json:"-"`
}

func main() {
    l,_ := time.LoadLocation("Asia/Shanghai")
	birthday := time.Date(2000, 1, 1, 10, 8, 0, 0, l)
	someOne := User{
		Age:      20,
		Name:     "老王",
		Address:  "",
		Skill:    []string{},
		Phone:    []string{"18888888888","15555555555"},
		Birthday: birthday,
		Password: "admin123",
	}
	b, err := json.Marshal(someOne)
	if err != nil {
		fmt.Printf("json.Marshal failed, err:%v\n", err)
		return
	}
	fmt.Printf("%s\n", b)
}
```

### 指定json字段名
User结构体中，json标签指定了它的输出字段名，执行将输出

``` json
{
    "age":20,
    "name":"老王",
    "phone":[
        "18888888888",
        "15555555555"
    ],
    "skill":null,
    "birthday":"2000-01-01T10:08:00+08:00"
}
```

### 输出了null
切片、map、等零值为nil，对应json的值为null  
如果要输出[]则要赋值为空切片
``` golang
// someOne赋值为
someOne := User{
		Age:      20,
		Name:     "老王",
		Address:  "",
		Skill:    []string{},
		Phone:    []string{"18888888888","15555555555"},
		Birthday: birthday,
		Password: "admin123",
	}
// 对应的输出
{
    "age":20,
    "name":"老王",
    "skill":[

    ],
    "phone":[
        "18888888888",
        "15555555555"
    ],
    "birthday":"2000-01-01T10:08:00+08:00"
}
```
>要注意的是空map对应的json输出为 {} 而非 []

### 忽略零值字段
通过设置`tag`添加`omitempty`，当字段值为其类型对应的零值时，可达到忽略零值字段的目的。  
在输出结果中`Address`被设置为空字符串，也未被输出，因为`string`类型的零值即为`""`

### 忽略指定字段
`User`结构体中`Password`的`tag`被指定为`-`所以，即便被设置了具体的值，在json中也未被输出

### 结构体嵌套输出
比如将用户的技能和新增的个人主页地址，并将一些展示信息独立为`Profile`结构

``` golang
type Profile struct {
    Phone    []string  `json:"phone,omitempty"`
    Skill    []string  `json:"skill,omitempty"`
    Birthday time.Time `json:"birthday,omitempty"`
    HomePage string    `json:"url"`
    Address  string    `json:"address,omitempty"`
}

type User struct {
    Profile

	Age      uint8     `json:"age"`
	Name     string    `json:"name"`
	Password string    `json:"-"`
}

// someOne 赋值为
someOne := User{
    Profile: Profile{
        Skill: []string{"吃","喝","玩","乐"},
        Birthday: birthday,
    },

    Age:      20,
    Name:     "老王",
    Password: "admin123",
}
// 对应输出为
{
    "skill":[
        "吃",
        "喝",
        "玩",
        "乐"
    ],
    "birthday":"2000-01-01T10:08:00+08:00",
    "url":"",
    "age":20,
    "name":"老王"
}
```
由于是匿名嵌套，所以Profile中的字段与User同级输出

### 层级输出
```golang
type User struct {
    Profile `json:"profile"`

	Age      uint8     `json:"age"`
	Name     string    `json:"name"`
	Password string    `json:"-"`
}
```
输出为
``` json
{
    "profile":{
        "skill":[
            "吃",
            "喝",
            "玩",
            "乐"
        ],
        "birthday":"2000-01-01T10:08:00+08:00",
        "url":""
    },
    "age":20,
    "name":"老王"
}
```

### 忽略嵌套结构体
仅设置`omitempty`是无法屏蔽嵌套字段的输出的

```golang
type User struct {
    Profile `json:"profile,omitempty"`

	Age      uint8     `json:"age"`
	Name     string    `json:"name"`
	Password string    `json:"-"`
}
```
输出为
``` json
{
    "profile":{
        "birthday":"0001-01-01T00:00:00Z",
        "url":""
    },
    "age":20,
    "name":"老王"
}
```
解决方案为使用嵌套结构体指针
``` golang
type User struct {
    *Profile `json:"profile,omitempty"`

	Age      uint8     `json:"age"`
	Name     string    `json:"name"`
	Password string    `json:"-"`
}
```
输出为
``` json
{"age":20,"name":"老王"}
```

### 忽略字段输出（不修改原结构体）
比如要忽略`Name`输出，利用的是定义相同json tag来完成
可使用任意类型，空struct有利于节省内存及统一
``` golang
type Omit *struct{}
type JsonUser struct {
	*User
	Name Omit `json:"name,omitempty"`
}
```
临时使用的话，一个匿名结构体也可以

### json.RawMessage 延迟解析
可通过一个结构体灵活匹配多种类型定义
``` golang
type MultiType struct {
	Type string `json:"type"`
	Data json.RawMessage `json:"data"`
}
```
可以通过先匹配type再对应解析数据，如
```json
{
	"type":"array",
	"data": [1,2,3,4]
}
{
	"type":"string",
	"data": "any"
}
```

### json传递数字
json的数字，对应的是go的float64类型
这就意味着只有显示的指定字段类型，go才会将数字转换，所以如果超出float64范围的数字，就需要特殊处理  
另外，如果字段定义为数字类型，int int8等，但入参为字符串"42"这种，会解析错误  
以上问题可通过使用`json.Number`类型解决，见[参考链接](https://www.jianshu.com/p/bbcc81074089) 

``` golang
type User struct {
	Age      json.Number     `json:"age"`
	Name     string          `json:"name"`
	Password string          `json:"-"`
}
```
如果确定入参类型只为字符串类型数字，也可通过tag中指定类型来解决
>通过这种定义，输出的数字也将是字符串类型
``` golang
type User struct {
	Age      uint8     `json:"age,string"`
	Name     string    `json:"name"`
	Password string    `json:"-"`
}
```

### 自定义规则
``` golang
type Marshaler interface {
    MarshalJSON() ([]byte, error)
}
type Unmarshaler interface {
    UnmarshalJSON([]byte) error
}
```

通过实现以上两个接口，达到自定义json格式或解析的目的  
比如修改time的输出格式以及兼容各种格式的时间字符串解析，时间戳的转换等  
扩展类型的方式或者内嵌方式  
``` golang
type CustomTime struct {
    time.Time
}

// MarshalJSON 输出常用的时间格式
func (t CustomTime) MarshalJSON() ([]byte, error) {
    tune := t.Format(`"2006-01-02 15:04:05"`)
    return []byte(tune), nil
}
```