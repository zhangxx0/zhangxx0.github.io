---
layout:     post
title:      "RPC（一）-dubbo入门"
subtitle:   ""
date:       2017-11-14 12:00:00
author:     "zhang.xx"
header-img: "img/post-everydayguess.jpg"
catalog: true
tags:
    - RPC
---

> 漫天黄沙掠过，走遍每个角落，行走在无尽的苍茫星河. ——**沙漠骆驼**

---

## 前言
最近进入了新的公司，公司需要，同时也是自己的技术延伸方向，于是来研究下PRC框架dubbo，以及为之后的SOA服务化等做储备，dubbo已经出来好长时间了，据说github还停止维护过，一度导致要用当当的dubbo好像，好在现在是重新活跃的状态，可见dubbo的生命力还是很强的。  
本篇主要还是做一个入门的展现，至于dubbo底层是怎么实现的，什么netty什么的暂时先不考虑，先会用了再来深究。  

## 概览

还是直接整英文吧：  
> Dubbo |ˈdʌbəʊ| is a high-performance, java based RPC framework open-sourced by Alibaba. As in many RPC systems, dubbo is based around the idea of defining a service, specifying the methods that can be called remotely with their parameters and return types. On the server side, the server implements this interface and runs a dubbo server to handle client calls. On the client side, the client has a stub that provides the same methods as the server.

