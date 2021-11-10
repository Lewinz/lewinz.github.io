---
layout: wiki
title: Mac 各类配置记录
categories: [Mac, 配置 ]
description: Mac 各类配置记录
keywords: Mac, 配置
---

## mac 提示文件已损坏 

终端执行`sudo spctl --master-disable`  

安全性与隐私-> 通用-> 设置任何来源

终端输入命令  
`sudo xattr -r -d com.apple.quarantine `  
先别回车，将引用程序中软件拖入终端，回车执行

## oh-my-zsh 自定义主题
配置详细文件因 makedown 格式受限  
存放在 github 仓库/config/oh-my-zsh-config.txt 中

## mac 使用 item2 进行 ssh 连接
使用命令连接
`ssh -p22 root@47.117.136.250`

### 安装 lrzsz (使用 rz/sz)

`brew install lrzsz`

### 新增配置文件

``` shell
touch /usr/local/bin/iterm2-recv-zmodem.sh
touch /usr/local/bin/iterm2-send-zmodem.sh
```

修改 `vi /usr/local/bin/iterm2-recv-zmodem.sh`
``` shell
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
``` shell
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
``` shell
chmod 777 /usr/local/bin/iterm2-*
```

### iTerm2 配置添加 rz sz 功能

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
使用 item2 远程 ssh 连接服务器后，同样需要在远程服务器上安装 lrzsz，否则无法上传下载文件
`yum install lrzsz`

### item2 添加 ssh tab 区分
安装 oh-my-zsh 之后，修改 oh-my-zsh 文件，文件末尾添加以下配置  
`vi ~/.oh-my-zsh/oh-my-zsh.sh`
``` shell
# Usage:
# source iterm2.zsh

# iTerm2 tab color commands
# https://iterm2.com/documentation-escape-codes.html

if [[ -n "$ITERM_SESSION_ID" ]]; then
    tab-color() {
        echo -ne "\033]6;1;bg;red;brightness;$1\a"
        echo -ne "\033]6;1;bg;green;brightness;$2\a"
        echo -ne "\033]6;1;bg;blue;brightness;$3\a"
    }
    tab-red() { tab-color 255 0 0 }
    tab-green() { tab-color 0 255 0 }
    tab-blue() { tab-color 0 0 255 }
    tab-reset() { echo -ne "\033]6;1;bg;*;default\a" }

    function iterm2_tab_precmd() {
        tab-reset
    }

    function iterm2_tab_preexec() {
        if [[ "$1" =~ "^ssh " ]]; then
            if [[ "$1" =~ "prod" ]]; then
                tab-color 255 160 160
            else
                tab-color 160 255 160
            fi
        else
            tab-color 160 160 255
        fi
    }

    autoload -U add-zsh-hook
    add-zsh-hook precmd  iterm2_tab_precmd
    add-zsh-hook preexec iterm2_tab_preexec
fi
```

修改完成后重新启动 item2 ，使用 ssh 连接服务器之后，tab 页会显示与其他不同的颜色

线上操作，谨慎而为。

### ssh 别名配置
`~/.ssh/config`
``` shell
Host myserver # myserver 可以替换为想设置的别名
    HostName ip # 远程主机的 IP 地址
    User user # 远程主机的用户名
    Port port # 远程主机的端口号
```