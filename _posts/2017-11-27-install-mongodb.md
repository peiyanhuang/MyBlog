---
layout: post
title:  mongodb的安装使用
date:   2017-11-27 19:00:00 +0800
categories: 数据库
tag: 数据库
---

* content
{:toc}

### Windows环境下

下载：[MongoDB Download Center](https://www.mongodb.com/download-center/community)

[windows下安装mongodb4.x版本](https://blog.csdn.net/sj2050/article/details/82838882)

安装比较简单，中间主要是选择 `Custom` 自定义安装路径。然后不断“下一步”，安装至结束。

创建 mongodb 服务有两种方式，如下：

* 方式一：

首先创建一个 data 的目录然后在 data 目录里创建 db 目录

```s
c:\>cd c:\
c:\>mkdir data
c:\>cd data
c:\data>mkdir db
c:\data>cd db
c:\data\db>
```

因为启动 mongodb 服务之前需要创建数据库文件的存放文件夹，否则命令不会自动创建，而且不能启动成功。

然后为了从命令提示符下运行 MongoDB 服务器，你必须从 MongoDB 目录的 bin 目录中执行 mongod.exe 文件。

```s
C:\mongodb\bin\mongod --dbpath c:\data\db
```

如果执行成功，会输出如下信息：

```txt
2017-09-25T15:54:09.212+0800 I CONTROL  Hotfix KB2731284 or later update is not
installed, will zero-out data files
2017-09-25T15:54:09.229+0800 I JOURNAL  [initandlisten] journal dir=c:\data\db\j
ournal
2017-09-25T15:54:09.237+0800 I JOURNAL  [initandlisten] recover : no journal fil
es present, no recovery needed
2017-09-25T15:54:09.290+0800 I JOURNAL  [durability] Durability thread started
2017-09-25T15:54:09.294+0800 I CONTROL  [initandlisten] MongoDB starting : pid=2
488 port=27017 dbpath=c:\data\db 64-bit host=WIN-1VONBJOCE88
2017-09-25T15:54:09.296+0800 I CONTROL  [initandlisten] targetMinOS: Windows 7/W
indows Server 2008 R2
2017-09-25T15:54:09.298+0800 I CONTROL  [initandlisten] db version v3.0.6
……
```

我们可以在命令窗口中运行 mongo.exe 命令即可连接上 MongoDB，执行如下命令：

    C:\mongodb\bin\mongo.exe

* 方式二：

管理员模式打开命令行窗口

创建目录，执行下面的语句来创建数据库和日志文件的目录

    mkdir c:\data\db
    mkdir c:\data\log

创建一个配置文件。该文件必须设置 systemLog.path 参数，包括一些附加的配置选项更好。

例如，创建一个配置文件位于 C:\mongodb\mongod.cfg，其中指定 systemLog.path 和 storage.dbPath。具体配置内容如下：

```s
systemLog:
    destination: file
    path: c:\data\log\mongod.log
storage:
    dbPath: c:\data\db
```

通过执行 mongod.exe，使用 `--install` 选项来安装服务，使用 `--config` 选项来指定之前创建的配置文件。

    C:\mongodb\bin\mongod.exe --config "C:\mongodb\mongod.cfg" --install --serviceName "MongoDB"

启动MongoDB服务

    net start MongoDB

关闭MongoDB服务

    net stop MongoDB

移除 MongoDB 服务

    C:\mongodb\bin\mongod.exe --remove

在浏览器输入 http://localhost:27017 （27017是mongodb的默认端口号）可以查看是否成功开启服务。

### Linux下

[安装与配置MongoDB(Linux)](https://blog.csdn.net/qq_33206732/article/details/79863885)

### MongoDB的可视化工具

1，推荐 Robomongo

Robomongo 是开源，免费的MongoDB管理工具

2，MongoBooster

支持MongoDB 3.2 版本，个人使用免费，用于商业收费