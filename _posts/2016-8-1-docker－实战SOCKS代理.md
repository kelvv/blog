---
layout: post
title: "docker－实战SOCKS代理"
description: "实现自己的socket代理，做梯子"
tags: [docker]
---

>　　Docker 是一个开源的应用容器引擎，让开发者可以打包他们的应用以及依赖包到一个可移植的容器中，然后发布到任何流行的 [Linux](http://baike.baidu.com/view/1634.htm) 机器上，也可以实现[虚拟化](http://baike.baidu.com/view/729629.htm)。容器是完全使用[沙箱](http://baike.baidu.com/subview/720343/13998082.htm)机制，相互之间不会有任何接口（类似 iPhone 的 app）。

## Docker是啥？
### 简介　
　　Docker自2013年以来非常火，各公司、个人陆陆续续地玩起了Docker,下面简单介绍下Docker的组成，并且会以一个科学上网的实例来实操。
　　
### 拆解Docker
 1. Dockerfile: 相当于脚本文件，可让docker执行的脚本文件，可用它来生成一个镜像Images，我们可以把Images里面应该有些什么应用、构建的时候执行什么脚本、运行的时候又进行什么动作，都可以通过Dockerfile进行描述。
　　
 2. Images：相信大家应该装过系统吧？装系统之前需要先准备好你想要的系统的镜像文件iso，在Docker中的Images和iso很想，也一个镜像，它的用处就是能被用于安装、保存、传递。我们创建的镜像Images可以发布到docker hub（理解为镜像市场）,然后全世界都可以使用你的镜像了。
　　
 3. Contianer:有了镜像Images，每一次安装Images，就会产生相应一个Contianer，每一个Contianer其实就是一间小房子，里面运行一个系统(linux)，系统里面默认运行着各个应用，但是神奇的是，这个小房子相对独立，当它不想和外界交流的时候，绝对是隔离的。当然也有方式提供接口给外界访问（有窗口嘛）。

### 实战
　　每个程序员都会经历折腾怎么去google查资料，去twitter等等，现在主流的就是VPN、ss等，那么今天我们创建一个ss的镜像，我的主要目的是为了通过例子去让人更容易理解Docker的运行原理，该例子是一个小例子，用于入门，谢谢。

![思维导图](http://upload-images.jianshu.io/upload_images/1952818-2af4f0b6cf30fbca.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 开始
1. 准备一个香港的或者海外的服务器(Ubuntu为例)，用于测试也可以在本地虚拟机内运行。
2. 使用ssh连接到服务器，默认在用户根目录：～ 
```
    //git环境准备
    apt-get install git
    git //查看是否安装成功
    git clone https://github.com/Jarvin-Guan/shadowsocks-docker.git //拉取代码

    //安装Docker (Ubuntu)
    apt-get install docker.io
    service docker.io status
    service docker.io start
    ln -sf /usr/bin/docker.io /usr/local/bin/docker

    //创建Images
    docker build -t username/imagename .

    //上传images
    //先在docker hub中注册一个账号
    docker login --username=用户名 --email=邮箱 //登录docker hub
    docker push username/imagename //推送镜像到docker hub

    //安装ss
    //你只需要修改端口(7878),密码(docker)。
    //下载shadowsock客户端，使用：
    //账号：你的服务器地址。
    //端口：你修改后的端口号。
    //密码：你修改后的密码。
    docker run -it -p 7878:431 -e "password=docker" -d username/imagename
    //这里使用-e设置容器内的环境变量，便于容器内的某些脚本使用该变量。

    //ok,现在你可以下载ss客户端，然后使用你的服务器ip地址，端口号，密码来科学上网了
  ```

 3. 代码讲解
  大家可以浏览[代码](https://github.com/Jarvin-Guan/shadowsocks-docker),里面有两个文件

Dockerfile
{% highlight js %}
FROM ubuntu:14.04  //使用ubuntu版本14.04为基础，也就是说该镜像创建出来的容器时运行在ubuntu上的
RUN apt-get update //更新apt-get
RUN apt-get install -y python-pip //安装python-pip，这里的-y意思是遇到询问的一律选yes
RUN pip install shadowsocks  //使用pip安装ss
COPY ./ServerStart.sh ./ServerStart.sh //把sh脚本文件拷贝到容器容器里面，这种方式便于扩展。
ENTRYPOINT ["/bin/sh","./ServerStart.sh"] //运行sh文件，所以，所有的动作都是在sh中。
{% endhighlight %}

ServerStart.sh
{% highlight js %}
 //#!/bin/sh
ssserver -p 431 -k ${password} -m aes-256-cfb //这里使用#{password}，这是引用环境变量。
//具体在哪里设置环境变量呢？当然是用户启动镜像的时候。
{% endhighlight %}

## 总结
1. docker build 用于使用Dockerfile创建出镜像。
2. docker run 用于使用镜像生成一个镜像对应的Container。
3. 环境变量在docker run 的时候传入-e参数传入，使用环境变量的时候，因为大多数都是sh文件，所以在sh文件使用${变量名}来进行使用。