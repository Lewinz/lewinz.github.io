---
layout: post
title: vscode 插件开发
categories: [vscode, 插件]
description: vscode 插件开发
keywords: vscode, 插件
---

## 插件发布
想要把开发好的插件发布至 vscod 插件市场应用，需要以下步骤：

1. 注册账号
2. 插件打包发布

### 注册账号

1. 注册/登录 Microsoft 账号<https://login.live.com/>
2. 注册/登录 Azure 账号<https://aka.ms/SignupAzureDevOps>
3. 创建 Azure 组织，并生成 `Personal Access Tokens`

> Token 生成需要授权 Manager 权限，或者 Full 权限  
> 生成后拷贝保存，下次打开无法查看

### 插件打包发布
使用 vsce 工具

``` shell
## 1. 全局下载
npm install -g vsce

## 2. 项目打包
vsce package

## 3. 创建 publisher
https://marketplace.visualstudio.com/manage/createpublisher

## 4. 本地 login publisher
## publishername 为第 3 步创建的组织名
vsce login publishername

输入之前生成的 Personal Access Tokens

## 5. 公开
vsce publish publishername
```

> 进行到这一步，就可以在插件市场搜索到自己发布的插件了


