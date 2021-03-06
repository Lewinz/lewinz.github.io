---
layout: post
title: linux 与 freeBSD
categories: [linux, FreeBSD]
description: linux 与 freeBSD
keywords: linux, FreeBSD
---

传说中 FreeBSD 比 linux 稳定，大型网站几乎都建立在 FreeBSD 系统上，我一直疑惑难道 linux 是否真的不能做大型网站。于是用 netcraft 网站做了个测试：  
<http://toolbar.netcraft.com/site_report?url=www.phpchina.com>  
按照上面的链接你就可以查询任何一个网站的服务器架构，当然，可信度和准确度我不能保证。下面是我测试的案例：

- www.phpchina.com 清一色 linux;
- www.tencent.com 清一色 linux;
- www.qq.com 清一色 linux;
- www.taobao.com linux;
- www.ebay.com.cn 查询显示 OS 清一色 linux，WebServer 清一色 IIS，令人费解；
- www.alibaba.com 清一色 linux;
- www.bokee.com 清一色的 linux；
- www.google.com 清一色 linux;
- www.pconline.com.cn linux;
- www.yninfo.com 清一色的 linux;
- www.tom.com 清一色 Debian;
- www.cctv.com linux+sun 的服务器；
- www.126.com 清一色 linux
- www.163.com 清一色 linux，大家或许都认为网易是使用 FreeBSD 的，但 163/126 就全部用上了 linux，令人费解。

看来用 linux 做大站的也不少啊！！！谁说 linux 不能做大站呢？

另外又发现两个奇怪的东东：
- www.ebay.com 居然清一色的 win2000!!!
- http://www.myspace.com 全美访问量第一，居然也清一色的 win2003;

在我印象中，大型网站是压根不能用 windos 系统的。但这两个案例给我的理论一个有力的回击：** 系统稳定与否，关键还是在人！**

无论是 Windows 还是 FreeBSD 还是 Linux 都可以做大型网站，只要人足够牛 X 就行。这里不谈 windows 了，还是从大家口水仗打得最厉害的 linux 和 freebsd 分析分析吧。

首先说明一点：为什么不拿 linux 和 windows 比较，而只是和 freebsd 比较呢？答案在于 linux 或是 freebsd 都感觉到了对方带来的压力，都认定对方是自己的竞争对手。既然称得上是对手，自然是各有所长，难分轩轾，谁也不能把谁压倒罢了。

论坛里争论 FreeBSD 和 linux 谁谁更好，其实是从一个静态的角度来看的，在某个特定时间里，FreeBSD 或许比 linux 更稳定，linux 或 许比 FreeBSD 更快捷，但两家都在动态发展，没有谁永远领先，没有谁永远落后，FreeBSD 稳定的特性，Linux2.6 可以超越它；而 linux 快捷的优势，FreeBSD 也会迅速居上。我就不信，linus 和他的黑客团队在技术上会输给学院派的 FreeBSD 团队？或者 FreeBSD 的高手们比不上一群黑客，？他们谁都可以暂时领先，谁都可以暂时落后，但谁都不是吃干饭的！

目前流行这么一种传说：linux 和 freebsd 内核性能上相比：linux2.2 比 freebsd 要差，linux2.4 和 freebsd 难分伯仲， 而 linux2.6 比 freebsd 好得多。这里 freebsd 被静态化了，以一个动态发展的 linux 去比较某个固定版本的 freebsd，显然是有失 公平的。有道是：士别三日，即更刮目相看，更何况是技术日新月异的 IT 行业！

又有这么一种说法：LINUX 被黑的多而 FreeBSD 被黑的少，盖出于安全性较逊？这也是无稽之谈，用 liunx 的人基数比 freebsd 大，菜鸟自然也就更多了。系统安不安全关键在人，如果你不信，可以尝试去黑一下 www.ebay.com 或 www.myspace.com，他们的服务器可都是 windows 哟。

其实两家最根本的差别不在技术，而在于设计理念：linux 不求最稳，但求最新；FreeBSD 不求最新，只求最稳 —— 这样说也许不对，但也能反映一些问题。

我对 FreeBSD 与 Linux 比较的最终结论是：谁好谁稳定都只是暂时的，两家的存在状态，是一个 “既生瑜何生亮” 的问题，在长久的发展过程中，技术 上的常胜将军并不存在，双方只有此消彼长，各领风骚。至于大家为什么非要证明 FreeBSD 比 Linux 好或 Linux 比 FreeBSD 好，我想程序员普 遍都喜欢追求完美，非要用最好最完美的系统才甘心吧！

