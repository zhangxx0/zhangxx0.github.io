---
layout:     post
title:      "docker初始"
subtitle:   "Happy tiger year！"
date:       2022-01-10 00:00:00
author:     "Zhangxx"
header-img: "img/post-bg-wotplus-nine.png"
catalog: true
tags:
    - Docker
---

> 黄沙百战穿金甲，不破楼兰终不还.

## 前言
---

学习**Docker**，主要是为了节约时间；  
一次创建或配置，可以在任意地方正常运行，我认为这是Docker的核心所在；  


## Docker入门

Docker 包括三个基本概念

 - 镜像（Image）类比Java中的类
 - 容器（Container）类比Java类的实例
 - 仓库（Repository）集中的存储、分发镜像的服务

```shell
# helloworld
docker pull hello-world
docker images
docker run hello-world

# 查看当前有哪些容器正在运行  
docker ps
docker ps -a

# 停止容器  
docker stop **（id前缀）

# 删除镜像  
docker rm container_id
docker rmi image_id
```

## Dockerfile

Dockerfile 是一个文本文件，其内包含了一条条的 指令(Instruction)，每一条指令构建一层，因此每一条指令的内容，就是描述该层应当如何构建。

所谓定制镜像，那一定是以一个镜像为基础，在其上进行定制。就像我们之前运行了一个 nginx 镜像的容器，再进行修改一样，基础镜像是必须指定的。而 FROM 就是指定 基础镜像，因此一个 Dockerfile 中 FROM 是必备的指令，并且必须是第一条指令。

RUN 指令是用来执行命令行命令的。由于命令行的强大能力，RUN 指令在定制镜像时是最常用的指令之一。其格式有两种：
- shell 格式：RUN <命令>，就像直接在命令行中输入的命令一样。刚才写的 Dockerfile 中的 RUN 指令就是这种格式。
- exec 格式：RUN ["可执行文件", "参数1", "参数2"]，这更像是函数调用中的格式。

构建镜像：  
在Dockerfile所在目录执行docker build创建镜像  

`docker build -t jpress:latest . `

将制作的镜像上传到private registry  

`docker tag test docker.example.com/test
docker push docker.example.com/test`


**上传到阿里云镜像仓库：**   

```shell
# 登录阿里云Docker Registry
docker login --username=37724****@qq.com registry.cn-hangzhou.aliyuncs.com

# 从Registry中拉取镜像
docker pull registry.cn-hangzhou.aliyuncs.com/amx1390/product:[镜像版本号]

# 将镜像推送到Registry
docker login --username=37724****@qq.com registry.cn-hangzhou.aliyuncs.com
docker tag [ImageId] registry.cn-hangzhou.aliyuncs.com/amx1390/product:[镜像版本号]
docker push registry.cn-hangzhou.aliyuncs.com/amx1390/product:[镜像版本号]
```

![](https://gitee.com/zhangxx0/blog_image/raw/master/dockerbigdata/springcloud-server-alibabadocker.png)


## Docker Compose

**Defining and running multi-container Docker applications**  

```
# 目前使用过的命令
docker-compose up
docker-compose down
```




**参考** 
[Docker 从入门到实践](https://vuepress.mirror.docker-practice.com/)  
[美团Docker相关文章](https://tech.meituan.com/tags/docker.html)  
[配置阿里云镜像加速](https://blog.csdn.net/lizy0327/article/details/114024916)  
 