![](http://dubbo.io/images//dubbo-architecture.png)

废话少说，直接上手，代码基于jdk1.7、maven3.X；

下面直接描述demo的搭建过程：  
代码托管在github：[dubbo-demo](https://github.com/zhangxx0/dubbo-demo)

## （1）创建idea Maven项目，并新建3个Module

![](http://zhangxx0.gitee.io/blog_image/java/dubbo-1.png)

## （2）编写providerapi Module代码

**定义接口**：  
```java
  package com.danke.service;

  /**
   * demo接口
   * 2017年11月14日14:25:16
   */
  public interface DemoService {
      String sayHello(String name);
  }
```

## （3）编写provider Module代码
**服务提供者**:  

**添加pom依赖**：  
此处是参照官方的demo的pom引入的依赖，和网上一些教程的做法不一致，暂时还不了解这两种引入方式的利弊：  

```xml
      <dependencies>
        <!--引入api的依赖-->
        <dependency>
            <groupId>com.danke</groupId>
            <artifactId>provider-api</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>

        <!--使用demo的pom配置-->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>dubbo</artifactId>
            <version>2.5.7</version>
        </dependency>
        <dependency>
            <groupId>org.javassist</groupId>
            <artifactId>javassist</artifactId>
            <version>3.20.0-GA</version>
        </dependency>
        <dependency>
            <groupId>org.jboss.netty</groupId>
            <artifactId>netty</artifactId>
            <version>3.2.5.Final</version>
        </dependency>
        <dependency>
            <groupId>org.apache.zookeeper</groupId>
            <artifactId>zookeeper</artifactId>
            <version>3.4.9</version>
        </dependency>
        <dependency>
            <groupId>com.101tec</groupId>
            <artifactId>zkclient</artifactId>
            <version>0.2</version>
        </dependency>
        <dependency>
            <groupId>org.apache.curator</groupId>
            <artifactId>curator-framework</artifactId>
            <version>2.12.0</version>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.31</version>
        </dependency>
        <dependency>
            <groupId>log4j</groupId>
            <artifactId>log4j</artifactId>
            <version>1.2.16</version>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
            <version>1.7.25</version>
        </dependency>
    </dependencies>

```

**在服务提供方实现接口**：
```java
package com.danke.service.impl;

import com.alibaba.dubbo.rpc.RpcContext;
import com.danke.service.DemoService;

import java.text.SimpleDateFormat;
import java.util.Date;

/**
 * demo接口实现类
 * 2017年11月14日14:28:58
 */
public class DemoServiceImpl implements DemoService {

    /**
     * sayHello方法实现
     * @param name
     * @return
     */
    public String sayHello(String name) {
        return "Hello " + name + ", response form provider: " + RpcContext.getContext().getLocalAddress();
    }
}
```

**用Spring 配置声明暴露服务**：  
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://code.alibabatech.com/schema/dubbo http://code.alibabatech.com/schema/dubbo/dubbo.xsd">
    <!-- 提供方应用信息， 用于计算依赖关系 -->
    <dubbo:application name="provider" />
    <!-- 使用zookeeper广播注册中心暴露服务地址 -->
    <dubbo:registry address="zookeeper://127.0.0.1:2181" />
    <!-- 用dubbo协议在20880端口暴露服务 -->
    <dubbo:protocol name="dubbo" port="20880" />
    <!-- 声明需要暴露的服务接口 -->
    <dubbo:service interface="com.danke.service.DemoService" ref="demoService" />
    <!-- 和本地bean一样实现服务 -->
    <bean id="demoService" class="com.danke.service.impl.DemoServiceImpl" />
</beans>
```

**加载 Spring 配置**：  
```java
package com.danke.service;

import org.springframework.context.support.ClassPathXmlApplicationContext;

import java.io.IOException;

/**
 * 注册服务提供者
 * @date 2017年11月14日14:59:00
 */
public class DemoPrividerTest {
    public static void main(String args[]) {
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext(new String[]{"provider.xml"});
        context.start();
        System.out.println("服务提供者已经注册成功！");

        try {
            System.in.read(); // 让此程序一直跑，表示一直提供服务
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

## （4）启动服务提供者，在dubbo admin中查看其是否注册成功

![](http://zhangxx0.gitee.io/blog_image/java/dubbo-2.png)

## （5）编写consumer Module代码
**服务消费者**:  

其pom同样需要引入api项目的依赖；  

**通过 Spring 配置引用远程服务**：  
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://code.alibabatech.com/schema/dubbo http://code.alibabatech.com/schema/dubbo/dubbo.xsd">
    <!-- 消费方应用名， 用于计算依赖关系， 不是匹配条件， 不要与提供方一样 -->
    <dubbo:application name="consumer" />
    <!-- 使用zookeeper广播注册中心暴露发现服务地址 -->
    <dubbo:registry address="zookeeper://127.0.0.1:2181" />
    <!-- 生成远程服务代理， 可以和本地bean一样使用demoService -->
    <dubbo:reference id="demoService" interface="com.danke.service.DemoService" />
</beans>
```

**加载Spring配置， 并调⽤远程服务**:  
```java
package com.danke.service;

import org.springframework.context.support.ClassPathXmlApplicationContext;

/**
 * 服务消费者
 * @date 2017年11月14日15:33:32
 */
public class Consumer {

    public static void main(String args[]) {
        // 加载Spring的配置，并调用远程服务
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext(new String[]{"consumer.xml"});
        context.start();

        DemoService demoService = (DemoService) context.getBean("demoService"); // 获取远程服务代理
        String hello = demoService.sayHello("world"); // 执行远程方法

        System.out.println(hello); // 显示调用结果
    }
}
```


## （6）运行消费提供则代码，在dubbo admin中查看其是否注册成功

![](http://zhangxx0.gitee.io/blog_image/java/dubbo-3.png)  

## 遇到的两个问题：

1.**处理（4）时遇到错误：Caused by: java.lang.ClassNotFoundException: javassist.ClassPath**  
由于是粘贴的demo中的pom配置，但是jar都没有配置version属性，导致好像jar包没有形成依赖，但是Spring的jar自动就形成依赖了，这个地方即上述的那个问题，暂时不知道它是怎么直接引入的，待查；

2.**处理（6）时遇到问题，服务消费者在admin中没有注册记录，并且返回的远程调用结果为空的字符串；这是什么原因呢？**  
服务消费者的pom引入的是api的依赖；这没有问题;  
将DemoProviderTest即注册服务提供者的class点击红方块关闭之后，重新编译代码，并调用，得到正确的返回结果；  

## 几个疑问？？？
1.**官方的demo中，为什么要给提供者再做一个API的项目，然后在提供者工程中导入API工程的依赖，为什么要分为两个工程来实现这个功能呢？ 这样有什么优点?**  
![](http://zhangxx0.gitee.io/blog_image/java/dubbo-4.png)

2.**注册中心和监控中心都是可选的， 服务消费者可以直连服务提供者那为什么还要使用注册中心呢，有什么好处吗？**  
[zookeeper在Dubbo中扮演了一个什么角色，起到了什么作用啊？](https://www.zhihu.com/question/25070185)  
还是不是很理解  
[为什么要用zookeeper?](https://m.2cto.com/kf/201708/668587.html)  
通讯录的解释比较通俗易懂；  

## zookeeper的下载与启动

[ Zookeeper注册中心的搭建](http://blog.csdn.net/u013142781/article/details/50395650)

## dubbo admin的编译与本地启动

[dubbo-admin管理平台搭建](http://blog.csdn.net/u013142781/article/details/50396621)  

dubbo的使用，其实只需要有注册中心，消费者，提供者这三个就可以使用了，但是并不能看到有哪些消费者和提供者，为了更好的调试，发现问题，解决问题，因此引入dubbo-admin。通过dubbo-admin可以对消费者和提供者进行管理。

## dubbo monitor的配置
```java
  TODO
```

## dubbo的原理简述
```java
  TODO
```

## 总结与参考
这里只是简单的一个dubbo入门的例子，至于dubbo怎么和SSM进行结合编程，以及之后与MQ等框架的结合，需要进一步的学习；

2017年11月16日00:24:54
昨天早上，也就是15号，上班之后看到阿里技术公众号发了一篇文章：  
[分布式服务框架Dubbo疯狂更新！阿里开源要搞大事情？](https://mp.weixin.qq.com/s?__biz=MzIzOTU0NTQ0MA==&mid=2247486338&idx=1&sn=e5decc5fe99c4cfe46db8ea5e7e12701&chksm=e929368dde5ebf9bc031a3c18dbe9884b46d91053cf2928a69589a2bed7a7732d65a61236975&mpshare=1&scene=23&srcid=1115z5kFIvEREYhmh4BMCW98#rd)  


参考：

[Dubbo分布式服务框架入门（附工程）](https://www.kancloud.cn/digest/javaframe/125576)  
[DUBBO](http://dubbo.io/)  
[github-dubbo](https://github.com/alibaba/dubbo)  
[dubbo-user-book.pdf](http://dubbo.io/docs/dubbo-user-book.pdf)     


2017年11月-by zhang.xx
