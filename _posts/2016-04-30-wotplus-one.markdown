---
layout:     post
title:      "WOTPlus(1) - 由来"
subtitle:   "风暴降生的WOTPlus"
date:       2016-04-29 03:00:00
author:     "Zhangxx"
header-img: "img/post-wotplus-one.jpg"
catalog: true
tags:
    - WOTPlus
---

> 寄意寒星荃不察，我以我血荐轩辕

# WOTPlus 的历史渊源

做一个坦克世界APP的想法很早就有了，然而由于个人技术的限制，拖到今天才得以有一个基础的版本；  

我从大二开始玩坦克世界（**WOT**），细想一下，竟然有1415天了（这个是WOTPlus算出来的，官网并没有），马上就4年了，很难想想我这么一个三分钟热度的人能坚持这么久的玩一款游戏，这本身也因为我从小学开始基本上就是个军迷，那个时候各种杂志的看，还记得当初看的第一本军事杂志是《兵器》，还是过期好久的，但即使这样还是一眼着了迷，然后就是各种杂志书的看，我那时候的一大爱好是造船模，基本每个假期都会造一艘，这好像和坦克没什么关系哎，，，扯远了，主要是坦克比较难弄，我所造过的比较复杂的是一个双联舰炮，可以遥控发射火箭弹（天老鼠）的，然后到高中又渐渐迷上了搏击，军事相关的渐渐远了一些，但是始终没有放下，后来在大学玩的一些游戏也是战争题材，譬如：《使命召唤》系列、《狙击精英》等；

直到大二遇上了坦克世界，便一发不可收拾的爱上了他，当然我是冲着德系去的（*哭晕在厕所*），，，我记得我开始玩的版本是7点几来，名字倒是记住了 **前进乌拉** ，然后就开始了坑比之路，当时我是万也没想到能达到今天的水平，一直坑一直坑，直到后来出来了一个叫做 **效率** 的东西，才渐渐意识到自己的水平之洼难以想象，也就是现在常说的小学生水平，虽然当时是在上大学，，，

后来，随着场次的增加，自然也就渐渐的会玩了，效率也在稳步的上升，当时我们学校坦克世界很繁荣，还有个军团叫做 **陆虎** 可惜后来改名了，，，当时还是铁狼团长带队训练的，然而我训练了几次就再也不去了，我还是喜欢单野，或者和同学组队，陆续发展了好几个同学玩坦克，也直接导致了我们整个周末都在组队玩坦克。

列一下我们专业woter的昵称：

* 康恩饭_
* 打错了_
* 风吹菊花灬姐
* 五星de巴顿
* 紫雾怨影
* Gost_spark
* so_what_so
* 丶sevn
* 夕阳西下

随着效率的增长，我越来越关心自己的战绩，后来空中网出了战绩查询页面，譬如说我的：  
[康恩饭_ 的战绩页面](http://ncw.worldoftanks.cn/zh-cn/community/accounts/1509154099-%E5%BA%B7%E6%81%A9%E9%A5%AD_/)  

我是几乎每天都要看下自己的战绩的，后来又出了个人排名，于是可以查到自己在国服的总排名了，我现在是在全服7千多名，效率1200+，也算是个合格的玩家了，再加上我主玩德系，最近刚买了老鼠，而坦克世界流行的就是：苏系太强了，我们削一下德系吧！所以我认为我的综合水平还是可以的，哎呀，怎么又吹上b了呢，，，不好意思，一天不装比就过不了这一天啊。

然而，

却没有一款坦克世界的APP来进行战绩的查询以及其它，之前官网出了个APP，在上面能领任务，这两年也不更新，以至于销声匿迹了，就算下载了也没啥用；
还有一个 **坦克世界知识库** 的APP，这是个数据党性质的，而我想查自己的战绩却无能为力，而且这个APP的传播之闭塞可以用落后来形容，我知道和下载它是在 百度贴吧 里，这，，，好难找啊，而且是放在网盘上下载，就不能上传到商店么，，，或许有别的难言之隐的，这里就不再揣测了；  

除了坦克世界官网，我还会在XVM上查询战绩，这个也比较专业，每场的战斗数据都有，但是XVN主要是做web与战争沙盘的，对于手机的APP没什么涉猎，不过一些接口还是挺好的，我当初的想法就是把XVM的战绩查询做到手机上来，还专门加了战争沙盘的群，结果里面竟是吹比，我这么说似乎有些诽谤了，但是，那确实不是个讨论技术的地方，于是过了一阵我被踢了，，，

再者就是坦克世界官网的微信账号了，里面有查询战绩的页面，之前的一个版本是很简单，信息量太少；后来五周年换了一个版本，结果数据却是来自《捞月狗》，这，，，这也比较扯淡，自己的数据自己处理不好，反倒成了从别人的地方拿来的了，，，手动鄙视

于是我就不淡定了，分析了下官网的战绩页面，刷刷刷就画了个APP的”设计图"，如下：
![ WOTPlus手绘设计图 ](http://zhangxx0.gitee.io/blog_image/amx1390/wotplus-design-hand2.png)

把整个战绩页面分成了一个APP的样子，甚至在上么标注了那个页面用什么控件实现，，，  
实在是有些low，当时也是想做一个快速原型嘛，也管不了那么多了；

接下来的事情就是边学习Android边开发WOTPlus了。

# WOTPlus 开发历程

## 时间点：

* 项目始于：**2016年3月19日** 10:30:04 周六（*偷偷看了下开发日志*）
* 基本完成：**2016年4月14日**
* 美化优化：**2016年4月24日**
* 准备上线：**

期间我在网上各种找资料，结果发现了坦克世界外服的APP：  
**World of Tanks Assistant**
**World of Tanks Blitz Assistant**

[Google Play ](https://play.google.com/store/apps/dev?id=7063148299497943432)

安装了这两个APP之后，惊为天人！感叹自己还是太嫩了，别人的设计，别人的效果！我只能用很牛笔来形容，当时我的设计页面基本定型并完成了一半左右的开发，所以还是按照之前的设计稿来实现，当然与这两个APP有些地方还是挺像的，毕竟有时候，英雄所见略同嘛！

然后，我就把这俩APP解压、反编译，获得了好多的图片资源，嘻嘻嘻。

# 其他
---

这是 WOTPlus 系列的开篇，主要还是扯淡了，，，之后我将参照我的开发日志将整个开发过程写出来，并与此同时完成最后的优化与上架，并尽我所能的推广，毕竟是自己的第一个APP，就像养了个孩子一样，当然要对她好好的啦！

以上。

2016年4月30日00:52:58 by [zhangxx](http://amx1390.com)
