---
layout: wiki
title: vscode设置
categories: vscode
description: vscode设置
keywords: vscode
---
## .json配置文件变量
${workspaceFolder} :表示当前workspace文件夹路径，也即/home/Coding/Test  

${workspaceRootFolderName}:表示workspace的文件夹名，也即Test  

${file}:文件自身的绝对路径，也即/home/Coding/Test/.vscode/tasks.json  

${relativeFile}:文件在workspace中的路径，也即.vscode/tasks.json  

${fileBasenameNoExtension}:当前文件的文件名，不带后缀，也即tasks  

${fileBasename}:当前文件的文件名，tasks.json

${fileDirname}:文件所在的文件夹路径，也即/home/Coding/Test/.vscode  

${fileExtname}:当前文件的后缀，也即.json  

${lineNumber}:当前文件光标所在的行号  

${env:PATH}:系统中的环境变量  

## launch.json 配置文件
`ctrl+shift+p`输入 Debug: Open launch.json 打开 launch.json 文件，如果第一次打开,会新建一个配置文件，默认配置内容如下
```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Launch",
            "type": "go",
            "request": "launch",
            "mode": "auto",
            "program": "${fileDirname}",
            "env": {},
            "args": []
        }
    ]
}
```
常见属性如下

|   属性    |   介绍    |
|   ----    |   ----    |
|   name    |   调试界面下拉选择项的名称    |
|   type    |   设置为go无需改动，是 vs code 用于计算调试代码需要用哪个扩展    |
|   mode    |   可以设置为 auto, debug, remote, test, exec 中的一个    |
|   program    |   调试程序的路径（绝对路径）    |
|   env    |   调试时使用的环境变量。例如:{ "ENVNAME": "ENVVALUE" }    |
|   envFile    |   包含环境变量文件的绝对路径，在 env 中设置的属性会覆盖 envFile 中的配置    |
|   args    |   传给正在调试程序命令行参数数组    |
|   showLog    |   布尔值，是否将调试信息输出    |
|   logOutput    |   配置调试输出的组件（debugger, gdbwire, lldbout, debuglineerr, rpc）,使用,分隔， showLog 设置为 true 时，此项配置生效    |
|   buildFlags    |   构建 go 程序时传给 go 编译器的标志    |
|   remotePath    |   远程调试程序的绝对路径，当 mode 设置为 remote 时有效    |
|   ${workspaceFolder}    |   调试 VS Code 打开工作空间的根目录下的所有文件    |
|   ${file}    |   调试当前文件    |
|   ${fileDirname}    |   调试当前文件所在目录下的所有文件    |