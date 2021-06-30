---
layout: wiki
title: Mac各类配置记录
categories: [Mac, 配置]
description: Mac各类配置记录
keywords: Mac, 配置
---

## mac提示文件已损坏 

终端执行`sudo spctl --master-disable`  

安全性与隐私->通用->设置任何来源

终端输入命令  
`sudo xattr -r -d com.apple.quarantine `  
先别回车，将引用程序中软件拖入终端，回车执行

## oh-my-zsh自定义主题
配置详细文件因makedown格式受限  
存放在github仓库/config/oh-my-zsh-config.txt中

## mac 软件列表
**命令行**  
<https://www.iterm2.com/>  

**shell**  
<https://ohmyz.sh/>  

**软件包管理**  
<https://brew.sh/>  

**MySQL 客户端**  
<https://www.sequelpro.com/>  

**神器alfred**
<https://www.alfredapp.com/>  

**神器tldr**
`brew tap tldr-pages/tldr && brew install tldr`

## mac使用item2进行ssh连接
使用命令连接
`ssh -p22 root@47.117.136.250`

### 安装lrzsz (使用rz/sz)

`brew install lrzsz`

### 新增配置文件

``` sh
touch /usr/local/bin/iterm2-recv-zmodem.sh
touch /usr/local/bin/iterm2-send-zmodem.sh
```

修改 `vi /usr/local/bin/iterm2-recv-zmodem.sh`
```sh
#!/bin/bash
# Author: Matt Mastracci (matthew@mastracci.com)
# AppleScript from http://stackoverflow.com/questions/4309087/cancel-button-on-osascript-in-a-bash-script
# licensed under cc-wiki with attribution required 
# Remainder of script public domain

osascript -e 'tell application "iTerm2" to version' > /dev/null 2>&1 && NAME=iTerm2 || NAME=iTerm
if [[ $NAME = "iTerm" ]]; then
    FILE=`osascript -e 'tell application "iTerm" to activate' -e 'tell application "iTerm" to set thefile to choose folder with prompt "Choose a folder to place received files in"' -e "do shell script (\"echo \"&(quoted form of POSIX path of thefile as Unicode text)&\"\")"`
else
    FILE=`osascript -e 'tell application "iTerm2" to activate' -e 'tell application "iTerm2" to set thefile to choose folder with prompt "Choose a folder to place received files in"' -e "do shell script (\"echo \"&(quoted form of POSIX path of thefile as Unicode text)&\"\")"`
fi

if [[ $FILE = "" ]]; then
    echo Cancelled.
    # Send ZModem cancel
    echo -e \\x18\\x18\\x18\\x18\\x18
    sleep 1
    echo
    echo \# Cancelled transfer
else
    cd "$FILE"
    /usr/local/bin/rz -E -e -b
    sleep 1
    echo
    echo
    echo \# Sent \-\> $FILE
fi
```

修改 `vi /usr/local/bin/iterm2-send-zmodem.sh`
```sh
# Author: Matt Mastracci (matthew@mastracci.com)
# AppleScript from http://stackoverflow.com/questions/4309087/cancel-button-on-osascript-in-a-bash-script
# licensed under cc-wiki with attribution required 
# Remainder of script public domain

osascript -e 'tell application "iTerm2" to version' > /dev/null 2>&1 && NAME=iTerm2 || NAME=iTerm
if [[ $NAME = "iTerm" ]]; then
    FILE=`osascript -e 'tell application "iTerm" to activate' -e 'tell application "iTerm" to set thefile to choose file with prompt "Choose a file to send"' -e "do shell script (\"echo \"&(quoted form of POSIX path of thefile as Unicode text)&\"\")"`
else
    FILE=`osascript -e 'tell application "iTerm2" to activate' -e 'tell application "iTerm2" to set thefile to choose file with prompt "Choose a file to send"' -e "do shell script (\"echo \"&(quoted form of POSIX path of thefile as Unicode text)&\"\")"`
fi
if [[ $FILE = "" ]]; then
    echo Cancelled.
    # Send ZModem cancel
    echo -e \\x18\\x18\\x18\\x18\\x18
    sleep 1
    echo
    echo \# Cancelled transfer
else
    /usr/local/bin/sz "$FILE" -e -b
    sleep 1
    echo
    echo \# Received $FILE
fi
```

给两个文件授权
```sh
chmod 777 /usr/local/bin/iterm2-*
```

### iTerm2 配置添加rz sz 功能

点击 iTerm2 的设置界面 Perference-> Profiles -> Default -> Advanced -> Triggers 的 Edit 按钮

新增两条配置
``` txt
Regular expression: rz waiting to receive.\*\*B0100
Action: Run Silent Coprocess
Parameters: /usr/local/bin/iterm2-send-zmodem.sh

Regular expression: \*\*B00000000000000
Action: Run Silent Coprocess
Parameters: /usr/local/bin/iterm2-recv-zmodem.sh
```

**注意**
使用item2远程ssh连接服务器后，同样需要在远程服务器上安装lrzsz，否则无法上传下载文件
`yum install lrzsz`