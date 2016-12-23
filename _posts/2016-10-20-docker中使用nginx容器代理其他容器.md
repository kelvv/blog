---
layout: post
title: "docker中使用nginx容器代理其他容器"
description: "相信nginx大家也不陌生，大部分情况下都是在服务器中直接安装Nginx代理，但是如今Docker当道，如何结合Do..."
tags: [docker]
---

> Nginx  is an HTTP and reverse proxy server, a mail proxy server, and a generic TCP/UDP proxy server, originally written by [Igor Sysoev](http://sysoev.ru/en/)

### 概述

相信nginx大家也不陌生，大部分情况下都是在服务器中直接安装Nginx代理，但是如今Docker当道，如何结合Docker的容器化优势和Nginx的反向代理和域名设别？

下面利用一个Demo，搭建一组服务
* Nginx
* Ghost

利用Nginx容器内联到Ghost，转发，有以下优点：
1. 服务器只需要开一个端口给Nginx容器作为入口
2. 添加其他应用的时候，只需要在Nginx中配置转发规则就可以
3. 所有应用自带开机重启
4. 部署内容转变为文件形势，可轻松传递、维护

放出Github地址： [docker-nginx-demo](https://github.com/kelvv/docker-nginx-demo)

### 准备

需要准备Docker环境：
* [docker-engine](https://docs.docker.com/engine/installation/linux/ubuntulinux/)
* [docker-compose](https://docs.docker.com/compose/install/)

### 分析

demo的文件结构：
{% highlight markdown %}
- docker-compose.yml  //docker-compose所需的文件，用于描述所有应用的配置信息
- nginx
   - Dockerfile
   - sites-enabled
      - default   //nginx容器的配置文件，用于配置如何连接并解析其他的容器
{% endhighlight %}

docker-compose.yml:
{% highlight markdown %}
version: "2"
services:
  ghost:
    image : ghost
    restart: always
    container_name: ghost

  nginx-host:
    build: ./nginx
    restart: always
    ports:
      - "80:80"
    links:
      - ghost
    container_name: nginx-host
{% endhighlight %}

注意： links节点是内联其他的容器，该处定义了两个容器，一个是Ghost，一个是Nginx，Nginx容器是基于./nginx文件夹进行build的，端口只需要开通80指向nginx容器即可，其他的全部有nginx负责转发

Dockerfile：
{% highlight markdown %}
FROM tutum/nginx
ADD sites-enabled/ /etc/nginx/sites-enabled
{% endhighlight %}

sites-enabled/default:
{% highlight js %}
server {
	#listen 443;
	listen 80;
   server_name blog.kelvv.com
	#ssl on;
	access_log /var/log/nginx/api-dev.log;
	error_log /var/log/nginx/api-dev.error.log;
	location / {
		proxy_pass 		http://ghost:2368;
		proxy_set_header 	Host $host;
		proxy_set_header 	X-Real-IP $remote_addr;
		proxy_set_header	X-Forwarded-Proto https;
		proxy_set_header 	X-Forwarded-For $proxy_add_x_forwarded_for;
		proxy_connect_timeout 	150;
		proxy_send_timeout 	100;
		proxy_read_timeout	100;
		proxy_buffers		4 32k;
		client_max_body_size	8m;
		client_body_buffer_size	128;	
	}
}
{% endhighlight %}

注意：该处的ghost就是刚才docker-compose文件内的links中的关联 , server_name为域名

### 运行
```
$ cd docker-nginx-demo
$ docker-compose up -d
```

这个时候在浏览器中输入 域名就可以自动连接到Ghost ，服务器无污染

### 总结
最好是使用一个端口，然后所有应用都是通过80端口连接服务器，通过域名区分不同应用。 
还有一种方式就是使用多个端口，不同端口不同应用，这就需要在docker-compose.yml中开放多个端口，并且在nginx配置文件中配置多份server用于相应。