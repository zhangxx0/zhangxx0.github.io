---
layout:     post
title:      "SpringCloud(4) - 服务部署"
subtitle:   ""
date:       2022-01-08 00:00:00
author:     "Zhangxx"
header-img: "img/post-bg-wotplus-nine.png"
catalog: true
tags:
    - SpringCloud
---

> 长风万里送秋雁，对此可以酣高楼.

## 前言
---

个人使用的是腾讯云的轻量应用服务器和云服务器，因为单用一台轻应用撑不起安装的这一整套东西，麻雀虽小，五脏俱全；  
轻应用使用的是Docker基础镜像-CentOS7.6-Docker20，感觉还很方便；  

镜像仓库使用的是阿里云的**容器镜像服务**的镜像仓库个人实例，毕竟网易云的改了名，也不给个人开发者用了；  



## 单机Nacos部署

[quick-start](https://nacos.io/zh-cn/docs/quick-start.html)  

```shell
tar -xvf nacos-server-$version.tar.gz
cd nacos/bin
sh startup.sh -m standalone

sh shutdown.sh

netstat -na|grep 8848
```
**注意：** 虽然端口在监听，但是防火墙是屏蔽的，需添加规则  

## RabbitMQ、MySQL、Redis

```shell
docker pull rabbitmq:3.9.13-management

docker run -d --name rabbitmq3.9 -p 5672:5672 -p 15672:15672 -v `pwd`/data:/var/lib/rabbitmq --hostname myRabbit -e RABBITMQ_DEFAULT_VHOST=my_vhost  -e RABBITMQ_DEFAULT_USER=admin -e RABBITMQ_DEFAULT_PASS=admin rabbitmq:3.9.13-management
```

```shell
docker pull mysql

docker run -d -p 3306:3306 -e MYSQL_ROOT_PASSWORD=** mysql:latest
```

```shell
docker pull redis
docker run -itd --name redis-xinxin -p 6379:6379 redis
docker exec -it redis-test /bin/bash
```

## 部署Product项目（商品服务）

编写Dockerfile文件

```Shell
FROM hub.c.163.com/library/java:8-alpine

MAINTAINER xinxin 377241804@qq.com

ADD server1/target/*.jar app.jar

EXPOSE 8082

ENTRYPOINT ["java", "-jar", "/app.jar"]
```

```Shell
docker build -t product:latest .

#push到阿里云仓库：
docker login --username=37724****@qq.com registry.cn-hangzhou.aliyuncs.com
docker tag 22c6879c9438 registry.cn-hangzhou.aliyuncs.com/amx1390/product:latest
docker push registry.cn-hangzhou.aliyuncs.com/amx1390/product:latest

#服务器端拉取：
docker pull registry.cn-hangzhou.aliyuncs.com/amx1390/product:latest

docker run -d -p 8082:8082 registry.cn-hangzhou.aliyuncs.com/amx1390/product


#查看启动日志：
docker logs [id]
```

product启动后，nacos直接挂掉：  
```java
java.net.ConnectException: [NACOS HTTP-POST] The maximum number of tolerable server reconnection errors has been reached
  at com.alibaba.nacos.client.config.http.ServerHttpAgent.httpPost(ServerHttpAgent.java:181) ~[nacos-client-1.4.1.jar!/:na]
  at com.alibaba.nacos.client.config.http.MetricsHttpAgent.httpPost(MetricsHttpAgent.java:68) ~[nacos-client-1.4.1.jar!/:na]
  at com.alibaba.nacos.client.config.impl.ClientWorker.checkUpdateConfigStr(ClientWorker.java:441) ~[nacos-client-1.4.1.jar!/:na]
  at com.alibaba.nacos.client.config.impl
```

尝试在新的云服务器上部署nacos  
新的腾讯服务器使用的并不是Docker基础镜像，需自己安装docker：  
```Shell
curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun

systemctl status docker
systemctl start docker.service
```

将Product项目部署到新的云服务器上，可正常访问；  


## 部署Order项目（订单服务）

基本和部署Product差不多，由于需要连接轻应用上的RabbitMQ，需开放其5672端口，并设置用户权限：`/`  



## 部署Gateway项目（网关服务）

同上；  

部署完毕后的测试链接：  
http://101.43.186.49:8848/nacos  
http://101.43.186.49:15672/  
http://101.43.186.49:9411/zipkin/  

product：  
http://82.156.237.145:8082/product/list  
order：  
http://82.156.237.145:8081/loadbanlanceTest?name=xinxin  
gateway：  
http://82.156.237.145:8088/order/loadbanlanceTest?name=xinxin  


