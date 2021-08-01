---
layout: wiki
title: golang 开发环境部署
categories: ["golang","开发环境"]
description: 开发环境
keywords: golang, 开发环境
---

### windows

下载 msi 文件安装 [golang 中文网](https://studygolang.com/dl)

环境变量配置  

```
GOROOT 安装目录
GOPATH 项目目录

在 path 环境变量中添加 %GOROOT%\bin

# 设置 go 代理，拉取 go 依赖
go env -w GOPROXY=https://goproxy.io,direct

或者

go env -w GOPROXY=https://goproxy.cn,direct
```

### Mac
```
vim ~/.zshrc

# 这个自定义，选择一个你愿意存放文件的位置
export GOPATH=/Users/hopkings/www/Go
export GOBIN=$GOPATH/bin
export PATH=$PATH:$GOPATH/bin
export GOPROXY=https://goproxy.io,direct

source ~/.zshrc
---
go env -w GO111MODULE=on
```

### go 命令解析



