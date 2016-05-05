---
layout:     post
title:      "WOTPlus(2) - 开发_0"
subtitle:   "WOTPlus开发历程_0"
date:       2016-05-04 22:00:00
author:     "Zhangxx"
header-img: "img/post-wotplus-three.jpg"
catalog: true
tags:
    - WOTPlus
---
> 假如我要写什么，我根本就不管他格调不格调，正如谈恋爱时我绝不从爱祖国开始谈起。

# 写在前面

以下计划有几篇开发相关的文章，我并不是想做一个教程，而只是想记录下我开发 **WOTPlus** 时所一步步走过的路；  
文章的大体走向按照时间线，开发的顺序除数据获取外基本上是按照功能模块次第进行的；  
文章参照，我的印象笔记 - **坦克世界战绩查询WOTPlus开发日志**

---

# Let's Rock

首先还是 **数据** 的问题，毕竟坦克世界官网不是个开放平台，也没有给开发者使用的数据接口，所以只好自己来找了；  
抽象数据之前，先把工程建立出来，实现类似 **知乎日报** 的放大图片动画的启动页：

### 启动页

放大动画动画效果代码

```
iv_begin.setImageResource(R.mipmap.begin);
final ScaleAnimation scaleAnim = new ScaleAnimation(1.0f, 1.2f, 1.0f, 1.2f,
        Animation.RELATIVE_TO_SELF, 0.5f, Animation.RELATIVE_TO_SELF,
        0.5f);
scaleAnim.setFillAfter(true);
scaleAnim.setDuration(3000);
scaleAnim.setAnimationListener(new Animation.AnimationListener() {
    @Override
    public void onAnimationStart(Animation animation) {
    }

    @Override
    public void onAnimationEnd(Animation animation) {
        startActivity();
    }

    @Override
    public void onAnimationRepeat(Animation animation) {

    }
});
iv_begin.startAnimation(scaleAnim);

```

实现的效果图（*我先学习下怎么制作gif*）：

*弄了好一阵子，命令行、录屏软件之类的都试了，但是在我的MX2上就是不好使，统统闪退，所以先来个静态图先看着，等我有空了再弄动态演示图*

![启动图](http://7xsvfv.com2.z0.glb.clouddn.com/wotplus_001.jpg)

启动图片的尺寸：1080x1920

### 查询页与主页

这里的主页架构参照：Chris Banes 的轮子 [**cheesesquare**](https://github.com/chrisbanes/cheesesquare)  
使用侧滑菜单罗列功能模块；  
![ 侧滑菜单图 ](http://7xsvfv.com2.z0.glb.clouddn.com/wotplus_bolg_01_01.png)

**Navigation的图标问题**   
全部取自 Google 的 [**Material icans**](https://design.google.com/icons/)，免去了自己找或者自己画的痛苦，当然我也不会画，，，

然后开始写首页的UI，首页是一个Card的RecyclerView，所以先写了几个静态的CardView：  
![两个Card](http://7xsvfv.com2.z0.glb.clouddn.com/wotplus_blog_01_02.jpg)  

### 数据的获取

空中网并没有对外暴露玩家信息的json的接口，我用chrome抓了无数遍的包，分析了好久，始终没找到相应的接口， 所以只好从html中获取想要的数据了，这与json接口相比显然是慢了好多，但是这也是没办法的事情。  

大体的思路是：

* 获取玩家信息html字符串；
* 使用jsoup处理字符串并将相应信息对应到实体类 **Woter**
* 使用Woter来进行赋值操作；

首先获取html：  
一开始使用的是httpUrlconnection，但是返回的却是：**getErrorStream  ---错误 403: 权限不足**
应该是header之类的需要设置，但是我比较不习惯使用这个，于是我开始寻找网络框架来做请求；

试用了Volley、 [android-async-http](https://github.com/loopj/android-async-http)，最后选择了Volley，毕竟是Google的官方产品，莫名有种靠谱感；

Volley的使用教程，来自郭神：[Android Volley完全解析(一)，初识Volley的基本用法](http://blog.csdn.net/guolin_blog/article/details/17482095)  

获取到源码时候使用jsoup进行分析：
添加jsoup的依赖： `compile 'org.jsoup:jsoup:1.8.3'`  

jsoup的官方文档：[jsoup: Java HTML Parser](http://www.open-open.com/jsoup/parsing-a-document.htm)  
[使用jsoup解析HTML之获取html源码的几种方法](https://liuzhichao.com/p/1490.html)  

（*2016年5月5日22:47:20 今天先写到这里；*）

#### 概要信息抽取

#### 军团信息获取

#### 抽取成就信息
