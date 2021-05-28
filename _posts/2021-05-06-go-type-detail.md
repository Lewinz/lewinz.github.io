---
layout: post
title: golang基本类型
categories: [golang,基本类型]
description: golang基本类型
keywords: golang,基本类型
---

## 数据类型默认值

```
int：0
float：0
string：""
bool：false
```

## 整数类型

```
int/uint、int/uint(8/16/32/64)
    其中8/16…是位，1字节等于8位，具体表示几个字节以及申明此类型的变量取值范围等自己算。注意：区分有符号位和无符号位！
rune：等价int32，表示一个unicode码；有符号
byte：等价uint8，存储字符；无符号
```

## 浮点类型 符号位+指数位+尾数位

```
单精度 float32 4字节
双精度 float64 8字节 （go默认）

表示形式：
    十进制 1.23、.23 =0.23(必须有小数点)
    科学计术法 123E2、23e-2
```

## 布尔类型

只能为true或false  

占一个字节

## 字符串

go统一使用utf-8

字符串类型已赋值，则该变量不可变

双引号("")识别转义字符

反引号(``)原生输出

字符串拼接 str1 + str2；若字符串过多 + 留在上一行

## 类型转换

int/float：type(b) 将b转为type类型  
注意：**转换后的值需要赋给新变量，原变量类型不变。 小心溢出**


>基本数据类型与string  
>>1. 基本数据类型转string  
>>> fmt.Pringf(%参数，表达式)  
>>>使用strconv  
>>2. string转基本数据类型  
>>>使用strconv  

## 派生数据类型

指针 pointer  
数组  
结构体 struct  
管道 channel  
函数  
切片 slice  
接口 interface  
map  