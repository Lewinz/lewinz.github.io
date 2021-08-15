---
layout: post
title: golang 执行外部命令
categories: [golang, cli]
description: golang 执行外部命令
keywords: golang, cli
---

## Golang 执行系统命令使用 os/exec Command 方法
``` golang
func Command(name string, arg ...string) *Cmd
```
第一个参数是命令名称，后面参数可以有多个命令参数。

``` golang
cmd := exec.Command("ls", "-a")
if stdout, err := cmd.StdoutPipe(); err != nil {     //获取输出对象，可以从该对象中读取输出结果
    log.Fatal(err)
}

defer stdout.Close()   // 保证关闭输出流
if err := cmd.Start(); err != nil {   // 运行命令
  log.Fatal(err)
}

if opBytes, err := ioutil.ReadAll(stdout); err != nil {  // 读取输出结果    
  log.Fatal(err)
} else {
  log.Println(string(opBytes))
}
```

## 将命令的输出结果重定向到文件中
``` golang
stdout, err := os.OpenFile("stdout.log", os.O_CREATE|os.O_WRONLY, 0600)   
if err != nil {
  log.Fatalln(err)
}

defer stdout.Close()

cmd.Stdout = stdout   // 重定向标准输出到文件
// 执行命令
if err := cmd.Start(); err != nil {
  log.Println(err)
}
```
## cmd 的 Start 和 Run 方法的区别
Start 执行不会等待命令完成，Run 会阻塞等待命令完成。
``` golang
cmd := exec.Command("sleep", "10")
err := cmd.Run()  //执行到此处时会阻塞等待10秒
err := cmd.Start()   //如果用start则直接向后运行
if err != nil {
  log.Fatal(err)
}

err = cmd.Wait()   //执行Start会在此处等待10秒
```