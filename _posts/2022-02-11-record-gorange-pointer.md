---
layout: post
title: 记录 golang range 值拷贝的一个坑
categories: [golang, range, pointer]
description: 记录 golang range 值拷贝的一个坑
keywords: golang, range, pointer
---

## 问题背景
``` golang
type StorageTypeInfo struct{
  StorageTypes []StorageType
}

type StorageType struct{
  CategoryType string
  Others []Other
  ...
}

func stoTypeAppend(typeInfo *StorageTypeInfo, cateType string)
```
`stoTypeAppend` 函数的作用是在「`typeInfo.StorageTypes`」中寻找「`CategoryType`」与参数「`cateType`」相等的元素，对字段「`Others`」进行 append 操作。

所以我最开始的代码是这样的：
``` golang
type StorageTypeInfo struct {
	StorageTypes []StorageType
}

type StorageType struct {
	CategoryType string
	Others       []Other
}

type Other struct {
	Name string
}

func TestPointer(t *testing.T) {
	s := initStorageTypeInfo()

	fmt.Printf("before append ====== %#v\n\n", s)

	stoTypeAppend(s, "type1")

	fmt.Printf("after append ====== %#v\n\n", s)
}

func stoTypeAppend(typeInfo *StorageTypeInfo, cateType string) {
	for _, stoType := range typeInfo.StorageTypes {
		if stoType.CategoryType == cateType {
			stoType.Others = append(stoType.Others, Other{
				Name: "appendOther",
			})
			break
		}
	}
}

func initStorageTypeInfo() *StorageTypeInfo {
	return &StorageTypeInfo{
		StorageTypes: []StorageType{
			{
				CategoryType: "type1",
				Others: []Other{
					{
						Name: "other1",
					},
				},
			},
			{
				CategoryType: "type2",
				Others: []Other{
					{
						Name: "other2",
					},
				},
			},
		},
	}
}
```
输出结果：
``` shell
before append ====== &raccoon.StorageTypeInfo{StorageTypes:[]raccoon.StorageType{raccoon.StorageType{CategoryType:"type1", Others:[]raccoon.Other{raccoon.Other{Name:"other1"}}}, raccoon.StorageType{CategoryType:"type2", Others:[]raccoon.Other{raccoon.Other{Name:"other2"}}}}}

after append ====== &raccoon.StorageTypeInfo{StorageTypes:[]raccoon.StorageType{raccoon.StorageType{CategoryType:"type1", Others:[]raccoon.Other{raccoon.Other{Name:"other1"}}}, raccoon.StorageType{CategoryType:"type2", Others:[]raccoon.Other{raccoon.Other{Name:"other2"}}}}}
```
发现，`append` 之后 `CategoryType` 为 `type1` 的 `StorageTypes` 并没有像预期的一样增加长度。

## 排查问题
最开始以为是因为函数值传递的时候，拷贝指针时，参数下字段包含的切片地址传递的是第一个元素的地址（这部分的解释请看前面发布的博客内容），后面经过思考和单测之后，排除了这个可能，因为函数、方法在值传递的时候，只会拷贝值本身，例如这个例子传递的是指针，那么传递时会将指针类型对应的值拷贝一遍，但拷贝之后的值依旧是指向最原始的内存地址。

## 解决问题
经过深（qing）思（jiao）熟（da）虑（lao）之后，发现是因为 for range 时，`stoType` 是从切片中拷贝出来的值，所以对这个值进行改变，并不能修改切片的值。

``` golang
type StorageTypeInfo struct {
	StorageTypes []StorageType
}

type StorageType struct {
	CategoryType string
	Others       []Other
}

type Other struct {
	Name string
}

func TestPointer(t *testing.T) {
	s := initStorageTypeInfo()

	fmt.Printf("before append ====== %#v\n\n", s)

	stoTypeAppend(s, "type1")

	fmt.Printf("after append ====== %#v\n\n", s)
}

func stoTypeAppend(typeInfo *StorageTypeInfo, cateType string) {
	for index, stoType := range typeInfo.StorageTypes {
		if stoType.CategoryType == cateType {
			typeInfo.StorageTypes[index].Others = append(typeInfo.StorageTypes[index].Others, Other{
				Name: "appendOther",
			})
			break
		}
	}
}

func initStorageTypeInfo() *StorageTypeInfo {
	return &StorageTypeInfo{
		StorageTypes: []StorageType{
			{
				CategoryType: "type1",
				Others: []Other{
					{
						Name: "other1",
					},
				},
			},
			{
				CategoryType: "type2",
				Others: []Other{
					{
						Name: "other2",
					},
				},
			},
		},
	}
}
```

结果输出：
``` shell
before append ====== &raccoon.StorageTypeInfo{StorageTypes:[]raccoon.StorageType{raccoon.StorageType{CategoryType:"type1", Others:[]raccoon.Other{raccoon.Other{Name:"other1"}}}, raccoon.StorageType{CategoryType:"type2", Others:[]raccoon.Other{raccoon.Other{Name:"other2"}}}}}

after append ====== &raccoon.StorageTypeInfo{StorageTypes:[]raccoon.StorageType{raccoon.StorageType{CategoryType:"type1", Others:[]raccoon.Other{raccoon.Other{Name:"other1"}, raccoon.Other{Name:"appendOther"}}}, raccoon.StorageType{CategoryType:"type2", Others:[]raccoon.Other{raccoon.Other{Name:"other2"}}}}}
```

## 总结
其实不止是 slice，map 在 for range 的时候也会进行值拷贝，当我们使用 for range 获取到数据时，最好不要使用指针进行数据修改，以免掉入各种坑。