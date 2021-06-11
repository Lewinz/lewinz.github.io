---
layout: post
title: 监控测试方法-跑满CPU
categories: [监控,CPU]
description: 监控测试方法-跑满CPU
keywords: 监控,CPU
---

### 命令
``` sh
for i in `seq 1 $(cat /proc/cpuinfo |grep "processor" |wc -l)`; do dd if=/dev/zero of=/dev/null; done
```

### 分析
首先，我们把这一行命令拆分一下，增强下代码的可读性：
``` sh
#!/bin/bash
N=`cat /proc/cpuinfo | grep "processor" | wc -l`
List=`seq 1 $N`
for i in $List
do
dd if=/dev/zero of=/dev/null
done
```
下面来解释下每一行吧。
```sh
cat /proc/cpuinfo | grep "processor" | wc -l
```
这条最简单了，获取CPU核数。
```sh
seq 1 N
```
根据N的值生成一个序列。
``` sh
for i in `seq 1 N`
```
循环执行命令，从１到Ｎ。
``` sh
dd if=/dev/zero of=/dev/null
```
执行dd命令，输出到`/dev/null`, 实际上只占用CPU，没有IO操作。  
由于连续执行Ｎ个的dd 命令, 且使用率为100%，这时调度器会调度每个dd命令在不同的CPU上处理，最终就实现所有CPU使用率达到100%。  

### Kill方法
还是要履行下运维的职责，把这个引起CPU使用率100%的进程干掉。
`pkill -9 dd`

