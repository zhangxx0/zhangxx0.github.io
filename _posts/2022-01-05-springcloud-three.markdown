---
layout:     post
title:      "SpringCloud(3) - 博客大纲"
subtitle:   "Happy tiger year！"
date:       2022-01-05 00:00:00
author:     "Zhangxx"
header-img: "img/post-bg-wotplus-nine.png"
catalog: true
tags:
    - SpringCloud
---

> 黄沙百战穿金甲，不破楼兰终不还.

## 前言
---



## Spring Cloud Sleuth + zipkin

链路追踪

使用Docker方式本地安装Zipkin  
启动命令：docker run -d -p 9411:9411 openzipkin/zipkin  
[Zipkin Quickstart](https://zipkin.io/pages/quickstart.html)  

![](https://gitee.com/zhangxx0/blog_image/raw/master/springcloud/pringcloud-zipkin1.png)  
![](https://gitee.com/zhangxx0/blog_image/raw/master/springcloud/springcloud-zipkin2.png)  

[SpringCloud 2020版本教程4：使用spring cloud sleuth+zipkin实现链路追踪](https://forezp.blog.csdn.net/article/details/115632914)  

## Nacos

![](https://gitee.com/zhangxx0/blog_image/raw/master/springcloud/springcloud-nacos1.png)  

[SpringCloud 2020版本教程1：使用nacos作为注册中心和配置中心](https://forezp.blog.csdn.net/article/details/115632826)  

[官方快速开始](https://nacos.io/zh-cn/docs/quick-start-spring-cloud.html)  
[使用Nacos作为配置中心](https://blog.csdn.net/forezp/article/details/90729945)  

项目中yml配置只需：
```xml
cloud:
  nacos:
    config:
      server-addr: 127.0.0.1:8848
      file-extension: yaml
      prefix: product
    discovery:
      server-addr: 127.0.0.1:8848
```

**注意：** nacos控制台中新建的配置文件名字的后缀应该为yaml，而不是yml；

