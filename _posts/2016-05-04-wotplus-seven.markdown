---
layout:     post
title:      "WOTPlus(6) - 开发_4"
subtitle:   "WOTPlus开发历程_4"
date:       2016-05-15 01:00:00
author:     "Zhangxx"
header-img: "img/post-wotplus-seven.jpg"
catalog: true
tags:
    - WOTPlus
---


> 马作的卢飞快，弓如霹雳弦惊。了却君王天下事，嬴得生前身后名。可怜白发生！

# APP基础功能

核心功能开发完毕，开始着手APP基础功能开发；

## 关于页面

#### fir.im 检查更新

检查更新逻辑图：
![](http://zhangxx0.gitee.io/blog_image/wotplus/firim.jpg)

使用 **fir.im** 的 [版本查询接口](http://fir.im/docs/version_detection)，编写一个工具方法：`checkVersion` ；通过对比版本号和版本名称来检测是否有更新，若有，则打开一个 `AlertDialog` ，进行下载；若没有，则提示已是最新版本；  
具体代码不想再贴了，没什么技术含量；

#### 使用 NestedScrollView 出现冲突
关于页面使用的是Activity填充Fragment的方式，而Fragment继承自 `PreferenceFragment` ，于是只要配置一个xml文件就可以了；  
起初，我打算在外层的Activity的布局文件中使用折叠的toolbarlayout放个图片好看一些，但是使用 NestedScrollView 包含关于选项时，不能向下滑动到底；  
原因推测：与 `PreferenceFragment` 中使用的listView滑动冲突了；  
[NestedScrollView 之 ScrollView 嵌套 ListView](http://blog.csdn.net/iknownu/article/details/50476023)  
看了一下，没怎么看懂，还是使用普通的简洁**关于**页面吧。

#### 其余的一些功能
应用介绍、个人博客之类，使用了链接`uri`的跳转；  
分享使用了`Intent.createChooser`；  
点赞、打赏使用了 `AlertDialog` ；  
github、邮箱等使用了 `copyToClipboard` 方法，复制到剪切板；

## 设置页面

#### 清除缓存
这个基本上是把 **NBAPlus** 项目的清除缓存功能搬过来的，主要有获取缓存大小和清除缓存两个功能，清除的缓存包括：内部缓存、外部缓存和自定义缓存；  
获取和删除的时候，都使用了 **递归**，保证每个路径下所有文件夹的文件都能遍历到；  

#### 意见反馈
采用发送邮件的方式，将用户的反馈内容以及一些技术信息发到我的gmail邮箱中，为啥不用qq邮箱呢？你懂得。

```
// 发送邮件
String model = android.os.Build.MODEL; // 型号
String brand = android.os.Build.BRAND; // 品牌
String version = android.os.Build.VERSION.RELEASE; // 系统版本
Intent data = new Intent(Intent.ACTION_SENDTO);
data.setData(Uri.parse("mailto:zxx377241804@gmail.com"));
data.putExtra(Intent.EXTRA_SUBJECT, "WOTPlus安卓客户端反馈");
data.putExtra(Intent.EXTRA_TEXT, "\n\n\n技术信息:\n" + "WOTPlus Version-" + CommonUtil.getVersion(getActivity()) + "\n" + brand+" "+model + "\n" + version);
startActivity(data);
```

上线后证明这个功能还是能够获得一些反馈的！

## 后记

所有的功能基本全部完成，接下来是bug修改与美化了；
2016年5月15日18:30:31 by zhang.xx
