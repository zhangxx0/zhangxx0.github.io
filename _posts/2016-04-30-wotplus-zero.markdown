---
layout:     post
title:      "WOTPlus(0) - 绍介"
subtitle:   "坦克压倒一切"
date:       2016-04-30 02:00:00
author:     "Zhangxx"
header-img: "img/post-bg-wotplus-zero.jpg"
catalog: true
tags:
    - WOTPlus
---

> 中国一向就少有失败的英雄，少有韧性的反抗，少有敢单身鏖战的武人，少有敢抚哭叛徒的吊客；见胜兆则纷纷聚集，见败兆则纷纷逃亡。

## 前言
---
**WOTPlus** 是一款 **坦克世界**（World of Tanks）战绩查询APP，旨在为国服坦克世界玩家提供一个方便且专业而又不失优美的战绩查询APP；    

作为一个坦克世界的老司机，我还是比较关心自己的战绩与排名，然而并没有一款好的APP来让我实时的掌握我的战绩动态；  

与其等待，不如自己动手，且好几年过去了，并不见坑爹网（空中网）的APP有些许的进步，国服对这个好像不怎么重视，外服倒是有几个不错的应用，像**"World of tanks Assistant"**，但是这个软件不给国服的玩家提供服务，虽然都说外服好，国服我认为也不错啊，毕竟我是用来娱乐的，而坦克世界也带给了我好久的欢愉了，于是，在学习了半年左右的Android之后，我动手来做人生第一个开源安卓项目，我想用可以接触到的最先进的安卓技术来开发这款APP，因为我觉得这是一件很酷的事情，而，上班以来，好久没有做过很酷的事情了。  

## 截图

_ _ _

![](http://zhangxx0.gitee.io/blog_image/wotplus/wotplus_xiaoguo2.jpg)
![](http://zhangxx0.gitee.io/blog_image/wotplus/wotplus_xiaoguo1.jpg)
![](http://zhangxx0.gitee.io/blog_image/wotplus/wotplus_xiaoguo3.jpg)
![](http://zhangxx0.gitee.io/blog_image/wotplus/wotplus_xiaoguo4.jpg)
![](http://zhangxx0.gitee.io/blog_image/wotplus/wotplus_xiaoguo5.jpg)

## 功能简介
---

WOTPlus主要提供坦克世界战绩查询的功能，分为以下几个模块展示战绩信息：

- 概要信息
- 成就
- 统计
- 等级
- 战车

## 功能展望
---

第一个版本的功能比较简单，根据昵称来查询玩家的战绩信息，但我显然没有满足于此，因此还有好多的工作要做，当然，我是心甘情愿的：  

* 效率的查询（这个的数据源还没有找，不确定是否能实现，但是效率党也是重要的一党啊，毕竟wot不是一党专政）；
* **XVM**战绩查询（计划使用XVM的json接口）；
* 排行榜；
* 军团信息；

暂时有以上四个想法，各位woter如果有好的想法可以email：zxx377241804@gmail.com；也可以直接在WOTPlus中反馈给我，毕竟一个人的力量是有限的。


## 技术阐述
---

WOTPlus遵循 **Material Design** 设计原则，使用最新的控件，诸如：DrawerLayout、CoordinatorLayout、Toolbar、FloatingActionButton、NavigationView、RecyclerView、CardView等

**数据源**：由于坦克世界官网的战绩查询页面并没有暴露json接口，所以我使用的是获取战绩页面html并使用jsoup解析数据，实属无奈之举，也造成查询时耗时较长；

开源库：

* html解析：jsoup；
* json解析：gson；
* 网络请求：Volley；  
* 图片加载：Glide；
* 图表展示：MPAndroidChart；
* gif展示：android-gif-drawable；
* materialedittext -by 扔物线；
* 左滑回退：swipebacklayout；
* RecyclerView动画：recyclerview-animators；

第一版并没有使用架构诸如MVP、或者Rxjava等流行技术之类，别问我为什么，还没学会。。。因此编写之初就有了重构的计划，初步构想是使用Rxjava+Retrofit进行第二个版本的重构

## 版本更新

* Fir.im：[http://fir.im/WOTPlus](http://fir.im/WOTPlus)
* 魅族应用中心：[ WOTPlus-魅族应用中心 ](http://app.meizu.com/apps/public/detail?package_name=com.xinxin.wotplus)
* 应用宝：[ WOTPlus-应用宝](http://android.myapp.com/myapp/detail.htm?apkName=com.xinxin.wotplus)

或者直接扫描下面的二维码下载：  

![二维码分发](http://zhangxx0.gitee.io/blog_image/wotplus/%E4%BA%8C%E7%BB%B4%E7%A0%81%E5%88%86%E5%8F%91.png)


## TODO

- [ ] 代码重构：RxJava+Retrofit；  
- [ ] fresco 替代 glide；
- [ ] butterknife使用；

## 关于作者
_ _ _

一说关于作者就想胡吹海吹一番，矜持，矜持，，，  
作者是一个竞技水平比较低的玩家，说来惭愧，从小到大玩的比较好的游戏有两个，一个是《坦克世界》，另一个就是《扫雷》了；    
然而，作者也是有一颗拯救世界的心的！  
想深入了解作者的可以戳这个：[ Zhangxx的博客 | 关于 ](http://amx1390.com/about/)  

微博：[http://weibo.com/2112920937](http://weibo.com/2112920937)  
个人博客：[http://amx1390.com](http://amx1390.com)  


差点忘了，土豪要是想打赏我的话，可以扫下面的支付宝(不说了，先去搬砖了)：
![ Zhangxx的支付宝 ](http://zhangxx0.gitee.io/blog_image/amx1390/zhifubao)


2016年4月30日23:15:18 by [zhangxx](http://amx1390.com)
2016年5月6日22:17:40 update  
