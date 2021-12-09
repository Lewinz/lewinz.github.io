---
layout: post
title: gorm/mysql timestamp 异常引发的 debug
categories: [golang, gorm, timestamp]
description: gorm/mysql timestamp 异常引发的 debug
keywords: golang, gorm, timestamp
---

## 问题上下文
因为业务需要，写了一个每小时执行一次的自动任务，用来生成每小时的计量数据  
计量表存储的数据为每小时