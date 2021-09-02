---
layout: post
title: golang 文件操作权限位
categories: [golang, file]
description: golang 文件操作权限位
keywords: golang, file
---

## OpenFile
Go 语言的 os 包下有一个 OpenFile 函数，其原型如下所示：
``` golang
func OpenFile(name string, flag int, perm FileMode) (file *File, err error)
```

其中 name 是文件的文件名，如果不是在当前路径下运行需要加上具体路径；flag 是文件的处理参数，为 int 类型，根据系统的不同具体值可能有所不同，但是作用是相同的。

flag 文件处理参数：
- O_RDONLY：只读模式 (read-only)
- O_WRONLY：只写模式 (write-only)
- O_RDWR：读写模式 (read-write)
- O_APPEND：追加模式 (append)
- O_CREATE：文件不存在就创建 (create a new file if none exists.)
- O_EXCL：与 O_CREATE 一起用，构成一个新建文件的功能，它要求文件必须不存在 (used with O_CREATE, file must not exist)
- O_SYNC：同步方式打开，即不使用缓存，直接写入硬盘
- O_TRUNC：打开并清空文件

perm 参数：  
除非创建文件时才需要指定，不需要创建新文件时可以将其设定为０. 虽然 go 语言给 perm 权限设定了很多的常量，但是习惯上也可以直接使用数字，如 0666 (具体含义和 Unix 系统的一致).

示例：
``` golang
package main

import (
    "bufio"
    "fmt"
    "os"
)

func main() {
  //创建一个新文件
  filePath := "./test_open_file.txt"
  file, err := os.OpenFile(filePath, os.O_WRONLY|os.O_CREATE, 0666)
  if err != nil {
      fmt.Println("文件打开失败", err)
      return
  }

  //及时关闭file句柄
  defer file.Close()

  //写入文件时，使用带缓存的 *Writer
  write := bufio.NewWriter(file)
  for i := 0; i < 5; i++ {
      write.WriteString("Hello Word! \n")
  }
  
  //Flush将缓存的文件真正写入到文件中
  write.Flush()
}
```