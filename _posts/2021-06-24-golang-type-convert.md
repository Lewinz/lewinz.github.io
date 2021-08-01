---
layout: post
title: Golang 基础类型转换
categories: [golang,转换 ]
description: Golang 基础类型转换
keywords: golang,转换
---

### string
``` golang
// string 到 int
// Atoi 是 ParseInt(s, 10, 0) 的简写
int, err := strconv.Atoi(string)

// string 到 int64
int64, err := strconv.ParseInt(string, 10, 64)

// string 到 float64
float, err := strconv.ParseFloat(string, 64)

// string 到 float32
float, err = strconv.ParseFloat(string, 32)
```

### int/int64
``` golang
// int 转 int64
int64_ := int64(int)

// int 转 float32
b := float64(a)

// int 转 float64
b := float32(a)

// int 转 string
string := strconv.Itoa(int)
=
string := strconv.FormatInt(int64(int),10)

// int64 转 string
//第二个参数为基数，可选 2~36
//对于无符号整形，可以使用 FormatUint(i uint64, base int)
string := strconv.FormatInt(int64,10)
```

### float32/float64
``` golang
// float 转 string
// 'b' (-ddddp±ddd，二进制指数)
// 'e' (-d.dddde±dd，十进制指数)
// 'E' (-d.ddddE±dd，十进制指数)
// 'f' (-ddd.dddd，没有指数)
// 'g' ('e':大指数，'f':其它情况)
// 'G' ('E':大指数，'f':其它情况)
string := strconv.FormatFloat(float32,'E',-1,32)
string := strconv.FormatFloat(float64,'E',-1,64)

// float 转 int   四舍五入
value := int64(math.Floor(a+0.5))

// float 精度转换
value,err := strconv.ParseFloat(fmt.Sprintf("%.2f", value), 64)
```

### interface 转 其他类型
``` golang
inter.(type)
```