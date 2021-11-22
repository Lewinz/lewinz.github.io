---
layout: post
title: 时间戳
categories: [时间戳 ]
description: 时间戳
keywords: 时间戳
---

## 基本概念

定义：时间戳是指格林威治时间 1970 年 01 月 01 日 00 时 00 分 00 秒 (北京时间 1970 年 01 月 01 日 08 时 00 分 00 秒) 起至现在的总秒数。

时间戳（timestamp），通常是一个字符序列，唯一地标识某一刻的时间。

## 个人理解
时间戳是标识一个时间节点，时间戳的计时起点根据各时区的不同相应提前或推后（我是这么理解的！），所以从这个角度上看，**时间戳没有时区这句话**，是个伪命题。（每个人的理解不用，表述不同，因人而异。）

## golang 注意点
`time.unix()` 这个方法，会根据 time 的 Location 属性不同，输出的时间戳也会不同，所以在 time 初始化的时候注意一定要赋予正确的 Location。