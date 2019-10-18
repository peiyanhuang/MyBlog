---
layout: post
title:  React 应用的 Docker 化(译文)
date:   2019-06-25 19:00:00 +0800
categories: 译文
tag: Docker
---

* content
{:toc}

翻译自[Dockerizing a React App](https://mherman.org/blog/dockerizing-a-react-app/)

Docker是一种容器化工具，可帮助加快开发和部署过程。如果您正在使用微服务，Docker可以更轻松地将小型独立服务链接在一起。它还有助于消除特定环境的错误，因为您可以复制生产环境到本地。

本教程演示了如何使用 Create React App 生成一个 Dockerize React 应用程序。我们需要特别关注

1. 使用代码热更新设置开发环境
2. 使用多级构建配置生产环境的映像

### 环境

- Docker v18.09.2
- Create React App v3.0.1
- Node v12.2.0

### 项目设置

全局安装 Creact React App:

```bash
$ npm install -g create-react-app@3.0.1
```

生成一个新的项目：

```bash
$ create-react-app sample
$ cd sample
```

### Docker

将 Dockerfile 文件添加到项目根目录：

```bash
# base image
FROM node:12.2.0-alpine

# set working directory
WORKDIR /app

# add `/app/node_modules/.bin` to $PATH
ENV PATH /app/node_modules/.bin:$PATH

# install and cache app dependencies
COPY package.json /app/package.json
RUN npm install --silent
RUN npm install react-scripts@3.0.1 -g --silent

# start app
CMD ["npm", "start"]
```

> 通过 `--silent` 设置不要 NPM 输出 log 是个人选择。但是不赞成使用，因为它可以吞下错误。请记住这一点，这样您就不会浪费时间进行调试。

添加 `.dockerignore`:

```
node_modules
```

这将加速 Docker 构建过程，因为我们的本地依赖项将不会发送到 Docker 的守护进程。

构建并标记 Docker 镜像:

```bash
$ docker build -t sample:dev .
```

然后，在构建完成后运行容器：

```bash
$ docker run -v ${PWD}:/app -v /app/node_modules -p 3001:3000 --rm sample:dev
```

> 如果遇到 `"ENOENT: no such file or directory, open '/app/package.json"` 的错误，您可能需要添加额外的参数： `-v /app/package.json`。

在这里发生了什么？

1. `docker run` 命令用我们刚刚创建的镜像来创建一个新的容器实例，然后运行它;
2. `-v ${PWD}:/app` 将代码安装到了 `/app` 的容器中;
> `{PWD}` 可能无法在 Windows 上运行。有关详细信息，请参阅[此Stack Overflow问题](https://stackoverflow.com/questions/41485217/mount-current-directory-as-a-volume-in-docker-on-windows-10)。
3. 因为我们想要使用 `node_modules` 文件夹中的容器版本，我们配置了另一个 volumes：`-v /app/node_modules`。你现在应该可以尝试删除本地的 `node_modules` 了。
4. `-p 3001:3000` 将端口 3000 暴露给同一网络上的其他 Docker 容器（用于容器间通信），将端口 3001 暴露给主机。
> 更多信息请查看此 Stack Overflow 上的[问题](https://stackoverflow.com/questions/22111060/what-is-the-difference-between-expose-and-publish-in-docker)。
5. 最后，`- rm` 在退出容器后删除容器和 volumes。

将浏览器打开到 `http://localhost:3001/`，您应该会看到该应用程序。尝试在代码编辑器中更改 App 组件。你应该看到应用程序的热重载。完成后关闭服务器。

>添加 `-it` 会发生什么？
>`$ docker run -it -v ${PWD}:/app -v /app/node_modules -p 3001:3000 --rm sample:dev`检查您的理解，并自己查看。


想使用 Docker Compose？将 `docker-compose.yml` 文件添加到项目根目录：

```yml
version: '3.7'

services:

  sample:
    container_name: sample
    build:
      context: .
      dockerfile: Dockerfile
    volumes:
      - '.:/app'
      - '/app/node_modules'
    ports:
      - '3001:3000'
    environment:
      - NODE_ENV=development
```

注意 `volumes`。如果没有[匿名 volumes](https://success.docker.com/article/different-types-of-volumes)（`/app/node_modules`），`node_modules` 目录将在运行时通过挂载主机目录来覆盖。换句话说，这时会发生：

- Build：`node_modules` 目录会在镜像中创建；
- Run：当前目录将被装入容器中，覆盖构建期间安装的 `node_module`。

构建镜像并启动容器：

```bash
$ docker-compose up -d --build
```

确保应用程序在浏览器中运行并再次测试热重载。在继续之前停止容器：

```bash
$ docker-compose stop
```

> Windows用户：在使 volumes 正常工作时遇到问题？可以查看以下资源：
1. [Docker on Windows–Mounting Host Directories](https://rominirani.com/docker-on-windows-mounting-host-directories-d96f3f056a2c?gi=7222eb753bf1)
2. [Configuring Docker for Windows Shared Drives / Volume Mounting with AD](https://blogs.msdn.microsoft.com/stevelasker/2016/06/14/configuring-docker-for-windows-volumes/)
> 您还可能需要将 `COMPOSE_CONVERT_WINDOWS_PATHS = 1` 添加到 Docker Compose 文件的环境部分。查看[Declare default environment variables in file](https://docs.docker.com/compose/env-file/)指南以获取更多信息。

### Docker Machine

要在 Docker Machine 和 VirtualBox 上使用热加载，您需要经过 [`chokidar`](https://github.com/paulmillr/chokidar) 启用轮询机制（包含了`fs.watch`，`fs.watchFile`和`fsevents`）

创建一个新机器(sample)：

```bash
$ docker-machine create -d virtualbox sample
$ docker-machine env sample
$ eval $(docker-machine env sample)
```

抓取机器(sample)的IP地址:

```bash
$ docker-machine ip sample
```

然后，构建镜像并运行容器：

```bash
$ docker build -t sample:dev .

$ docker run -v ${PWD}:/app -v /app/node_modules -p 3001:3000 --rm sample:dev
```

在浏览器中再次测试应用程序：`http://DOCKER_MACHINE_IP:3001/`(确保将 `DOCKER_MACHINE_IP` 替换为 Docker Machine 的实际 IP 地址)。此外，确认热加载不起作用。您也可以尝试使用 Docker Compose，但结果将是相同的。

要使热加载工作，我们需要添加一个环境变量：`CHOKIDAR_USEPOLLING=true`。

```bash
$ docker run -v ${PWD}:/app -v /app/node_modules -p 3001:3000 -e CHOKIDAR_USEPOLLING=true --rm sample:dev
```

再试一次。确保热加载再次起作用。

更新下 `docker-compose.yml` 文件：

```yml
version: '3.7'

services:

  sample:
    container_name: sample
    build:
      context: .
      dockerfile: Dockerfile
    volumes:
      - '.:/app'
      - '/app/node_modules'
    ports:
      - '3001:3000'
    environment:
      - NODE_ENV=development
      - CHOKIDAR_USEPOLLING=true
```

### Production

让我们创建一个单独的 Dockerfile，用于生产环境下 `Dockerfile-prod`：

```bash
# build environment
FROM node:12.2.0-alpine as build
WORKDIR /app
ENV PATH /app/node_modules/.bin:$PATH
COPY package.json /app/package.json
RUN npm install --silent
RUN npm install react-scripts@3.0.1 -g --silent
COPY . /app
RUN npm run build

# production environment
FROM nginx:1.16.0-alpine
COPY --from=build /app/build /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

在这里，我们利用[多阶段构建(multistage build)](https://docs.docker.com/develop/develop-images/multistage-build/)模式来创建用于构建工件的临时镜像 - 生产环境下的React静态文件 - 然后将其复制到生产镜像。临时构建的镜像将会与原始文件和与映像关联的文件夹一起被丢弃。这样可以生成精简的生产环境镜像。

> 查看[Builder pattern vs. Multi-stage builds in Docker](https://blog.alexellis.io/mutli-stage-docker-builds/)以了解更多*多阶段构建*的信息。

使用生产环境下的 Dockerfile，构建并标记 Docker 镜像：

```bash
$ docker build -f Dockerfile-prod -t sample:prod .
```

运行容器：

```bash
$ docker run -it -p 80:80 --rm sample:prod
```

如果您仍在使用 Docker Machine，在浏览器中导航到 `http://DOCKER_MACHINE_IP`。

使用新的 Docker Compose 文件进行测试--`docker-compose-prod.yml`：

```bash
version: '3.7'

services:

  sample-prod:
    container_name: sample-prod
    build:
      context: .
      dockerfile: Dockerfile-prod
    ports:
      - '80:80'
```

启动容器：

```bash
$ docker-compose -f docker-compose-prod.yml up -d --build
```

在浏览器中再次进行测试。之后，如果你已经完成了，那么可以把机器销毁了：

```bash
$ eval $(docker-machine env -u)
$ docker-machine rm sample
```

### React Router and Nginx

如果您使用 React Router，那么您需要在构建时更改默认的 Nginx 配置：

```bash
RUN rm /etc/nginx/conf.d/default.conf
COPY nginx/nginx.conf /etc/nginx/conf.d
```

添加一些修改到 `Dockerfile-prod` 中:

```bash
# build environment
FROM node:12.2.0-alpine as build
WORKDIR /app
ENV PATH /app/node_modules/.bin:$PATH
COPY package.json /app/package.json
RUN npm install --silent
RUN npm install react-scripts@3.0.1 -g --silent
COPY . /app
RUN npm run build

# production environment
FROM nginx:1.16.0-alpine
COPY --from=build /app/build /usr/share/nginx/html
RUN rm /etc/nginx/conf.d/default.conf
COPY nginx/nginx.conf /etc/nginx/conf.d
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

创建以下文件夹以及 `nginx.conf` 文件：

```
└── nginx
    └── nginx.conf
```

`nginx.conf`:

```conf
server {

  listen 80;

  location / {
    root   /usr/share/nginx/html;
    index  index.html index.htm;
    try_files $uri $uri/ /index.html;
  }

  error_page   500 502 503 504  /50x.html;

  location = /50x.html {
    root   /usr/share/nginx/html;
  }

}
```

### Next Steps

有了这个，您现在应该能够将 React 添加到一个更大的 Docker 驱动项目中，无论是开发或生产环境。如果您想了解有关使用 React 和 Docker 以及构建和测试微服务的更多信息，请查看[Microservices with Docker, Flask, and React](https://testdriven.io/)的课程。