---
layout: post
title: Linux Shell Pipe
categories: [linux, shell, Pipe]
description: Linux Shell Pipe
keywords: linux, shell, Pipe
---

## 命令执行顺序控制
通常情况下，我们在终端只能执行一条命令，然后按下回车执行，那么如何执行多条命令呢？

1. 顺序执行多条命令：command1;command2;command3;  
    简单的顺序指令可以通过 ; 来实现
2. 有条件的执行多条命令：which command1 && command2 || command3  
    && : 如果前一条命令执行成功则执行下一条命令，如果 command1 执行成功（返回 0）, 则执行 command2  
    || : 与 && 命令相反，执行不成功时执行这个命令
3. $?: 存储上一次命令的返回结果

``` shell
栗子：
$ which git>/dev/null && git --help  // 如果存在 git 命令，执行 git --help 命令
$ echo $? 
```

## 管道命令
> 管道是一种通信机制，通常用于进程间的通信（也可通过 socket 进行网络通信），它表现出来的形式将前面每一个进程的输出（stdout）直接作为下一个进程的输入（stdin）。

管道命令使用 | 作为界定符号，管道命令与上面说的连续执行命令不一样。

* 管道命令仅能处理 standard output, 对于 standard error output 会予以忽略。  
  less,more,head,tail... 都是可以接受 standard input 的命令，所以他们是管道命令  
  ls,cp,mv 并不会接受 standard input 的命令，所以他们就不是管道命令了。

* 管道命令必须要能够接受来自前一个命令的数据成为 standard input 继续处理才行。

### 选取命令 「cut」「grep」

* cut: 从某一行信息中取出某部分我们想要的信息。
> cut -d '分隔字符' -f field // 用于分隔字符  
> cut -c 字符范围  
> [参数说明]  
> -d : 后面接分隔字符, 通常与 -f 一起使用  
> -f : 根据 - d 将信息分隔成数段，-f 后接数字 表示取出第几段  
> -c : 以字符为单位取出固定字符区间的信息  
``` shell
栗子 1：
打印 / etc/passwd 文件中以: 为分隔符的第 1 个字段和第 6 个字段分别表示用户名和家目录
[root@izuf6i29flb2df231kt91hz /]# cat etc/passwd | cut -d ':' -f 1,6
root:/root
bin:/bin
daemon:/sbin
adm:/var/adm
lp:/var/spool/lpd
```

``` shell
栗子 2：
打印 / etc/passwd 文件中每一行的前 10 个字符：
[root@izuf6i29flb2df231kt91hz /]# cat /etc/passwd | cut -c 1-10
root:x:0:0
bin:x:1:1:
daemon:x:2
adm:x:3:4:
lp:x:4:7:l
```
**ps:cut 在处理多空格相连的数据时，比较吃力。**

* grep: 分析一行信息，如果其中有我们需要的信息，就将该行拿出来
> grep [-acinv] [--color=auto] '查找字符串' filename  
> [参数]  
> -a : 将 binary 文件以 text 文件的方式查找数据  
> -c : 计算找到 '查找字符串'的次数  
> -i : 忽略大小写的不同  
> -n : 输出行号  
> -v : 反向选择，显示没有查找内容的行  
> --color=auto : 将找到的关键字部分加上颜色显示  

``` shell
栗子 3：
取出含有 fanco 的 / etc/passwd 文件的行
[root@izuf6i29flb2df231kt91hz /]# cat etc/passwd | grep -n -c 'fanco'
1
[root@izuf6i29flb2df231kt91hz /]# cat etc/passwd | grep -n 'fanco'
23:fanco:x:1001:1001::/home/fanco:/bin/bash
[root@izuf6i29flb2df231kt91hz /]# cat etc/passwd | grep -n -v 'fanco'
1:root:x:0:0:root:/root:/bin/bash
2:bin:x:1:1:bin:/bin:/sbin/nologin
3:daemon:x:2:2:daemon:/sbin:/sbin/nologin
4:adm:x:3:4:adm:/var/adm:/sbin/nologin
5:lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
...
```

### 排序命令 「sort」「uniq」「wc」
* sort
> sort [-fbMnrtuk] [file or stdin]  
> [参数]  
> -f ：忽略大小写的差异，例如 A 与 a 视为编码相同  
> -b ：忽略最前面的空格部分  
> -M ：以月份的名字来排序，例如 JAN, DEC 等等的排序方法  
> -n ：使用『纯数字』进行排序默认是以文字型态来排序的)  
> -r ：反向排序  
> -u ：就是 uniq ，相同的资料中，仅出现一行代表  
> -t ：分隔符号，预设是用 [tab] 键来分隔  
> -k ：以那个区间 (field) 来进行排序的意思  