FreeBSD 和 Linux 我都用过，不在超大型应用中，很难感受两者的差别。个人选择的 linux，考虑到使用 linux 的人比较多，商机自然也就更多吧，钱在哪眼光就看哪，至少 linux 的就业机会比 FreeBSD 多。当然，这是非技术因素的考虑了。

------------------------------------------------------------------------------------------------------

从 linux 迁移到 freebsd

FreeBSD 和各种 Linux 发行版一样，是一种免费的 Unix 类操作系统。由于 FreeBSD 的推出稍晚于 Linux，因此国内介绍 FreeBSD 的文章较少，而且 由于某些煤体不负责任的推波助澜，Linux 被过分夸大、神化了。实际上，很多大网站都在用 FreeBSD，如 Yahoo，甚至包括 Microsoft 的 Hotmail（Microsoft 收购 Hotmail 4 年了，但直到去年 11 月 Microsoft 才宣布 Hotmail 向 Windows/IIS 迁移，而且，至今 Hotmail 主机群中依然包括 FreeBSD Boxes，可见 Microsoft 对于 Windows 并不是真的很放心，这一事实也证明了 FreeBSD 引以为耀的稳定性）等等。和 Linux 各种混乱 不堪的发行版相比，FreeBSD 只有唯一的版本，同时，FreeBSD 关注的是操作系统的稳定性、性能和品质，适合作为服务器的操作系统。当然，对于选 择 FreeBSD 还是某种 Linux 发行版本作为操作系统，不同的人肯定有不同的偏爱，但译者认为，FreeBSD 作为服务器来说，比 Linux 更好一 些，当然，如果不考虑开发成本，仅仅考虑运行效率，也更远好于 Windows。原因很多，有机会的话，我会在今后的文章中一一提到。

需要说明的是，译者不认为最近一两年之内免费操作系统阵营能够和 Microsoft 等商业系统阵营决出胜负高下，因为 他们的操作系统的设计理念存在差别。我自己很喜欢命令行方式的操作，然而在图形界面方面，X Window 的效率是很难超过 Windows 的，这是因为 X 是以用户模式运行图形界面，而 Windows 则是以核心模式运行，这实际上就造成了 Unix 类 操作系统在图形界面上的先天不足。而且，由于 Microsoft 的产品是要卖钱的，因此它为了维护自己的市场地位，会为客户提供比较好的服务，而免费 Unix 操作系统在这一点则比较困难，而且，它对用户的要求较高。在这里我不想给出一个确切的建议，如桌面操作系统应该选择什么，服务器操作系统应该选择 什么，一切要从实际情况出发。请大家注意这样的事实：专业人员维护的 Windows 服务器的安全性未必就 差，非专业人员维护的 * nix 服务器的安全性也肯定是无法接受。在抨击 Nimda 横行的同时，也请注意，Microsoft 早在 Nimda 出现前半年的时 候就已经推出了预防性的补丁；*nix 一样也有非常让人难堪的安全问题，同样的，他们也会及时推出补丁，因此，绝大多数服务器的安全问题是由于管理员的疏 忽造成的。选择操作系统之前，最好是先想好：什么样的配置对于您最有利，请考虑整体拥有成本 (TCO) 而不仅仅是操作系统的价格。目前很多 网站采用的、非常流行的 Windows 2000+Apache+Php+MySQL 组合实际上就同时牺牲了 IIS 开发周期短和 Apache+Unix 组合相对比较容易定制的优点，可谓竹篮打水 一场空。

这篇文章比较客观地对 FreeBSD 和 Linux 进行了对比，值得一读，所以我决定把它介绍给国内的读者。总之，Think different，决定选择什么东西之前，最好先尝试一番，特别是那些同类产品。

** 简介 **：即使是和 IT 不沾什么边的企业信息技术人士大概也都听说过 Linux。有些人可能用过，或正在使用它，原因五花八门，有的甚至只是为了看看那 些大肆吹嘘它的人到底说的是不是实话。然而，GNU/Linux 并不是可用的唯一一个 “free” 的 Unix 类操作系统。FreeBSD 和它的堂兄弟， OpenBSD 和 NetBSD 都是商业 UNIX 版本 ——Berkeley Software Distribution 免费的分支产品。这篇文章让您更多地了解 FreeBSD，也帮助您更轻松地进行潜在的迁移过程。

与 Linux 使用的 GPL 授权不同，BSD 家族的操作系统使用 BSD 风格的授权。用一句话来概括两种授权的不同就是，GPL 要求源代码的任何衍生物也是公有的，并且使用 GPL 授权，而 BSD 授权没有这个要求。

