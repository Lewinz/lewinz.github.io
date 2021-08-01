---
layout: post
title: 监控测试方法-跑满 CPU
categories: [监控,CPU]
description: 监控测试方法-跑满 CPU
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
这条最简单了，获取 CPU 核数。
```sh
seq 1 N
```
根据 N 的值生成一个序列。
``` sh
for i in `seq 1 N`
```
循环执行命令，从１到Ｎ。
``` sh
dd if=/dev/zero of=/dev/null
```
执行 dd 命令，输出到`/dev/null`, 实际上只占用 CPU，没有 IO 操作。  
由于连续执行Ｎ个的 dd 命令, 且使用率为 100%，这时调度器会调度每个 dd 命令在不同的 CPU 上处理，最终就实现所有 CPU 使用率达到 100%。  

### Kill 方法
还是要履行下运维的职责，把这个引起 CPU 使用率 100%的进程干掉。
`pkill -9 dd`

