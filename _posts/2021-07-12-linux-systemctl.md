---
layout: post
title: Linux systemd && systemctl
categories: [linux, 自启动, systemd, systemctl]
description: Linux systemd && systemctl
keywords: linux, 自启动, systemd, systemctl
---

## systemd 文件夹配置文件夹
systemd 配置文件存在于以下三个文件夹中：          

- `/etc/systemd/system` 存放系统启动的默认级别及启动的 unit 的软连接，优先级最高
- `/run/systemd/system` 系统执行过程中产生的服务脚本，优先级次之
- `/usr/lib/systemd/system` 存放系统上所有的启动文件。优先级最低


## unit 分类
unit 的定义文件可以根据其后缀名称识别其定义的类型，可以使用 systemctl -t help 查看。
- `.servicre` 定义了系统服务的启动
- `.target`  定义了系统启动的级别标签，systemd 没有运行级别的概念，创建标签只是为了兼容老版本。
- `.socket` 定义了进程通信用到的套接字，套接字与进程是分离的
- `.device` 定义了系统启动时内核识别的文件，systemd 提供了设备的管理功能，/dev 下的设备由 /etc/udev/ 下的配置文件与.device 共同定制
- `.mount` 定义了系统的文件系统的挂载点
- `.snapshop` 系统快照
- `.swap` 用于标识 swap 设备
- `.automount` 文件系统的自动挂载点
- `.path` 用于定义文件系统中的一个文件或目录使用。常用于文件系统发生变化时，延迟激活服务。

## 文件组成
文件通常由 3 段组成：
``` shell
[Unit]

[unit 的类型：service target socket]

[install]
```

### [Unit]
不属于第二个标签的定义都放在这里，或存放不属于 unit 类型的定义，描述信息，依赖的 unit
- `Description`：描述信息
- `After`：表明需要依赖的服务，作用决定启动顺序
- `Before`：表明被依赖的服务
- `Requles`：依赖到的其他 unit ，强依赖，即依赖的 unit 启动失败。该 unit 不启动。
- `Wants`：依赖到的其他 unit，弱依赖，即依赖的 unit 启动失败。该 unit 继续启动
- `Conflicts`：定义冲突关系

### [Unit 类型 ]
#### [Service]：
- `type`：启动时关系的定义，
  * `simple`：(当设置了 `ExecStart=` 、 但是没有设置 `Type=` 与 `BusName=` 时，这是默认值)， 那么 `ExecStart=` 进程就是该服务的主进程
  * `exec`：与 `simple` 类似，不同之处在于， 只有在该服务的主服务进程执行完成之后，`systemd` 才会认为该服务启动完成。
  * `forking` ：`exec` 启动的进程生成的其中一个子进程成为主进程，启动完成后，旧的主进程会退出。
  * `ontshot`：启动下一个进程前主进程退出。
  * `dbus`：与 `simple` 类似，不同之处在于， 该服务只有获得了 `BusName=` 指定的 `D-Bus` 名称之后，`systemd` 才会认为该服务启动完成，才会开始启动后继单元。
  * `notify`：与 `exec` 类似，不同之处在于， 该服务将会在启动完成之后通过 `sd_notify(3)` 之类的接口发送一个通知消息
  * `ldle`：与 `simple` 类似，不同之处在于， 服务进程将会被延迟到所有活动任务都完成之后再执行。
- `PIDFile`：`type` 字段如果设置为 `forking` ，建议同时设置 `PIDFile` 选项，以帮助 `systemd` 准确可靠的定位该服务的主进程。 `systemd` 将会在父进程退出之后 立即开始启动后继单元。
- `EnvironmentFile`：指定当前服务的环境参数文件。该文件内部的 `key=value` 键值对，可以用 `$key` 的形式，在当前配置文件中获取。
- `ExecStart`：启动服务需要执行的命令
- `ExecStartpre`：启动服务之前执行的命令
- `ExecStartpost`：启动服务之后执行的命令
- `ExecStop`：停止服务时执行的命令
- `Restart`：定义启动进程时执行的命令
- `ExecReload`：重启服务时执行的命令
- `KillMode`：定义如何停止服务
  * `control-group`（默认值）：当前控制组里面的所有子进程，都会被杀掉
  * `process`：只杀主进程
  * `mixed`：主进程将收到 `SIGTERM` 信号，子进程收到 `SIGKILL` 信号
  * `none`：没有进程会被杀掉，只是执行服务的 `stop` 命令。
- `RestartSec`：表示重启服务之前，需要等待的秒数。

#### [install]
定义如何安装这个配置文件，即怎样做到开机启动

- Alias
- RequlredBy: 被那些 unit 所依赖，
- WanteBy: 表示该服务所在的 Target，含义是服务组，表示一组服务。WantedBy=multi-user.target 指的是，sshd 所在的 Target 是 multi-user.target。

## systemctl 命令
``` shell
# 列出正在运行的 Unit
systemctl list-units，可以直接使用 systemctl

# 列出所有 Unit，包括没有找到配置文件的或者启动失败的
systemctl list-units --all

# 列出所有没有运行的 Unit
systemctl list-units --all --state=inactive

# 列出所有加载失败的 Unit
systemctl list-units --failed

# 列出所有正在运行的、类型为 service 的 Unit
systemctl list-units --type=service

# 显示某个 Unit 是否正在运行
systemctl is-active application.service

# 显示某个 Unit 是否处于启动失败状态
systemctl is-failed application.service

# 显示某个 Unit 服务是否建立了启动链接
systemctl is-enabled application.service

# 立即启动一个服务
systemctl start apache.service

# 立即停止一个服务
systemctl stop apache.service

# 重启一个服务
systemctl restart apache.service

# 重新加载一个服务的配置文件
systemctl reload apache.service

# 重载所有修改过的配置文件
systemctl daemon-reload
```

## 注意事项
注：修改了的 unit 文件 需要重载。`systemctl daemon-reload`