FreeBSD 和主要的 Linux 发行版本的工作方式有一定区别。这篇文章将告诉你我自己认为在把我的桌面操作系统由 Linux 切换到 FreeBSD 时的收获。

当然，肯定会有人坚持这样的观点：Linux 指的仅仅是操作系统的内核，而不是其他什么东西。在你的 Red Hat 或 Debian 匣子中，每天使用的应用程序是由相应的发行版本提供的。而 FreeBSD，则包括了操作系统内核和整个操作系统中的那些基本的应用程 序，例如复制、移动文件的命令等等。这一区别的结果是，Linux 由不同的发行版本，例如 Mandrake, SuSE, Debian 和 Slackware。任何使用过 Mandrake 和 Debian 的人都会告诉你这两套发行版本的世界有多么大的区别。相反，只有一个 FreeBSD，我的 FreeBSD 和你拥有的 FreeBSD 是完全一样的，只要他们的版本一样。

三中最主要的 Linux 发行版本，Red Hat、Mandrake 和 SuSE 使用了 RPM 安装包管理器。RPM 处理安装、升级、卸载，并检查安装在这些操作系统上的应用程序依赖关系。虽然在安装 程序之前检查依赖关系的错误，但 RPM 遗留了比我们期待的更多的问题。例如，它不能自动地下载它需要的其他 RPM。我知道至少 3 个项目试图解决这个问题， urpmi, Debian 的 apt-get，当然，附带说一句，只是一个 “仅 Debian 采用” 的特性，而且是一个 RPM 和 apt-get 的混血儿。所以，除非你打算 是用上面的方法，否则你将不得不手工寻找、下载所需要的 RPM。听起来很简单么？直到你用 RPM 安装 Gnome 或者升级 Xfree 的时候你才会知道事情有 多么严重。而且，即使你找到了正确的 RPM，如果他们是为 SuSE 设计的，而你运行的是 Red Hat，那么你的麻烦课就大了。

每一个 Linux 发行版本都存在一些差异，而它们之间最大的差异则在于文件系统的结构。我肯定绝大多数人都听说过 SuSE 把 KDE 放到 /opt，而 Red Hat 则放到 /usr 文件夹中。更糟糕的是，RPM 不能识别从源代码中编译得到的程序。所以，如果你拥有最新编译的程序，RPM 甚至无法知道他们的存在。

FreeBSD 使用 “包” 来安装、卸载和升级应用程序。‘pkg_add’命令被用于安装一个你手工下载到计算机的包。你也可以用‘-r’开关来让 它自动的从 Internet 获取，当然，也包括这个包所依赖的一切。不过，FreeBSD 包的真正美妙之处在于 “连接点”(Ports) 树。连接点树是 ——FreeBSD 包含的应用程序之间的继承关系。每一个文件夹都包含 Makefile，以及让特定应用程序能够在 FreeBSD 上正确运行所需要的补 丁。例如，如果我想安装 Apache web 服务器，我所需要做的只是 cd 到 /usr/ports/www/apache 文件夹，然后运行‘make && make install’，然后去小吃售卖机前。如果我拥有一台速度够快的电脑，同时拥有一个足够大方的 Internet 连接，当我回来的时候 Apache 源代码 的下载、补丁、编译和安装肯定都已经做完了。连接点树也能够处理 Apache 运行依赖的那些程序，无论我用连接点树安装、手工编译，还是通过安装已经编译 好的二进制包。连接点树能够通过 $PATH 找到它需要的东西。

Linux 和 FreeBSD 的另一个区别在于，对于 FreeBSD 而言，你安装的连接点或者包 99% 都会被放到 /usr/local，而在 Linux 上有时是 /usr，有时是 /opt。这可能只是一个很小的区别，但你至少可以知道你的程序安装到了 /usr/local，而不是扩散到了文件系 统的各个地方。

FreeBSD 系统使用 cvsup 来保持它是最新的。一旦你建立了‘sup-file’，cvsup 将会把你本地的系统和 cvsup 服务器上的进行 比较，并且下载那些修改过的东西。你可以用它来确保你的本地连接点树和 FreeBSD 源代码都是最新的。和 Linux 不同，Linux 通常只有内核被半正 规性的下载和变异。使用 cvsup，你可以很容易地下载整个 FreeBSD 操作系统的源代码。这样做的主要理由是，它使得 FreeBSD 从一个版本升级到 另一个的过程变的简单。Cvsup 之后，你可以用 make world 来编译整个操作系统，或者编译新的操作系统内核。这些都非常的简单。