``` shell
栗子 4：
对 / etc/passwd 的账号进行排序
[root@izuf6i29flb2df231kt91hz /]# cat /etc/passwd | sort
adm:x:3:4:adm:/var/adm:/sbin/nologin
bin:x:1:1:bin:/bin:/sbin/nologin
chrony:x:998:996::/var/lib/chrony:/sbin/nologin
...
通过 / etc/passwd 第 5 列来进行排序
[root@izuf6i29flb2df231kt91hz /]# cat etc/passwd | sort -t ':' -k 3
root:x:0:0:root:/root:/bin/bash
fanco:x:1001:1001::/home/fanco:/bin/bash
operator:x:11:0:operator:/root:/sbin/nologin
bin:x:1:1:bin:/bin:/sbin/nologin
games:x:12:100:games:/usr/games:/sbin/nologin
ftp:x:14:50:FTP User:/var/ftp:/sbin/nologin
这里排序还是按照文字进行排序的，切换成数字排序
[root@izuf6i29flb2df231kt91hz /]# cat etc/passwd | sort -t ':' -k 3 -n
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
adm:x:3:4:adm:/var/adm:/sbin/nologin
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
```

* uniq
> uniq [-ic]  
> [参数]  
> -i ：忽略大小写的不同  
> -c ：进行计数  

``` shell
栗子 5
使用 last 取出历史登录信息的账号，排序，去重
[root@izuf6i29flb2df231kt91hz /]# last | cut -d ' ' -f 1 | sort | uniq -c
      1 
      7 reboot
     19 root
      1 wtmp
```

* wc
> wc [-lwm]  
> [参数]  
> -l ：仅列出行  
> -w ：仅列出多少字 (英文单字)  
> -m ：多少字符  

``` shell
栗子 6
查看 etc/passwd 中有多少账号
[root@izuf6i29flb2df231kt91hz /]# cat /etc/passwd | wc -l
23
计算最近登录系统的人次
[root@izuf6i29flb2df231kt91hz /]# last | grep [a-zA-Z] | grep -v 'wtmp' | wc -l
2
查看某个文件的行数 字数 字符数
[root@izuf6i29flb2df231kt91hz /]# cat etc/passwd | wc
     23      32     997
```

### 双向重定向命令 「tee」
* tee：在数据流的处理过程中将某段信息保存下来，使其既能输出到屏幕又能保存到某一个文件中。

