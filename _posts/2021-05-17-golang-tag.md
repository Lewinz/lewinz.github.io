---
layout: post
title: Golang Tag 汇总
categories: [golan,tag,汇总]
description: Golang Tag 汇总
keywords: golan,tag,汇总
---

## Tag
在 Go 的 struct 定义中，有时需要对某个字段添加额外的元信息，这就需要用到 field tag，即成员标签变量。

### Json Tag
#### 示例
```golang
type peerInfo struct {
	HTTPPort int    `json:"http_port"`
	TCPPort  int    `json:"tcp_port"`
	versiong string `json:"versiong"`
}
```
#### Encode 和 Decode
##### Encode
要把 golang 的数据结构转换成 JSON 字符串（encode），可以使用 Marshal函数
```golang
func Marshal(v interface{}) ([]byte, error)
```
##### Decode
相对应的，要把 JSON 数据转换成 Go 类型的值（Decode）， 可以使用 json.Unmarshal。
```golang
func Unmarshal(data []byte, v interface{}) error
```
data 中存放的是 JSON 值，v 会存放解析后的数据，所以必须是指针，可以保证函数中做的修改能保存下来  

#### 更多控制
Json Tag控制字段有三种：
* -：不要解析这个字段  
* omitempty：当字段为空（默认值）时，不要解析这个字段。比如 false、0、nil、长度为 0 的 array，map，* * 
slice，string  
* FieldName：当解析 json 的时候，使用这个名字

### Xml Tag
导包`import "encoding/xml"`  

#### 解析和读取规则
golang对xml的解析和读取是通过stuct和refect实现的,对于struct中的tag以什么方式对应到xml的元素上,golang的文档中做了如下描述:

``` txt
1、结构体中的XMLName字段或者类型为xml.Name的字段,会被删除.使用此字段tag上定义的属性进行解析
2、结构体tag中”-” 在解析过程中会忽略结构体中的这个字段
3、结构体tag中”name,attr” 使用name作输出为xml属性,对应字段值作为属性值
4、结构体tag中”,attr” 使用字段名作为xml属性,字段值作为xml属性值
5、结构体tag中”,chardata” 不作为xml的节点输出,把该字段对应的值作为字符输出
6、结构体tag中 “,innerxml” 如果结构体改字段是基本类型如:string,int等,和”,chardata”输出无区别,如果是一个结构体,输出值会是一个完整的xml结构
7、结构体tag中 “,comment” 输出xml中的注释
8、结构体tag中”omitempty” 该字段是go中的空值:false, 0,空指针,空接口,任何长度为0的切片,数组,字符串和map. 都会被忽略
9、结构体中不包含tag 会以该字段作为xml属性名称,值作为xml属性值
```

代码示例
```golang
type TNote struct {
    Lang    string `xml:"lang,attr"`
    Content string `xml:",innerxml"`
}

type TFile struct {
    XMLName  struct{} `xml:"file"`
    FileName string   `xml:"name,attr"`
    Size     string   `xml:"size,attr"`
}

type Release struct {
    XMLName   struct{} `xml:"release"`
    Version   string   `xml:"version,attr"`
    TimeStamp string   `xml:",attr"`
    Lang      string   `xml:"-"`
    Skin      string   `xml:",chardata"`
    Site      string   `xml:",omitempty"`
    File      []TFile  `xml:",innerxml"`
    CnNotes   TNote    `xml:"cnnote"`
    EnNotes   TNote    `xml:"ennote"`
    Comment   string   `xml:",comment"`
}
```

### gorm Tag
**支持的结构标签**
|   标签    |   说明    |
|   ----    |   ----    |
|   column    |   指定列的名称    |
|   type    |   指定列的类型    |
|   size    |   指定列的大小，默认是 255    |
|   primary_key    |   指定一个列作为主键    |
|   unique    |   指定一个唯一的列    |
|   default    |   指定一个列的默认值    |
|   precision    |   指定列的数据的精度    |
|   not null    |   指定列的数据不为空    |
|   auto_increment    |   指定一个列的数据是否自增    |
|   index    |   创建带或不带名称的索引，同名创建复合索引    |
|   unique_index    |   类似 索引，创建一个唯一的索引    |
|   embedded    |   将 struct 设置为 embedded    |
|   embedded_prefix    |   设置嵌入式结构的前缀名称    |
|   -    |   忽略这些字段    |