处理分区的方式也有区别。Linux 将一个硬盘分为不同的分区，在这些分区中，有些又包括逻辑分去。我们常说的分区在 FreeBSD 中称为片断 (Slices)，没个片断中包括一个或多个 BSD 分去。BSD 分区在 /etc/fstab 中可以找到。

也许 Linux 和 FreeBSD 的下一个最大的区别就是操作系统设计的基本理念。Linux 强调最新的操作系统特性和驱动程序（例如不开放源代码的 nVidia 图形卡驱动程序）。FreeBSD 在这些方面比较保守。他们喜欢经过时间考验和测试过的东西，甚于最新特性。他们倾向于等待主要的 bug 被修 正。对于桌面操作系统来说，如果你使用最新的硬件，追求最新的驱动程序，或那些更酷的特性，保守是 FreeBSD 的一个毛病。然而在服务器中，你肯定希望 更加稳定的代码。另外，你会把一块价值 200 美元的显示卡放到你的不包括显示器的服务器上么？

另一个区别是默认安装的内容。如果你接受 SuSE 的默认安装选项，那么你至少会装上 1GB 的软件。而 FreeBSD 只是安装那些最基本的系统（注 意，我知道你会告诉 SuSE 仅仅安装‘基本系统’，但我说的是‘默认’安装）。他带给你那些最本质的东西，而你可以在以后通过连接点树安装 4000 多种应 用程序中的任何一个。几乎所有在 Linux 中运行的程序都已经被移植，并且能够正常运行于 FreeBSD，唯一的区别在于在 Linux 上，应用程序要么被 “默认安装”，要么，除非你用 Debian，你就必须手工下载它们。在 FreeBSD 上他们只是可选的，而且绝大多数过程已经被自动化了。另一些区别就 是，Linux 上默认的命令行外壳是 bash，而 FreeBSD 上则是 tcsh。

对于商业应用程序，如 Oracle 或 HP Openmail，FreeBSD 提供了一个 “Linux 兼容” 层。简而言之，它让 FreeBSD 能够以接近在 Linux 上运行的速度直接运行 Linux 的二进制应用。应用程序是否能够在 FreeBSD 上全速运行完全取决于它是否真的愿意在 Linux 上运行。兼容层比模拟更进一步。需要的 Linux 库被以 二进制形式安装在 BSD 系统中。当你试图运行 Linux 程序时，FreeBSD 识别它是 Linux 程序，并简单地指明它需要的 Linux 运行库的位置。同 时，FreeBSD 夜提供了商业 BSD、NetBSD、OpenBSD 和 SCO 的模拟。每种不同的操作系统获得不同的支持，其中最完善的是商业 BSD、 NetBSD 和 OpenBSD。

尽管 BSD 开发者更重视软件的品质和数量，但这并不意味着 FreeBSD 缺乏某些功能。预定于 2002 年 11 月推出的 FreeBSD 5.0 包括了更加精细的进程控制机制，这允许它更加有效地运行于最多 32 个处理器。版本 5.0 也将提供一个完整的 DEVDFS 设备文件系统。虽然这些在 Linux 上已经存在了一段时间，但你也许还没有听说过。DEVDFS 大体上是一个允许动态变化的设备文件系统。例如，如果你接入了一个 USB 键盘，它将 ‘魔术般地’加入到 /dev 文件夹。在日志文件系统方面，4.4 稳定版提供了‘soft updates’特性。尽管在技术上它也许不能北郊做日志文件系统，但它可以做得比你对日志文件系统的要求更好。

1998-1999 年. com 爆炸中，Linux 是真正的关键词。所有地方的电脑用户都听说了一种 * 免费 * 的，正在服务器领域和桌面领域挑战 Microsoft 地位的操作系统。即使在今天，Linux 的忠实用户仍然在增加。但是，很多人只是刚刚听说 FreeBSD。希望这篇文章能够帮助你对 FreeBSD 有一个初步的了解，并且把它作为满足你的需求的一种选择。在最后我想说的时，既然它们都是免费的，为什么不都试一试，看看谁更满足你的需要 呢？

***********************************************************************************************************************************************************

***********************************************************************************************************************************************************

## Linux

优点：充分发挥 PC 的功能，花样极多，玩起来很有趣，各方面的表现都不错。
缺点：太过自由，以致於发散掉了，维护方面比 FreeBSD 麻烦 (对一般人来说)。
-> 适合喜欢「玩 PC」，更甚於「玩 UNIX (Network)」的人。

