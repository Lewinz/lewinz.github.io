---
layout: post
title: golang 字符串中的潜在风险
categories: [golang,字符串 ]
description: golang 字符串中的潜在风险
keywords: golang,字符串
---

**出处**： [Go 语言中文网微信公众号发布文章](https://mp.weixin.qq.com/s/_ScaGaB8mr2aSyQjL-VIaA)  

在我之前的文章 Go 中我喜欢的东西中提到过，我喜欢的 Go 的东西其中之一就是它的字符串（通常还有切片）。

从一个 Python 开发者的角度看，它们之所以伟大，是因为创建它们时开销很少，因为它们通常不需要复制。在 Python 中，任何时候操作字符串都需要复制一部分或全部字符串，而 这很容易对性能造成影响。想要写高性能的 Python 代码需要谨慎考虑复制的问题。在 Go 中，几乎所有的字符串操作都是不复制的，仅仅是从原字符串取一个子集（例如去除字符串首尾的空白字符），因此你可以更自由地操作字符串。这个机制可以非常直接地解决你的问题，并且非常高效。（当然，不是所有的字符串操作都不复制。例如，把一个字符串转换成大写需要复制，尽管 Go 中的实现已经足够智能，在不需要改变原字符串时 — 例如由于它已经是一个全大写的字符串 — 可以规避掉复制。）

但是这个优势也带来了潜在的坏处，那些没有开销的子字符串使原来的整个字符串一直存在于内存中。Go 中的字符串（和切片）操作之所以内存开销很少，是因为它们只是底层存储（字符串或切片底层的数组的真实数据）的一些部分的引用；创建一个字符串做的操作就是创建了一个新的引用。

但是 Go（目前）不会对字符串数据或数组进行部分的垃圾回收，所以即使它一个很小的 bit 被其它元素引用，整个对象也会一直保持在内存中。换句话说，一个单字符的字符串（目前）足够让一个巨大的字符串不被 GC 回收。

当然，不会有很多人遇到这个问题。为了遇到它，你需要处理一个非常庞大的原字符串，或造成大量的内存消耗（或者两者都做），在这个基础上，你必须创建那些不持久的字符串的持久的小子字符串（好吧，你是多么希望它是非持久的）。很多使用场景不会复现这个问题；要么你的原字符串不够大，要么你的子集获取了大部分原字符串（例如你把原字符串进行了分词处理），要么子字符串生命周期不够长。简而言之，如果你是一个普通的 Go 开发者，你可以忽略这个问题。处理长字符串并且长时间维持原字符串的很小部分的人才会关注这个问题。

（我之所以关注到这个问题，是因为一次我花了大量精力用尽可能少的内存写 Python 程序，尽管它是从一个大的配置文件解析结果然后分块储存。这让我联想到了一些其他的事，如字符串的生命周期、限制字符串只复制一次，等等。然后我用 Go 语言写了一个解析器，这让我由重新考虑了一下这些问题，我意识到由于我的解析器截取出和维持的 bit 一直存在于内存中，从输入文件解析出的庞大字符串也会一直存在与内存中。）

顺便说一下，我认为这是 Go 做了权衡之后的正确结果。大部分使用字符串的开发者不会遇到这个问题，而且截取子字符串开销很小对于开发者来说用处很大。这种低开销的截取操作也减轻了 GC 的负担；当代码使用大量的子字符串截取（像 Python 中那样）时，你只需要处理固定长度的字符串引用就可以了，而不是需要处理长度变化的字符串。

当你的代码遇到这个问题时，当然有明显的解决方法：创建一个函数，通过把字符串转换成 []byte 来 ”最小化“ 字符串，然后返回。这种方法生成了一个最小化的字符串，内存开销是理论上最完美实现的只复制一次，而 Go 现在很容易就可以实现。
