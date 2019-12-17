---
layout:     post
title:      "WOTPlus(8) - 0.9版本介绍"
subtitle:   "XVM说明一切"
date:       2016-06-26 02:00:00
author:     "Zhangxx"
header-img: "img/post-bg-wotplus-eight.jpg"
catalog: true
tags:
    - WOTPlus
---

> I am back.

##  前言
---
**WOTPlus** 终于更新了一个版本，对于WOTPlus的第一批用户我感到十分的抱歉！因为我更新的实在太慢了，且期间功能出现了很大的问题。个中缘由，我简单的解释下，因为最近辞掉了毕业后的第一份工作，但是由于正处在一个APP项目的开发周期内，而我还是这个项目的核心开发，所以交接的一个月变得异常的忙碌，对于这个项目，一开始我是真不想做，主要是上次已经吃了这个客户的亏，不想再上第二次当，但是没办法，项目组没人了，只好又接手这个明知道会很难验收的项目，由于租的房子的到期原因，我离职的日期定在16号，距今天有10天的时间，这之前根本就没有时间来修改WOTPlus，期间空中网更新了一次战绩页面，改变了成就勋章页面的数据格式，于是直接导致了 **0.6 版本** 的WOTPlus各种报错，除了主页别的页面全部崩溃，也感谢邮件反馈信息给我的玩家，这让我心急如焚，但是没有办法，没有时间来修复这些问题，希望WOTPlus的用户能谅解；

辞职完成后的这些天，我一直在修改WOTPus，毕竟，我是不会对 WOTPlus 撒手不管的，除了之前版本所遇到的bug，我的主要精力还放在 **XVM战绩查询** 的功能上，于是，憋了10几天，终于将WOTPlus提升到了 **0.9 版本**；

最近也有一个《坦克世界手机盒子》的安卓APP上市，对于这个APP我不想做过多的评价，但是能不能把图片弄得清晰一点，这么模模糊糊的真的好么，，，WOTPlus与这个手机盒子的功能还是有区别的，WOTPlus以官网战绩展示与XVM战绩展示为主，盒子APP的主要是盒子的效率，对于数据的权威性我还是更相信XVM，当然我并不是在诋毁盒子APP，我只是在阐述两者的不同与侧重罢了，盒子的战斗实时查询还是不错的，当然我不会打一场看一下，毕竟这太浪费时间，我也曾考虑加上盒子的效率展示，但我考虑的也只是添加一个总的效率数字展示，别的数据未曾想过添加；

我想我说的够清楚了：**WOTPlus** 以 **官网战绩** 和 **XVM战绩** 为主体。

对于之前版本不了解的woter，可以看下这篇介绍：  
[WOTPlus(0) - 绍介](http://amx1390.com/2016/04/30/wotplus-zero/)



## 新增XVM功能截图

_ _ _

![](https://gitee.com/zhangxx0/blog_image/raw/master/wotplus/xvm_show0.jpg)
![](https://gitee.com/zhangxx0/blog_image/raw/master/wotplus/xvm_show1.jpg)


## 新增XVM功能简介
---

**WOTPlus** 所新增的XVM战绩查询主要依据与数据支撑：  

[XVM中文官方站](http://xvm.garphy.com/)  

[新版WOT战绩查询页面](http://182.18.61.50/search.html)   






## 功能展望
---

这个版本并没有把XVM的所有功能都搬移到WOTPlus上来，一些列表的点击展示明细还没有做，这个计划再  **1.0** 大版本中完善；

各位woter如果有好的想法可以email：zxx377241804@gmail.com；也可以直接在WOTPlus中反馈给我，毕竟一个人的力量是有限的。


## 新技术的使用
---
**WOTPlus** 这次更新的不止是外在的新的功能，而且其内在也全部使用了现在Android开发最流行的技术：

- RxJava
- Retrofit
- ButterKnife
- 极光推送SDK

**WOTPlus** 为开源项目：[https://github.com/zhangxx0/WOTPlus](https://github.com/zhangxx0/WOTPlus)


## 下载地址

由于这个打包换了签名的证书，所以0.6版本的用户在下载新版本0.9并替换安装的时候，可能会出现应用未安装的错误，解决办法是卸载掉0.6版本，重新安装新的0.9版本，以后更新1.0版本的时候就不会出现这个问题了，可以直接替换安装；

* Fir.im：[http://fir.im/WOTPlus](http://fir.im/WOTPlus)
* 应用宝：[ WOTPlus-应用宝](http://android.myapp.com/myapp/detail.htm?apkName=com.xinxin.wotplus)
* 魅族应用中心：[ WOTPlus-魅族应用中心 ](http://app.meizu.com/apps/public/detail?package_name=com.xinxin.wotplus)

由于是周末，魅族应用市场都还没有审核通过，所以魅族商店的还是0.6版本，请魅友先使用其他链接下载，估计明天就能审核通过了；



## 关于作者
_ _ _

Java 程序员，Android 菜鸟一枚；

想深入了解作者的可以戳这个：[ Zhangxx的博客-关于 ](http://amx1390.com/about/)  

微博：[http://weibo.com/2112920937](http://weibo.com/2112920937)  
个人博客：[http://amx1390.com](http://amx1390.com)  


差点忘了，土豪要是想打赏我的话，可以扫下面的支付宝(不说了，先去搬砖了)：
![ Zhangxx的支付宝 ](http://7xti0t.com2.z0.glb.clouddn.com/zhifubao)

2016年6月26日16:22:16 好热啊！
