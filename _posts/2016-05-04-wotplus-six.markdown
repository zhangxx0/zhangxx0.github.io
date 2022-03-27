---
layout:     post
title:      "WOTPlus(5) - 开发_3"
subtitle:   "WOTPlus开发历程_3"
date:       2016-05-14 24:00:00
author:     "Zhangxx"
header-img: "img/post-wotplus-six.jpg"
catalog: true
tags:
    - WOTPlus
---


> 运交华盖欲何求？未敢翻身已碰头。　　破帽遮颜过闹市，漏船载酒泛中流。　　横眉冷对千夫指，俯首甘为孺子牛。　　躲进小楼成一统，管他冬夏与春秋。

# 承上启下

写到这里，就剩下一个功能了，那就是 **战车** 了；

## 战车页面

关于战车页面的设计与数据提取；  
先来提取数据，看看获取的数据都有什么；然后再定页面的展示；  
初步的规划是三个级联页面；  

前两个页面，也就是 **战车类型列表** 和 **各类型战车列表** 的数据在之前的html中，仍然使用jsoup进行提取，比提取到对应的抽象类 `Tanks`，这个地方也比较复杂，代码比较多，我也不贴了，可以去源码中的工具类中去看；  
第三个页面 **战车战绩信息** 的数据是没有，继续抓请求，发现点击某个战车的时候回发出一个 json 请求，这就好办了；

类似下面的请求，需要参数acount_id和vehicle_cd：  
[http://ncw.worldoftanks.cn/zh-cn/community/accounts/1509154099/vehicle_details/?vehicle_cd=305](http://ncw.worldoftanks.cn/zh-cn/community/accounts/1509154099/vehicle_details/?vehicle_cd=305)

返回的 json 包括该战车的成就list，和其他的战绩信息；

**一些技术细节**  

第一个 **战车类型列表** 页面使用的是 `TanksFragment` 填充主页的空白，后两个则是使用的新的 Activity 来实现的，分别为：`AtyTanks` 和 `AtyTank`；  
`AtyTanks` 中根据 `TANKS_TYPE` 来判断展示的坦克类型名称，名称使用了一个数组来存储：`String[] TANKS_TITLE = {"LT", "MT", "HT", "TD", "SPG"}`，顺序与列表的点击 `position` 对应；  
在 `AtyTanks` 的页面文件 `activity_tanks` 中使用了 `CollapsingToolbarLayout` ，其中添加了一个代表各类型的坦克图片，我选的德系比较多，，，谁让我是个德粉呢！  
在 `AtyTank` 的页面文件 `activity_tank` 中的上半部分使用了 `NestedScrollView` 嵌套 `CardView` 再嵌套 `RecyclerView` 的方式，在可上下滚动的Card里实现了成就列表的左右滚动；  

其他的一些 **逻辑处理** ，如：国旗的处理、坦克成就的图片url拼接、提取勋章的ID与名称对照字段map（坦克成就json中没有成就名字汉字，于是拼装了一个中英文对照的map，此map用于以英文为id提取中文从而设置成就名称）等，不再详细描述，具体可参见代码；  


 **战车类型列表** 图：  
![](http://zhangxx0.gitee.io/blog_image/wotplus/tankpage11.jpg)

**各类型战车列表** 图：  
![](http://zhangxx0.gitee.io/blog_image/wotplus/tankpage22.jpg)

**坦克详细战绩页面** 图：  
![](http://zhangxx0.gitee.io/blog_image/wotplus/tankpage3.png)


## 两个问题

#### 1、上滑时toolbar只隐藏一半的问题

错误图：  
![](http://zhangxx0.gitee.io/blog_image/wotplus/tankproblem2.jpg)


[ 参照 toolbar的使用 ](https://guides.codepath.com/android/Handling-Scrolls-with-CoordinatorLayout#floating-action-buttons-and-snackbars)

原因：在 `CoordinatorLayout` 中多了下面这个属性：
`android:fitsSystemWindows="true"`


#### 2、网络请求又出问题了，，，不返回jsonobj

去掉好多header后可以获取到String，header还是不要乱加，够用就好；

## 后记

核心功能基本完成，下一步开发APP的基础功能：设置、关于；  
主要的功能点：版本更新、缓存清除等；  

明明还有好多要干的，可是不知从何下手了；

2016年5月15日16:12:23 by zhang.xx