## FreeBSD

优点：非常 UNIX、非常 Free、非常 BSD -- UNIX 的理想归宿！！
缺点：太过 UNIX，以致於玩下去很难收手 ^^;;
-> 适合喜欢 UNIX，有心好好经营 service 的人；也是 programmer 的理想 OS。

FreeBSD Core Team 并不是刻意忽略「入门的方便性」，只是人力有限，把主力投注在「UNIX 风味的主题」上。

FreeBSD 对硬体的需求实在也不会太严刻，对刚接触的人，建议使用「最一般化」的 硬体，像是: IDE (BigFoot)、ne2000 compatible 杂牌卡，S3Trito64，最烂的 14 寸 VGA，(atapi-cdrom)。

想说明的是，希望对 FreeBSD 有兴趣的人，别买些「太高档 (或者说奇怪)」的硬体， 到时候装不起来就骂 FreeBSD 怎麽这麽烂 ^^;;

可以想一下，到底想试试自己的 PC 能跑多少东西，还是真的有心进入 UNIX 的世界

## 为什麽要选择 FreeBSD ?!

嗯... 现在有许多免费的 i386 UNIX (在 386 以上 PC 执行的 UNIX)，例如 Linux、NetBSD、FreeBSD、OpenBSD、386BSD 等，究竟你要如何选择属於你的
UNIX ?

玩了三年多的 UNIX (一年半 Linux，两个月 NetBSD，两年 FreeBSD)
笔者只能以非正式的说法说说笔者的个人意见，希望这些意见不要引起争论
各个作业系统优缺点的大战。

Linux 是容易上手而且好玩的作业系统，也是现今最多人玩的，正因 为它太好装了，只要硬体没问题闭著眼睛都装的起来，因此 如果你是 i386 UNIX 的新手，这可说是你入门的最佳试金石。

NetBSD 支援 13 种硬体架构，这也是它的强处，算是 multi-platform
的典范。 也因此，i386 在里面只算是 13 种中的一种，自然无法取得全力的发展，再加上其 core team 比较不活跃，所以在 i386 上的硬体支援并不是很好。

OpenBSD 源自 NetBSD，刚出来半年左右，专门把 NetBSD 跟 FreeBSD 的 新功能跟修正加在一起，算是 NetBSD+FreeBSD 的混血儿，由於 其 core team 人数少，加上程式码很少是自己开发的，因此现在
前景还不明朗。

FreeBSD 跟 NetBSD 一样都是基於 4.4 BSD-lite，但是 FreeBSD 现在只支援 i386，所以在 PC 上来说要比 NetBSD/OpenBSD 好太多了， 在从前 NetBSD 跟 FreeBSD 的 core team 是一起的，後来分家了。 FreeBSD 具有一般 BSD 系统的稳定，又从其他作业系统学习了许 多优点，再加上自己开发的各种新功能，时时改进演算法以增加 执行效率，现在已是免费 BSD 系列中效率最好的，最主要是因为 core team 活跃又乐於接受使用者的意见并改进。

### 什麽是 core team ?

core team 是一个专门对原始程式码做发展跟维护的组织，Linux 没有 core team，NetBSD/OpenBSD/FreeBSD 有。有 core team 的优点是
原始程式码会有一致性，会有组织的被更新，但是整个 OS 的活力也操在 core team 的手中，这就是 NetBSD 在笔者眼中无法兴盛的原因。而没有 core team (如 Linux)，好处是全世界每个人都可以发表自己的修正 (patch) 不须经由 core team 的审核，但缺点是 source code 杂乱无章且可能会 不同步。所以 Linux 在更新东东的时候，必须由使用者自己注意 kernel、 gcc、library、net-tool、modules、甚至各种 kernel patch 版本的一致性。
(或许在 RetHat Linux 已经稍微好一点了)
而这些可怜的情形在 FreeBSD 身上都不会发生。

### 要选择怎样的 OS 必须看你自己的需求及能力，还有周遭玩的人多不多， 多装几种，多装几次，自己感觉一下才是真的！
(其实只要不怕 format 硬碟，吃饱撑著，装什麽东西、装几次都好说嘛)

#### 稳定性
一个作业系统最重要的就是稳定性，比方说能连续开机多久，能忍受 多少系统负荷，网路不稳时会不会当掉，网路负荷太大时网路会不会 死掉，笔者个人觉得 FreeBSD > Linux。
尤其许多研究已经提出，Linux 在高系统负荷下的表现相当不好，而
FreeBSD 却不会。要知道世界上最大的 ftp site - http://wcarchive.cdrom.com 是一台跑著

