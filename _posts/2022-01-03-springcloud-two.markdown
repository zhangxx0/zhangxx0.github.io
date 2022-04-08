---
layout:     post
title:      "SpringCloud(2) - 网关、熔断"
subtitle:   ""
date:       2022-01-03 00:00:00
author:     "Zhangxx"
header-img: "img/post-bg-wotplus-nine.png"
catalog: true
tags:
    - SpringCloud
---

> 独在异乡为异客，每逢佳节倍思亲.

## 前言
---



## Spring Cloud Stream


> Spring Cloud Stream is a framework for building message-driven microservice applications. Spring Cloud Stream builds upon Spring Boot to create standalone, production-grade Spring applications and uses Spring Integration to provide connectivity to message brokers. It provides opinionated configuration of middleware from several vendors, introducing the concepts of persistent publish-subscribe semantics, consumer groups, and partitions.

个人理解Stream是为了简化消息开发而产生的，基于 **Spring Messaging**、 **Spring Integration** ，通过实现各种消息中间件的 `Binder` 来屏蔽底层消息中间件的实现细节，做到对中间件无感知；  
其支持的中间件也很多，包括主流的RabbitMQ、Kafka、RocketMQ（阿里维护）等等，对于中间件的切换，只需修改配置文件即可；  


## Zuul & Gateway

由于在 **Spring Cloud 2020.0** 版本中， **Spring Cloud Netflix**的模块也移除殆尽，硕果仅存了个Eureka，Zuul也不再推荐，因此不再做深入研究；  
对于其替代品则使用 **Spring Cloud Gateway**，也就是Spring的亲儿子；  

> This project provides a library for building an API Gateway on top of Spring WebFlux. Spring Cloud Gateway aims to provide a simple, yet effective way to route to APIs and provide cross cutting concerns to them such as: security, monitoring/metrics, and resiliency.

应用场景：限流、鉴权、参数校验调整、统计、日志等  

[官方文档](https://docs.spring.io/spring-cloud-gateway/docs/3.1.0-SNAPSHOT/reference/html/#gateway-starter)  
基础功能：动态路由  
[spring cloud gateway之服务注册与发现](https://www.fangzhipeng.com/springcloud/2018/12/23/sc-f-gateway5.html)  
自定义过滤器实现Token等，这里使用了 `GlobalFilter`，实现 `GlobalFilter` 和 `Ordered` 两个接口；   
[spring cloud gateway之filter篇](https://www.fangzhipeng.com/springcloud/2018/12/21/sc-f-gatway3.html)  
使用令牌桶算法的限流拦截器，使用jmeter压测配置了 `RequestRateLimiter` 过滤器的链接，配置100线程，循环间隔1s，部分请求通过，redis中存储了两个key，由此 `RequestRateLimiter` 是使用**redis**进行限流的；线程为10时可能看不到这俩key，因为这俩key是随即销毁的；    
[spring cloud gateway 之限流篇](https://www.fangzhipeng.com/springcloud/2018/12/22/sc-f-gatway4.html)  

![](http://zhangxx0.gitee.io/blog_image/springcloud/springcloud-gateway1.png)

[SpringCloud 2020版本教程2：使用spring cloud gateway作为服务网关](https://forezp.blog.csdn.net/article/details/115632853)  

## Sentinel

**熔断**: 服务发生异常时能够更好的处理;  
**限流**: 限制每个服务的资源（比如说访问量）

同样，由于 **Hystrix** 的不推荐，改用其替代品 **Sentinel**；  

**Hystrix** 采用的是线程池隔离的方式，优点是做到了资源之间的隔离，缺点是增加了线程切换的成本；  
**Sentinel** 采用的是通过并发线程的数量和响应时间来对资源限制。

注意引入时与 **SpringCloudAlibaba** 的版本对应问题：  
> `2020.0` 分支对应的是 `Spring Cloud 2020`，最低支持 `JDK 1.8`。  
[版本对应关系](https://segmentfault.com/a/1190000039853009)

在主pom中引入以下配置：
```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-alibaba-dependencies</artifactId>
    <version>${spring-cloud-alibaba.version}</version>
    <type>pom</type>
    <scope>import</scope>
</dependency>
```

dashboard启动命令：  
```shell
java -Dserver.port=8748 -Dcsp.sentinel.dashboard.server=localhost:8748 -Dproject.name=sentinel-dashboard -jar sentinel-dashboard-1.8.2.jar
```

**Sentinel** 和 **Feign** 的结合使用，以及和 **Gateway** 的限流搭配使用参考：  
[SpringCloud 2020版本教程3：使用sentinel作为熔断器](https://forezp.blog.csdn.net/article/details/115632888)  
[Sentinel/wiki/网关限流](https://github.com/alibaba/Sentinel/wiki/%E7%BD%91%E5%85%B3%E9%99%90%E6%B5%81)  
[入门参考](https://github.com/alibaba/spring-cloud-alibaba/wiki/Sentinel)  

[如何使用 Sentinel starter 完成 Spring Cloud 应用的限流管理](https://github.com/alibaba/spring-cloud-alibaba/blob/2.2.x/spring-cloud-alibaba-examples/sentinel-example/sentinel-core-example/readme-zh.md)  

自定义限流处理逻辑  
[@SentinelResource 注解](https://github.com/alibaba/Sentinel/wiki/%E6%B3%A8%E8%A7%A3%E6%94%AF%E6%8C%81#sentinelresource-%E6%B3%A8%E8%A7%A3)
> blockHandler / blockHandlerClass: blockHandler 对应处理 BlockException 的函数名称，可选项。blockHandler 函数访问范围需要是 public，返回类型需要与原方法相匹配，参数类型需要和原方法相匹配并且最后加一个额外的参数，类型为 BlockException。blockHandler 函数默认需要和原方法在同一个类中。若希望使用其他类的函数，则可以指定 blockHandlerClass 为对应的类的 Class 对象，注意对应的函数必需为 static 函数，否则无法解析。