![linux_shell_1](https://cdn.jsdelivr.net/gh/Lewinz/lewinz.github.io@master/images/posts/linux_shell_1.png)

> tee [-a] file  
> [参数]  
> -a : 以累加的方式，将数据加入 file 中  

``` shell
栗子 7
查询最近用户登录情况，并将其保存到文件中
[root@izuf6i29flb2df231kt91hz /]# last | tee info | cut -d ' ' -f 1
root
...
[root@izuf6i29flb2df231kt91hz /]# less info
root     pts/0        112.28.181.159   Sun Jul  1 14:28   still logged in   
root     pts/0        112.28.181.159   Sun Jul  1 14:24 - 14:27  (00:03)    
root     pts/0        112.28.181.159   Sun Jul  1 13:19 - 14:24  (01:04)    
root     tty1                          Sun Jul  1 12:46   still logged in  
```
如果 tee 后接的文件已存在，内容会被覆盖掉，加上 -a 参数则会累加

### 字符转换命令 「tr」「col」「join」「paste」「expand」
* tr：用来删除一段信息当中的文字，或者进行文字信息得替换
> tr [-ds] set  
> [参数]  
> -d : 删除信息当中的 set1 这个字符串  
> -s : 替换掉重复的字符  

``` shell
栗子 8
将上一步生成的 info 文件删除掉所有的 root
删除前
[root@izuf6i29flb2df231kt91hz /]# cat info
root     pts/0        112.28.180.86    Thu May 10 18:01 - 18:12  (00:11)    
reboot   system boot  3.10.0-693.2.2.e Fri May 11 02:00 - 16:31 (51+14:30)  
 删除后
[root@izuf6i29flb2df231kt91hz /]# cat info | tr -d 'root'  
     ps/0        112.28.180.86    Thu May 10 18:01 - 18:12  (00:11)    
eb   sysem b  3.10.0-693.2.2.e Fi May 11 02:00 - 16:31 (51+14:30)  

删除时并不是只删除连续的字符，reboot 也被删除掉了 root 部分
```
``` shell
除去 dos 文件留下来的 ^M 符号
$ cat /root/passwd | tr -d '\r' > /root/passwd.linux
^M 可以用 \ r 替代
```

* col
> col [-xb]  
> [参数]  
> -x ： 将 tab 键换成对等的空格键  
> -b : 在文字内有反斜杠 (/) 时，仅保留反斜杠最后接的那个字符  
``` shell
栗子 9
将上图中的 ^I 换成空格键
[root@izuf6i29flb2df231kt91hz /]# cat info | col -x | cat -A | more
        root     pts/0        112.28.181.159   Sun Jul  1 14:28   still logged in$
col 经常被用于将 man page 转存为纯文本文件

join: 主要讲两个文件有相同数据的一行，相同字段放在前面
join [-ti12] file1 file2
[参数]
-t : join 默认以空格符分隔数据，并且对比第一个字段的数据 , 如果两个文件相同，则将两条数据连成一行
-i : 忽略大小写的差异
-1 : 说明第一个文件通过那个字段来进行分析
-2 : 说明第二个文件通过那个字段来分析
```
``` shell
栗子 10
将 / etc/passwd 与  /etc/shadow 相关数据整合成一列
[root@izuf6i29flb2df231kt91hz /]# head -3 /etc/passwd /etc/shadow
==> /etc/passwd <==
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin

==> /etc/shadow <==
root:$6$RNGEziM7$2e/EJd3hThS8TMqHSgDIfeDf7dJUG1dbJ0ik1goybGYmLGZL.sHNv1Ltb4.1HUksxTI0Cs3PJw5g/YirSImKg1:17643:0:99999:7:::
bin:*:17110:0:99999:7:::
daemon:*:17110:0:99999:7:::
[root@izuf6i29flb2df231kt91hz /]# join -t ':' /etc/passwd /etc/shadow
root:x:0:0:root:/root:/bin/bash:$6$RNGEziM7$2e/EJd3hThS8TMqHSgDIfeDf7dJUG1dbJ0ik1goybGYmLGZL.sHNv1Ltb4.1HUksxTI0Cs3PJw5g/YirSImKg1:17643:0:99999:7:::
bin:x:1:1:bin:/bin:/sbin/nologin:*:17110:0:99999:7:::
daemon:x:2:2:daemon:/sbin:/sbin/nologin:*:17110:0:99999:7:::

将 etc/passwd 按：分隔的第 4 个字段 与 etc/group 的第 3 个字段 比较，如果相同，则将他两同行数据放在一起
[root@izuf6i29flb2df231kt91hz /]# join -t ':' -1 4 /etc/passwd -2 3 /etc/group
0:root:x:0:root:/root:/bin/bash:root:x:
1:bin:x:1:bin:/bin:/sbin/nologin:bin:x:
2:daemon:x:2:daemon:/sbin:/sbin/nologin:daemon:x:
4:adm:x:3:adm:/var/adm:/sbin/nologin:adm:x:
join: /etc/passwd:6: is not sorted: sync:x:5:0:sync:/sbin:/bin/sync
7:lp:x:4:lp:/var/spool/lpd:/sbin/nologin:lp:x:
```

* paste: 直接将两个文件两行贴在一起，中间以 [tab] 键隔开
> paste [-d] file1 file2  
> [ 参数]  
> -d : 后面可以接分隔字符，默认以 [tab] 来分隔的  
> - : 如果 file 部分写成 -，表示接受 standard input 数据的意思  

``` shell
栗子 11
[root@izuf6i29flb2df231kt91hz /]# paste info info2
    root     pts/0        112.28.181.159   Sun Jul  1 14:28   still logged in       root     pts/0        112.28.181.159   Sun Jul  1 14:28   still logged in   
root     pts/0        112.28.181.159   Sun Jul  1 14:24 - 14:27  (00:03)        root     pts/0        112.28.181.159   Sun Jul  1 14:24 - 14:27  (00:03)    
root     pts/0        112.28.181.159   Sun Jul  1 13:19 - 14:24  (01:04)        root     pts/0        112.28.181.159   Sun Jul  1 13:19 - 14:24  (01:04) 
```

* expand: 把 tab 键转为空格键
> expand [-t] file  
> [参数]  
> -t : 后面接数字，一般，一个 tab 可以用 8 个空格代替，可以自行定义代表几个空格
``` shell
栗子 12
[root@izuf6i29flb2df231kt91hz /]# cat info | expand -3 info
   root     pts/0        112.28.181.159   Sun Jul  1 14:28   still logged in   
root     pts/0        112.28.181.159   Sun Jul  1 14:24 - 14:27  (00:03)    
root     pts/0        112.28.181.159   Sun Jul  1 13:19 - 14:24  (01:04)    
root     tty1                          Sun Jul  1 12:46   still logged in
```

* 切割命令：split, 顾名思义，讲一个大文件依据文件大小或行数切割成为小文件
> split [-bl] file prefix  
> [参数]  
> -b : 后面可接欲切割文件的大小，可加单位，例如 b,k,m 等  
> -l : 以行数来进行切割  
> PREFIX : 代表前导符，可作为切割文件的前导文字  
``` shell
栗子 13
$ split -b 300K /etc/passwd
将 ls -al 输出文件  按 10 行分成一个新的文件
[root@izuf6i29flb2df231kt91hz /]# ls -al / | split -l 10 - lsrrot
[root@izuf6i29flb2df231kt91hz /]# ls 
b    boot  dev  home  info2  lib64       lsrrotaa  lsrrotac  mnt  opt   root  sbin  sys  usr
bin  c     etc  info  lib    lost+found  lsrrotab  media     n    proc  run   srv   tmp  var
[root@izuf6i29flb2df231kt91hz /]# cd /
[root@izuf6i29flb2df231kt91hz /]# ls
b    boot  dev  home  info2  lib64       lsrrotaa  lsrrotac  mnt  opt   root  sbin  sys  usr
bin  c     etc  info  lib    lost+found  lsrrotab  media     n    proc  run   srv   tmp  var
[root@izuf6i29flb2df231kt91hz /]# wc -l lsrrot*
  10 lsrrotaa
  10 lsrrotab
   9 lsrrotac
  29 total
```