FreeBSD 的 Pentium pro 机器 (P6-150，512MB RAM，72GB HDs online
more than 1200 ftp users allowed)

注 : http://wcarchive.cdrom.com = ftp.cdrom.com

#### 网路
争夺封包 (packet) 的速度，除了网路卡好坏之外，最重要的还是作业系统跟 驱动程式，使用一样的网路卡 FreeBSD > Linux >>> DOS+NCSA. 而且
FreeBSD 在 RPC 及 NFS 上都比 Linux 来的稳定及快速。毕竟 BSD 在网路
这方面是始祖.

#### 移植软体的难易程度
现今一般的软体大多是为 BSD 写的，所以一般软体在 BSD 上会比在 SYSV 上容易编译。而 FreeBSD 是 4.4BSD based，Linux 是 SYSV 加 上 BSD-extension，所以在 Linux 上编译东西有时是个梦靥 (不是很 SYSV 也不是很 Posix 也不是很 BSD)。不过现在越来越多的软体会注 意到 Linux，因为 Linux 使用者太多了。
FreeBSD 有收集数百种软体的 ports，只要打个 make 就可以轻松编译，不然也有编译好的 binary 可以直接安装使用。

#### 硬体支援
Linux 支援最多种的硬体，NetBSD 最少，而 FreeBSD 夹在中间正急起
直追中，而且许多 FreeBSD 的 driver 都写的相当棒，反而後来被
移植到 NetBSD 跟 Linux。

#### Merged VM/buffer cache
Linux 的磁碟 I/O 速度是一流的，因为一来 Linux 的 ext2fs 是 async-mount 的，写入资料时不须一直更新 meta-data，最主要还是 Linux 会把目前没用到的记忆体尽量拿来做 I/O buffer。一般传统 BSD (如 SunOS，NetBSD) 都只有固定大小的 buffer，而 FreeBSD 自己发展出类似 Linux 的 Merged VM/buffer cache，大大提高了 I/O 时的效率以及记忆体利用率，而且现在 FreeBSD 已支援 async-mount， 使得 FreeBSD 的档案系统已经跟 Linux 不相上下，甚至更胜一筹。

#### tty 限制
现在 Linux 要用超过 64 个 tty 除了必须更改应用程式的原始程式码， 还必须做 kernel patch，而 FreeBSD 内定支援 tty [pqrsPQRS][0-9a-v] 总共 256 个 tty，只要到 /dev 下用 MAKEDEV 把 tty 建出来，在 /etc/ttys 加入新的 tty 设定，再到 kernel config file 中把 pty 的数目打入 256 就好了，要使用超过 256 tty 也相当容易修改。

#### 完整原始程式码取得
一般人使用的 Slackware 版 Linux 是由 Slackware 公司整理，所 以一般人要取得完整原始程式码必须自己东抓西抓，这也是 Linux 在 NCTUCCCA 的 mirror 量这麽大的缘故。但往往 Linux 使用者找不到 自己须要的原始程式码，如果没有那些整理 Linux packages 的公司， 以及帮忙 Linux 发展系统工具及函式库的人，Linux 充其量算是只有 Linus 写的 kernel 而已，不过最大的问题还是各家写出来的东东 一致性的问题。不过新出来的 RedHat 已经提供一个简单的软体同步 与更新的方法 - RPM，也算是稍微抒解这一类问题的严重性。 而 FreeBSD 提供完整的系统原始程式码，从 `/bin/sbin /usr/bin /usr/sbin/usr/lib ...` 甚至 `/etc/usr/share/FAQ` 都在里面， 让你可以很容易的更改自己想要的东东，要更新系统时也可以抓取 最新的 source 打个 make world 就成了 (当然也可以用 core team 做好的 binary)，它甚至会自动检查各目录的权限是否正确。 简单一句，就是非常的有组织！利用 binary 来升级只要不到一小时就可以完成，甚至有写好的 script 可以使用。

#### 目录档案组织化
FreeBSD 根据 4.4BSD 规范，什麽档案应该在那里，应该是什麽权限，编译时应该连结 (link) 成 static 或 dynamic，都非常的严谨，该有的 manpages 绝对不会少。不像 Linux，写 kernel 一个人、写 library 另一个，写 manpages 又另一个、整理 utility 又另一个，各自为政不同步，常常档案到处乱放或是重覆，manpages 不完整，许多目录档案为了新旧版本的相容性而 link 来 link 去。

