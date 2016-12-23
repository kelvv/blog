---
layout: post
title: "Node服务端极速搭建 －－ nvmhome"
description: "nodejs中，使用docker搭建服务"
tags: [nodejs]
---

> 本文意在让你掌握极速搭建Node服务端（任何Project）

{% highlight js %}
$ whoami
name:        kelvin
email:       kelvv@outlook.com
homepage:    www.kelvv.com
github:      https://github.com/kelvv
{% endhighlight %}
****
本文利用Docker搭建一个预安装好:
* pm2  -- Node应用的进程管理器
* git     -- 代码管理
* ssh    -- 远程登录
* nvm   -- node版本管理
* npm   -- node包安装
* node  -- So,you know

**本项目github：[Here](https://github.com/kelvv/nvmhome-docker)** , 欢迎Start

**> 当然可直接使用该镜像：**
{% highlight js %}
$ docker pull kelvv/nvmhome-docker
{% endhighlight %}

****

该项目由一份Dockerfile和.bashrc组成：

Dockerfile

{% highlight js %}
FROM ubuntu:14.04  

RUN apt-get update
//安装git ssh
RUN apt-get install -y curl git-all openssh-server
ADD .bashrc /root/.bashrc

//nvm安装
RUN curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.31.0/install.sh | bash
RUN . /root/.bashrc;nvm install 4.3.1;nvm use 4.3.1;npm install pm2 -g;pm2 startup ubuntu
         
RUN mkdir /var/run/sshd
RUN echo 'root:myssh' | chpasswd
RUN sed -i 's/PermitRootLogin without-password/PermitRootLogin yes/' /etc/ssh/sshd_config

//SSH登录
RUN sed 's@session\s*required\s*pam_loginuid.so@session optional pam_loginuid.so@g' -i /etc/pam.d/sshd

ENV NOTVISIBLE "in users profile"
RUN echo "export VISIBLE=now" >> /etc/profile
    
//对外开放端口
EXPOSE 22  
EXPOSE 80
    
CMD ["/usr/sbin/sshd", "-D"] 
{% endhighlight %}

上面给出了简单的注释，下面补充说明一下
1. ADD .bashrc /root/.bashrc  为什么需要把.bashrc添加进去呢？因为里面把**互动**去掉，这样pm2环境变量才能生效。
2. 默认ssh的密码为：'myssh'，如有需要，自行添加ssh无密登录，我对该技术有文章：[传送门](http://www.jianshu.com/p/27d8b8d1d571)
3. 该镜像对外开放两个端口，
      * **22:  ssh端口**   
      * **80:  默认网站端口**
4. 大家可以以该Dockerfile为基础，继续写属于自己应用的Dockerfile。Happy Node

## 使用

* 使用该镜像
连接到服务器，执行以下命令
{% highlight js %}
$ docker pull kelvv/nvmhome-docker
$ docker run -p 0.0.0.0:3000:22 -p 0.0.0.0:80:80 -d kelvv/nvmhome-docker
{% endhighlight %}
你可以回到自己的电脑，使用ssh root@xxx.xxx.xxx.xxx连接到刚创建的应用服务端，自由发挥。

* 对该镜像有改进，需重新构建
连接到服务器，下载Dockerfile，修改后执行以下命令
{% highlight js %}
$ docker build -t name .   
$ docker run -p 0.0.0.0:3000:22 -p 0.0.0.0:80:80 -d name
{% endhighlight %}

## 总结

以上已经建立好基础，下一篇文章会说说：如何使用nvmhome＋pm2，一键发布、管理自己的应用，，是一个很简单的个人主页,希望你们喜欢！

我的个人主页就是这样部署的：[Homepage传送门](http://www.kelvv.com/)