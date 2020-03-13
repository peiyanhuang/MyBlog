---
layout: post
title:  Node 的版本管理
date:   2020-03-08 19:00:00 +0800
categories: Node
tag: Node
---

* content
{:toc}

### 1、nvm是什么
    
nvm 全名 node.js version management，顾名思义是一个 nodejs 的版本管理工具。通过它可以安装和切换不同版本的 nodejs。下面列出下载、安装及使用方法。

### 2、下载
    
可在点此在[github](https://www.cnblogs.com/gaozejie/p/10689742.html)上下载最新版本，本次下载安装的是 windows 版本。打开网址我们可以看到有两个版本：

- nvm-noinstall.zip：绿色免安装版，但使用时需进行配置。
- nvm-setup.zip：安装版，推荐使用

### 3、安装

安装其实很简单，只需要按照步骤进行就可以了，但是需要注意的是，如果你的电脑上已经安装过node了，安装nvm的时候会询问你是否要用nvm控制你本地的node版本，确认就好。

但是如果安装完成后不能正常使用，可尝试卸载 nvm，并将本地的 nodejs 也卸载，并删除现有的 nodejs 安装目录（例如："C:\Program Files\nodejs"），然后重新安装即可。

还有一点需要注意，nvm 的安装路径名称中最好不要有空格。安装成功后，在命令行输入 nvm，有输出 usage 信息就表明安装成功了。

安装时 nvm 会自动配置环境变量，若没有则需手动配置。

### 4、命令提示

- `nvm arch`：显示 node 是运行在32位还是64位。
- `nvm install <version> [arch]`：安装 node，version 是特定版本也可以是最新稳定版本 `latest`。可选参数 `arch` 指定安装32位还是64位版本，默认是系统位数。可以添加 `--insecure` 绕过远程服务器的SSL。
- `nvm list [available]`：显示已安装的列表。可选参数 `available`，显示可安装的所有版本。`list` 可简化为ls。
- `nvm on`：开启 node.js 版本管理。
- `nvm off`：关闭 node.js 版本管理。
- `nvm proxy [url]`：设置下载代理。不加可选参数 url，显示当前代理。将 url 设置为 none 则移除代理。
- `nvm node_mirror [url]`：设置 node 镜像。默认是 `https://nodejs.org/dist/`。如果不写 url，则使用默认 url。设置后可至安装目录 `settings.txt` 文件查看，也可直接在该文件操作。
- `nvm npm_mirror [url]`：设置 npm 镜像。`https://github.com/npm/cli/archive/`。如果不写 url，则使用默认 url。设置后可至安装目录 `settings.txt` 文件查看，也可直接在该文件操作。
- `nvm uninstall <version>`：卸载指定版本 node。
- `nvm use [version] [arch]`：使用制定版本 node。可指定32/64位。
- `nvm root [path]`：设置存储不同版本 node 的目录。如果未设置，默认使用当前目录。
- `nvm version`：显示 nvm 版本。`version` 可简化为 `v`。