#### 系统安全
FreeBSD 使用 shadow password，支援 secure NFS，不像 Linux 要 自己安装 shadow password，将来编译 ftpd，sudo 时又得改来改去。

因为 USA 版的 DES 禁止输出到美加以外地区，FreeBSD 为了全世界广大的使用者，在密码系统上内定使用 MD5 编码，它比 DES 来的安全，如果你不跟 SunOS 类的 YP server 跑 NIS，那你是不须要安装 DES 的。如果你要使用 DES，你可以安装可以自由流动的 DES 版本 (非 USA 版)，在 / usr/share/FAQ/Text/FreeBSD.FAQ 中有提及那里可以取得，或是到台湾任何一个 FTP 站取得。

此外，FreeBSD 的使用者登入控制，以及档案安全层级保护都比其他作业系统来的好 (kernel secure level)。FreeBSD 的 core team 会注意 source code 跟 security 的同步性，一有新的问题或 sendmail 漏洞，就会立刻更新程式码，已达到最佳的系统安全。

#### core team 活跃
FreeBSD 的 core team 非常活跃而且谦虚，带动整个 FreeBSD 迅速发展，每天都有新的 patch 出来，让使用者以 sup/ctm 来定时自动更新原始程式码。

#### 4.4BSD-lite based
由於 FreeBSD 是基於 4.4BSD-lite 的，因此带来了许多 BSD 的好处，像网路速度稳定、容易移植软体、安全快速等。

#### 从 Linux 而来的优点
FreeBSD 正在把 Linux 的 dosemu 移植过来，甚至可以直接执行 linux 的 binary (linux emulator)，还有移植 Linux 支援的一些驱动程式。

