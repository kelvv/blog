---
layout: post
title: "使用docker-compose来打包开发环境"
description: "开发环境涉及到数据库、缓存数据库、管理工具等的应用，应该把它们打包好，一键安装，自带重启"
tags: [docker]
---

> BUILD, SHIP, RUN
Docker is the world’s leading software containerization platform

Docker的出现，让虚拟技术更上一个台阶。更有Docker Hub作为docker images的市场载体，让全世界分享你的成果。

建议使用Docker for Mac(10.10.3) 或 Docker for Windows(10)  , 抛弃旧的Docker Toolbox

我对docker的应用有：
1. 使用docker-compose封装公司后端组开发所需要的环境。
2. 使用docker-compose封装我的服务器需要运行的各个应用。

下面分别介绍用处，和优势

___
1.在团队开发中，会遇到这样一个问题：开发机器统一会用到一些必备的应用，例如mongodb、mencached、redis等等，那么如何维护项目所需的预装软件？传统的方式是纪录成文档，然后有新人来的话就给他文档，然后一个一个安装。docker的出现，给应用带来了福音，下面看如何用docker-compose解决上诉问题。

 解决方案：通过一个文件(docker-compose.yml)描述组内开发所需要的应用，然后上传git代码管理，有更新直接同步，有新同事来了或者一个新的机器要做成开发机，两步走：
* 安装docker和docker-compose   
* 获取描述文件，执行命令
事例：
 docker-compose.yml
{% highlight js %}
 version: '2'
 services:
  mongodb:
    image: tutum/mongodb
    ports:
      - "27017:27017"
      - "28017:28017"
    environment:
      - AUTH=no
    container_name: mongodb
    restart: always

  memcached:
    image: memcached
    ports:
      - "11211:11211"
    container_name: memcached
    restart: always

{% endhighlight %}
效果：只要运行docker-compose up，就会自动安装mongodb和memcached，并且会在机器重启的时候自启动。


2.服务器的部署.

docker-compose.yml
{% highlight js %}
version: '2'
services:
  homepage:
    image: kelvv/nvmhome-docker:v1.1.5
    ports:
      - "0.0.0.0:3000:22"
      - "0.0.0.0:81:3000"
      - "0.0.0.0:3101:3100"
    environment:
      - NODE_ENV=staging
      - projecturl=https://github.com/kelvv/my-site.git
      - autoupdate=true
    container_name: kelvv-homepage
    restart: always

  shadowsock:
    image: kelvv/shadowsock-docker
    ports:
      - "0.0.0.0:7878:431"
    environment:
      - password=docker
    container_name: kelvv-shadowsock
    restart: always

  ppt:
    image: kelvv/nvmhome-docker:v1.1.5
    ports:
      - "0.0.0.0:82:3000"
      - "0.0.0.0:3102:3100"
    environment:
      - projecturl=https://github.com/kelvv/my-ppt.git
      - autoupdate=true
    container_name: kelvv-ppt
    restart: always

{% endhighlight %}

 效果：安装完以后，在服务器会安装3个应用
  *  个人主页
  *  shadowsock 科学上网
  *  个人ppt源

总结 ：尽早使用docker，减少一点麻烦