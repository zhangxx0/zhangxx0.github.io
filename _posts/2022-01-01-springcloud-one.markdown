---
layout:     post
title:      "SpringCloud(1) - 注册、配置"
subtitle:   "Happy tiger year！"
date:       2022-01-01 00:00:00
author:     "Zhangxx"
header-img: "img/post-bg-wotplus-nine.png"
catalog: true
tags:
    - SpringCloud
---

> 海日生残夜，江春入旧年.

## 前言
---

微服务的特点：异构

Microservice + Docker + Devops

康威定律：  
任何组织在设计一套系统（广义概念上的系统）时，所交付的设计方案在结构上都与该组织的沟通结构保持一致。

CAP定理是由加州大学伯克利分校Eric Brewer教授提出来的，他指出WEB服务无法同时满足下面3个属性：

    + 一致性(Consistency) ：客户端知道一系列的操作都会同时发生(生效)

    + 可用性(Availability) ：每个操作都必须以可预期的响应结束

    + 分区容错性(Partition tolerance) ：即使出现单个组件无法可用,操作依然可以完成

BASE理论

    * Basically Available（基本可用）

    * Soft state（软状态）

    * Eventually consistent（最终一致性）


在微服务构架中，集群服务间的需要调用时就需要知道各个服务的IP和提供服务的端口等信息，如果每个部署一个服务就配置一次，那么必然时非常麻烦的，因此我们需要一个能够统一管理的东西来解决这个问题，由此诞生了注册中心。  
常见的注册中心有Eureka和Nacos；  

各个服务进行注册发现时，在启动类上使用`@EnableDiscoveryClient`注解，以便于更换注册中心也不用修改代码；  


## Eureka

Eureka是Netflix开发的服务发现框架，本身是一个基于REST的服务，主要用于定位运行在AWS域中的中间层服务，以达到负载均衡和中间层服务故障转移的目的。  

2018年7月的时候Netflix就宣布停止维护Eureka了。  


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

**注意：** nacos控制台中新建的配置文件名字的后缀应该为yaml，而不是yml；不对，要与配置中的后缀匹配上就可以；  


作为配置中心，Nacos：
- 保护敏感的配置信息
- 方便的进行动态配置
- 集中管理配置、配置重复利用
- 配置历史版本管理

原理说明：spring-boot启动的时候其实会加载所有的配置文件，最终合并为一份，最终符合激活文件的会最后加载，依此来覆盖其他配置；spring会以这个bootstrap配置文件的优先；

配置文件命名说明：最终组装的`DataID = spring.application.name + ‘-‘ + spring.profiles.active + spring.cloud.nacos.config.file-extension`


## Openfeign

openFeign  感觉像是RPC，实际是远程HTTP；

声明式REST客户端（伪RPC），采用了基于接口的注解，简化了直接使用RestTemplate来调用服务接口的开发量；  


```
Caused by: java.lang.IllegalStateException: No Feign Client for loadBalancing defined. Did you forget to include spring-cloud-starter-loadbalancer?
```
需要注意的是引入openfeign，必须要引入loadbalancer，否则无法启动。  

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-loadbalancer</artifactId>
</dependency>
```



## Config & bus

bootstrap.yml 优先级更高的启动引导配置文件；
优先级高于application.yml

添加依赖：
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-bootstrap</artifactId>
</dependency>
```

SpringCloud Bus 总线；webhook  
这里使用Spring Cloud Bus通过github的webhook功能实现配置的自动刷新时，由于版本更迭等各种原因，导致异常不顺利，要么是缺少依赖了，要么是webhook的url变了之类，同时由于网络等的原因又转到gitee上，最终费了九牛二虎之力才实现这个自动刷新；  

使用了Nacos之后，瞬间没了这种烦恼，一者兼具注册中心和配置中心两个核心功能，也不用再gitlab中写配置文件之类，个人感觉Nacos还是更简洁易用，令人神清气爽；  



