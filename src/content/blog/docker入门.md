---
author: magnum-zx
pubDatetime: 2023-06-07T14:27:00.000+08:00
title: Docker Desktop：你没有理由还不会使用docker了
postSlug: docker-desktop-usage
featured: true
ogImage: "https://cdn.jsdelivr.net/gh/magnum-zx/blog-image@main/img/Docker-Logo-2013.png"
tags:
  - docker
description: ""
---

# Docker Desktop：你没有理由还不会使用 Docker 了

<img src="https://cdn.jsdelivr.net/gh/magnum-zx/blog-image@main/img/Docker-Logo-2013.png"></img>

> Docker 是现在很流行的容器化技术，作为一名合格的开发者，没有理由还不会使用 Docker，利用 Docker Desktop 图形化界面可以很方便的对 Docker 镜像和容器进行操作

## 下载 Docker Desktop

首先，我们可以直接通过 docker 的[官方网站](https://www.docker.com/products/docker-desktop/)来下载 docker desktop 安装包

<img src="https://cdn.jsdelivr.net/gh/magnum-zx/blog-image@main/img/docker-desktop-download.png"></img>

安装完成后，可以通过桌面的 Docker App 进入 Docker Desktop 界面
<img src="https://cdn.jsdelivr.net/gh/magnum-zx/blog-image@main/img/docker-icon.png" width="200px"></img>

<img src="https://cdn.jsdelivr.net/gh/magnum-zx/blog-image@main/img/docker-desktop.png"></img>

- Containers: 用来管理 Docker 容器
- Images: 用来管理 Docker 镜像
- Volumes: 用来给 Docker 容器存储数据
- Dev Environments: 可以用来创建一个项目开发环境，方便团队成员在同一个环境下进行开发
- Learning Center: 可以学习如何使用 Docker

并且 Docker Desktop 提供了一个类似 VS code 的插件系统，可以使用第三方插件来拓展 Docker Desktop 的功能。

## 获取 Docker Image（镜像）

想要创建 Docker 容器，首先得有一个 Docker Image（镜像），镜像可以通过本地创建，也可以通过远程仓库（如 [Docker Hub](https://hub.docker.com)）获取。

### 通过 Dockerfile 创建本地镜像

假如想要创建一个 nodejs 容器，可以本地创建一个项目，如图：
<img src="https://cdn.jsdelivr.net/gh/magnum-zx/blog-image@main/img/docker-node-project-dir.png"></img>

然后在目录中创建一个 `Dockerfile` 文件：

```
FROM node:14-alpine # 基础镜像

COPY . . # 将源文件复制到目标文件

RUN npm install # RUN 用于执行后面跟着的命令行命令，在docker build时执行

CMD [ "node", "./index.js" ] # 类似于 RUN 指令，用于运行程序，但是在docker run时执行
```

然后通过 `docker build` 命令，创建一个镜像：

```shell
$ docker build -t IMAGE_NAME .
```

- -t: `--tag`的缩写，表示镜像的名字和标签，后接 name 或 name: tag

- .: Dockerfile 的路径

<image src="https://cdn.jsdelivr.net/gh/magnum-zx/blog-image@main/img/docker-build.png"></image>

然后，我们可以在 Docker Desktop 中看到这个本地镜像

<image src="https://cdn.jsdelivr.net/gh/magnum-zx/blog-image@main/img/docker-node-image.png"></image>

或者通过 `docker image ls` 命令查看本地镜像
<image src="https://cdn.jsdelivr.net/gh/magnum-zx/blog-image@main/img/docker-image-ls.png"></image>

### 通过 Docker Hub 下载镜像

假如将要创建一个 nodejs 容器，可以直接通过 `docker pull` 命令下载一个 nodejs 镜像：

```bash
$ docker pull node
```

其他更多的镜像，可以在[Docker Hub](https://hub.docker.com)中查询

## 创建一个 Docker 容器

Docker 容器可以通过 Docker Desktop 来创建，也可以通过传统的命令行方法来创建

<!-- 通过 Docker Desktop 创建 -->

在 Docker Desktop Image 列表界面直接创建容器：
<image src="https://cdn.jsdelivr.net/gh/magnum-zx/blog-image@main/img/docker-node-build.png"></image>

或者进入镜像详情，在详情中创建容器：
<image src="https://cdn.jsdelivr.net/gh/magnum-zx/blog-image@main/img/docker-node-build-2.png"></image>
<image src="https://cdn.jsdelivr.net/gh/magnum-zx/blog-image@main/img/docker-node-build-3.png"></image>

点击 `run` 按钮后，会出现一个配置，可以设置一些容器的配置项：
<image src="https://cdn.jsdelivr.net/gh/magnum-zx/blog-image@main/img/docker-node-run-option.png"></image>

<!-- 2）也可以使用 `docker run` 命令来创建容器：

```shell
$ docker run docker-node
``` -->

## 将本地镜像 push 到 Docker Hub

使用 Docker Desktop 将本地镜像 push 到 Docker Hub
<image src="https://cdn.jsdelivr.net/gh/magnum-zx/blog-image@main/img/docker-push-to-hub.png"></image>

如果在将本地镜像 push 到 Docker Hub 时，出现报错，如：
<image src="https://cdn.jsdelivr.net/gh/magnum-zx/blog-image@main/img/docker-image-push-error.png"></image>

1）可能是用户未登录，可以通过以下命令在命令行中登录，或直接通过 Docker Desktop 登录。

```shell
$ docker login
```

<image src="https://cdn.jsdelivr.net/gh/magnum-zx/blog-image@main/img/docker-login.png"></image>

2）也可能是镜像名字不规范，需要改成 `USER_NAME/IMAGE_NAME:TAG` 形式，省略 `:TAG` 会自动转化为 `:lastest`

```shell
$ docker tag docker-node USER_NAME/docker-node
```

> 如果想要更加深入了解 Docker，可以查看 [Docker](https://docs.docker.com/)的官方文档，或者[Docker 从入门到实践](https://vuepress.mirror.docker-practice.com/)
