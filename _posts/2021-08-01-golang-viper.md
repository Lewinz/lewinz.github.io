---
layout: post
title: Golang 基于 viper 的配置热加载或动态变更方法介绍
categories: [golang, viper, file, reload]
description: Golang 基于 viper 的配置热加载或动态变更方法介绍
keywords: golang, viper, file, reload
---

## 概述
在写 web server 时，往往需要引入各种各样的配置信息，如依赖的其他中间件（redis、elasticsearch）等，一旦这些服务发生变更，我们需要重新启动 web server，以使配置生效。在 Golang 中，基于 viper 的动态配置就可以省去这些繁琐的步骤了。接下来用一个示例来说明如何使用 viper 的配置热加载：

## 项目结构
整个项目的目录结构:
``` golang
- DynamicConfigDemo // 项目地址
	- conf   // 配置文件目录
		- base.yaml  // 采用yaml格式文件，viper同样支持toml、json等格式的配置文件
	- src // 代码文件夹
		- dynamic_config // 动态配置文件夹
			- dynamic_config.go // 配置加载脚本
	- go.mod // go package管理依赖的包文件
	- go.sum // go package管理打包产生的文件
	- main.go // web server的入口，主函数
```

## 代码细节
各文件的主体内容：
```
# conf/base.yaml
service:
  redis:
    host: 127.0.0.1
    port: 6379
```

conf/base.yaml 文件定义了配置项，包含 redis 的 host 及 port 信息。
``` golang
# src/dynamic_config/dynamic_config.go
package dynamic_config

import (
	"fmt"
	"github.com/fsnotify/fsnotify"
	"github.com/spf13/viper"
)

var GlobalConfig *viper.Viper

func init() {
	fmt.Printf("Loading configuration logics...\n")
	GlobalConfig = initConfig()
	go dynamicConfig()
}

func initConfig() *viper.Viper {
	GlobalConfig := viper.New()
	GlobalConfig.SetConfigName("base")
	GlobalConfig.AddConfigPath("conf/")
	GlobalConfig.SetConfigType("yaml")
	err := GlobalConfig.ReadInConfig()
	if err != nil {
		fmt.Printf("Failed to get the configuration.")
	}
	return GlobalConfig
}

func dynamicConfig() {
	GlobalConfig.WatchConfig()
	GlobalConfig.OnConfigChange(func(event fsnotify.Event) {
		fmt.Printf("Detect config change: %s \n", event.String())
	})
}
```
src/dynamic_config/dynamic_config.go 定义了全局配置信息的加载及动态监控方法，init 函数为初始化执行的脚本，initConfig 为初始化当前配置，dynamicConfig 为冬天监听，通过 viper 的内部方法 WatchConfig 实现。
``` golang
# main.go
package main

import (
	"DynamicConfigDemo/src/dynamic_config"
	"fmt"
	"github.com/gin-gonic/gin"
)

func main() {
	gin.SetMode(gin.ReleaseMode)
	r := gin.Default()
	r.GET("/ping", func(context *gin.Context) {
		fmt.Println("Current redis host is: ", dynamic_config.GlobalConfig.GetString("service.redis.host"))
		context.JSON(
			200, gin.H{
				"message": "You are welcome!",
			})
	})
	r.Run(":9292")
}
```

main.go 为主函数，调用 gin 包定义 web server 服务，实现了一个简单的 http server 服务器，每次请求发送时会打印配置的 redis host 信息。

## 调用示例
1. 第一次调用，打印出当前 redis host 为 127.0.0.1
2. 随后我们将 redis host 修改为 127.0.0.2
3. src/dynamic_config/dynamic_config.go 文件中的如下代码 GlobalConfig.OnConfigChange(func(event fsnotify.Event) { fmt.Printf("Detect config change: %s \n", event.String()) }) 监控到配置文件变更时会输出变更通知，如图中红框所示；
4. 第二次调用时，则打印出最新配置的 redis host 信息

![golang_viper_file_reload](https://cdn.jsdelivr.net/gh/Lewinz/lewinz.github.io@master/images/posts/golang_viper_file_reload.png)