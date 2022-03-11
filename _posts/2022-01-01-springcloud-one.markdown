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


## Eureka



## Openfeign

openFeign  感觉像是RPC，实际是远程HTTP；

声明式REST客户端（伪RPC），采用了基于接口的注解

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

使用了Nacos之后，瞬间没了这种烦恼，注册中心和配置中心一体，也不用再gitlab中写配置文件之类，个人感觉Nacos还是更简洁易用，令人神清气爽；  