#### 支援 LKM
FreeBSD 支援 Loadable kernel module，也就是说许多驱动程式在编译 kernel 时可以不必做进去，一旦你要用到时，kernel 会自动从 /lkm/*.o 载入该 driver，这样可以提高弹性并减小 kernel 使用的记忆体空间。未来 FreeBSD 会朝向 LKM device 迈进，就像 Solaris 一样不需编译 kernel。

#### 直接执行 gzip 的程式
FreeBSD 可以直接执行 gzip 的程式，如果你把所有的执行档都 gzip 起来，不就等於用 stacker/doublespace 一样了？！

13. 线上监控
kernel 支援 tty snoop，可以监控线上使用者 (不像 linux 那个半调子 ttysnoop，会导致许多问题)。

#### 众多档案系统
支援 MFS (Memory File System)，类似 SunOS tmpfs 的东东，还有许多 4.4BSD 定义的档案系统，如 LFS、NULLFS、PORTALFS、UMAPFS、UNIONFS。

#### Interleaved swap
当你有一个以上的 swap 装置时，会同时使用以增加速度 (尤其是使用 SCSI 装置时)，而不是像 Linux 一个接著一个使用。

#### 新的 slice 观念
新的 slice 观念使得 FreeBSD 对其他 OS 的 partition 相容性比传统的 BSD 好很多，在安装上也较为容易。

#### Binary 相容性
FreeBSD 可以执行 NetBSD-static，BSDI-static，Linux-a.out/elf，SCO-static 等等的 binary code，增加不少相容性。

#### ccd (软体 RAID)
Concatenated disk (ccd) 驱动程式能让你拥有 Strip、Mirror，甚至 Parity 等 RAID card 才有的功能。

#### 多国语言的支援
FreeBSD 的 localization 是所有免费作业系统中做的最好的，甚至已经有了亚洲语系 (中文、日文) 的安装介面。

#### 有组织的原始程式码
FreeBSD 的程式开发者在撰写程式码的时候，会去参考各种 RFC 规范以及 新的理论文献，因此 FreeBSD 的程式码有条不紊、层次鲜明；反观 Linux 常常为了急就章而走捷径写出来的东西，到最後开发新功能时又必须改来改去。

不过随著时间的发展，Linux、*BSD 都会进步，对於免费的作业系统能越来越好自然是乐见其成的。

一般而言，如果你须要一台稳定快速的 Internet Server，FreeBSD 是你绝对 的选择；如果你是个人使用或只是想学习 UNIX，Linux 跟 FreeBSD 都是很好 的试金石。

Linux 浮上台面已经四年了，而 FreeBSD 以短短的两年时间就拥有了众多的 使用者人口 (尤其是伺服器，以及程式开发者)，高手的选择必有他的道理。用过 FreeBSD 才知道，『PC 不只是很便宜的工作站』

但是，Linux 的优点是『好玩』，而且随著 kernel 日渐更新，很多东西也 越来越稳定。我们系上从两年前开始就用 Linux 当 mail, acounts, ftp, gopher, terminal, ppp, slip, BBS servers, 最近又加入 WWW server，服务几百位师生。 目前系上已经有好几台 Linux PC 一起运作，其中包含 NFS，与 WinNT，Win95 的连线与资源共享 (by SAMBA packages)，我们也在测试用其中一台摹拟 Novell Server.

我们的同时上线人数一般不会超过 100 人，用 Linux 来应付绰绰有馀。如果你想开 的是一次几百人上线的 BBS 大站，那可能 FreeBSD 会比较适合。不过话说 回来，能开这种大站的单位都很有钱，大都拿 SUN 或其他 workstation 级的来 run。

Linux 另一个优点是全球的 Linux users 远超过 FreeBSD，这使得 Linux 上面 新的软体跟硬体 drivers 更新数目及速度远超过 FreeBSD。例如，DOSEMU 可以 摹拟 DOS，WINE 可以摹拟 Windows 3.1，smbfs 可以将 Win95 或 WinNT 上的 partition 拿来用：这些在 FreeBSD 上面都还在发展中，甚至没有。新电脑 硬体 drivers 的更新也是如此，几乎任何新的硬体都会有 Linux 迷很快地帮大家 写好 drivers。你如果用过 FreeBSD 跟 Linux，你就会发现 FreeBSD 目前对 硬体要求仍然比较『严格』(其实是还没有人写 drivers)。我用的 scanner， 还有 voice modem，都已经有 Linux 迷写好程式，让我可以在 Linux 上 scan 以及有语音信箱。

我个人的建议是，如果你是个人使用，或者网路同时上线人数不超过一百人以上， Linux 的确是好玩又实用，而且新的硬体很快地几乎都可以在 Linux 上使用。 如果你要架的是几百人上站的机器，又没钱买 workstation，那 FreeBSD 在 网路壅塞时的 performance 的确不错。如果是个人要『玩』，我并不建议 FreeBSD，那会使你觉得提不起兴致 (纯属个人观点)。

在 csie gopher 中有关 Linux 与 FreeBSD 的比较中，有一项是 FreeBSD 上 software porting 比较 easy。但是这个 comment 随著 Linux users 群日渐庞大， 我觉得已经有些改变：现在在 Linux 很多东西根本用不著 porting，因为很多 软体根本就是 Linux fans 专门为 Linux 设计写出来的，反而要用这些东西 需要额外费心去修改以便能在 FreeBSD 上使用。DOSEMU，smbfs 即是其中几个例子。据最近的 newsgroups，FreeBSD core team 有五十多人，但是 Linux fans 散布在全球各地的 programmers 其数量根本无法计算。有心的话， 比较下 Linux 跟 FreeBSD announce newsgroups 就可知一二。

所以，我并不是很赞同一个 UNIX 的新手去玩 FreeBSD。但是，假如有人已经玩过 Linux ，或者在其他工作站级机器有过简单管理经验，那他们会发现 FreeBSD 极易入手。玩过 FreeBSD 的人一定知道光要新增 partitions 就已经是一件麻烦的事。堂堂一个 FreeBSD 的 fdisk 介面连 MDOS 的都不如， 可见 FreeBSD core team 之目标不在一般连 ls, cp, tar 都不懂的 newbie。 另外一个动机是假如你必须要架一台超稳定的 Internet server，那 FreeBSD 是目前的 best choice。

其实呢，如果有心要玩，大可弄个大点的硬碟，同时装上两个系统，一定可以如鱼得水。我的 office 中同时有一台 FreeBSD，一台 Linux，各做各的事， 也是很快乐。。。。

就目前使用者能观察到的来看，一般相信 linux 的 data-path-consumed process 的执行速度，是众多 x86 作业系统中最快的；而 high load 下的网路则 令人不能感到非常满意。虽然 linux 第二版後网路 部分有了很大的改善，据 Linus 本人的说法，linux 在传 single package 已比 FreeBSD 还优胜，但作为 NFS 或 high load netserver 还是显得略有不顺 (所谓 "不顺" 与 "不稳" 无关). 毕竟，考查 linux 的发展历史，的确是先在 x86-embeded scheduler, fs, 等核心 process 执行部分，最後才加进网路部分，process 执行最佳而网路稍逊乃是合理 的结果.

一般建议如果机器用来执行程式 (如跑 project) 为主，跑各式怪模怪样的小程式及 server, 或有非正统硬体者 使用 linux 可能较佳.