**关联的结构标签**
|   标签    |   说明    |
|   ----    |   ----    |
|   many2many    |   指定连接表名称    |
|   foreignkey    |   指定外键    |
|   association_foreignkey    |   指定关联外键    |
|   polymorphic    |   指定多态类型    |
|   polymorphic_value    |   指定多态的值    |
|   jointable_foreignkey    |   指定连接表的外键    |
|   association_jointable_foreignkey    |   指定连接表的关联外键    |
|   save_associations    |   是否自动保存关联    |
|   association_autoupdate    |   是否自动更新关联    |
|   association_autocreate    |   是否自动创建关联    |
|   association_save_reference    |   是否引用自动保存的关联    |
|   preload    |   是否自动预加载关联    |

### Map Tag
**mapstructure 大小写不敏感**
示例：
```golang
type Person struct {
  Name string `mapstructure:"username"`
}
```

#### 内嵌结构
内嵌结构可设置```mapstructure:",squash"```将该结构体的字段提到父结构中  
另外需要注意一点，如果父结构体中有同名的字段，那么mapstructure会将JSON 中对应的值同时设置到这两个字段中，即这两个字段有相同的值。

示例：
```golang
type Friend struct {
  Person `mapstructure:",squash"`
}

type Person struct {
  Name string
}
```

#### 未映射的值
如果源数据中有未映射的值（即结构体中无对应的字段），mapstructure默认会忽略它。

我们可以在结构体中定义一个字段，为其设置mapstructure:",remain"标签。这样未映射的值就会添加到这个字段中。注意，这个字段的类型只能为`map[string]interface{}`或`map[interface{}]interface{}`。

示例：
```golang
type Person struct {
  Name  string
  Age   int
  Job   string
  Other map[string]interface{} `mapstructure:",remain"`
}
```

#### 逆向转换
前面我们都是将`map[string]interface{}`解码到 Go 结构体中。mapstructure当然也可以将 Go 结构体反向解码为`map[string]interface{}`。在反向解码时，我们可以为某些字段设置`mapstructure:",omitempty"`。这样当这些字段为默认值时，就不会出现在结构的`map[string]interface{}`中：

```golang
type Person struct {
  Name string
  Age  int
  Job  string `mapstructure:",omitempty"`
}
```

#### Metadata
解码时会产生一些有用的信息，mapstructure可以使用Metadata收集这些信息。Metadata结构如下：
```golang
// mapstructure.go
type Metadata struct {
  Keys   []string
  Unused []string
}
```

Metadata只有两个导出字段：
* Keys：解码成功的键名；
* Unused：在源数据中存在，但是目标结构中不存在的键名。

为了收集这些数据，我们需要使用DecodeMetadata来代替Decode方法：
```golang
type Person struct {
  Name string
  Age  int
}

func main() {
  m := map[string]interface{}{
    "name": "dj",
    "age":  18,
    "job":  "programmer",
  }

  var p Person
  var metadata mapstructure.Metadata
  mapstructure.DecodeMetadata(m, &p, &metadata)

  fmt.Printf("keys:%#v unused:%#v\n", metadata.Keys, metadata.Unused)
}
```

#### 弱类型输入
有时候，我们并不想对结构体字段类型和`map[string]interface{}`的对应键值做强类型一致的校验。这时可以使用`WeakDecode/WeakDecodeMetadata`方法，它们会尝试做类型转换：

```golang
type Person struct {
  Name   string
  Age    int
  Emails []string
}

func main() {
  m := map[string]interface{}{
    "name":   123,
    "age":    "18",
    "emails": []int{1, 2, 3},
  }

  var p Person
  err := mapstructure.WeakDecode(m, &p)
  if err == nil {
    fmt.Println("person:", p)
  } else {
    fmt.Println(err.Error())
  }
}
```
虽然键name对应的值123是int类型，但是在WeakDecode中会将其转换为string类型以匹配Person.Name字段的类型。同样的，age的值"18"是string类型，在WeakDecode中会将其转换为int类型以匹配Person.Age字段的类型。 需要注意一点，如果类型转换失败了，WeakDecode同样会返回错误。例如将上例中的age设置为"bad value"，它就不能转为int类型，故而返回错误。

参考博客：  
<https://zhuanlan.zhihu.com/p/165419292>