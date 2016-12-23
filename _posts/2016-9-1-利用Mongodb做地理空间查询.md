---
layout: post
title: "利用Mongodb做地理空间查询"
description: "在移动开发中，经常会用到定位的功能，例如美团、饿了么、猫眼电影等的..."
tags: [mongodb]
---

> MongoDB
是一个基于分布式文件存储的数据库。由[C++](http://baike.baidu.com/view/824.htm)语言编写。旨在为WEB应用提供可扩展的高性能数据存储解决方案。

## 前言
在移动开发中，经常会用到定位的功能，例如美团、饿了么、猫眼电影等的app，都是使用了移动端定位，然后查找出自己地理位置附近的一些服务、信息。

所以本篇文章将会以Mongodb为数据库，讲述如何在数据库层级进行定位查询。

## 分析
举个例子，我们需要做一个app，商家可以把自己的商品发布到app上，用户打开app查看离自己从近到远的商品。

如果没有地理位置的需求，那好办，直接插库然后查库就ok了，但是如果用到了地理位置，则需要用到Mongodb的一些位置功能。

Mongodb有一种地理空间索引，利用它可以进行经纬度的计算，下面继续介绍如何使用该功能。

## 实现
下面以Nodejs+mongoose为例

1. 创建Schema：
{% highlight js %}
const mongoose = require( 'mongoose' );
let goodsSchema = new mongoose.Schema( {
    name: String,
    price: Number,
    location: {
        type: [ Number ],
        index: {
            type: '2dsphere',
            sparse: true
        }
    }
}, {
    collection: 'Goods'
} )
{% endhighlight %}

2. 创建Model
{% highlight js %}
let goodsModel = mongoose.model(‘Goods’, goodsSchema)
{% endhighlight %}

3. 插入数据
{% highlight js %}
按照以下数据格式往数据库插入数据：
{
    "name":"名字",
    "price":12,
    "location":[经度，纬度]
}
{% endhighlight %}

4. 查看用户附近的数据
{% highlight js %}
goodsModel.find( {
        'location': {
            $nearSphere: [
                parseFloat( 经度 ),
                parseFloat( 纬度 )
            ],
            $maxDistance: 1000
        }
    } ).limit(10).skip(0).lean().exec();
{% endhighlight %}

## 总结
本次主要分享位置索引的用法，十分实用的一个功能，demo没有做得那么手把手，但是主要的骨架是出来了，可以自由发挥～