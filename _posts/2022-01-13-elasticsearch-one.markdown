---
layout:     post
title:      "ElasticSearch 昨日重相逢"
subtitle:   "Happy tiger year！"
date:       2022-01-13 00:00:00
author:     "Zhangxx"
header-img: "img/post-bg-wotplus-nine.png"
catalog: true
tags:
    - java进阶
---

> 你相信死後得救
> 你是國家的槍手
> 我相信死後自由
> 我是自己的石頭


## 前言
---

初接触`Elasticsearch`是在蛋壳公寓的时候做搜房引擎，现在蛋壳早已倒闭了，原先的官网、APP也早已不再提供服务，想起当年那个还和CEO高靖一起做游戏搞活动之类的画面，还有年会那次大合影，以及后来我没有经历过的在新闻里看到蛋壳上市的新闻，一时风光无两，再后来，楼塌了，，，蛋壳也瞬间土崩瓦解，蒂芙尼蓝来带来不了好运气，可怜的不是高靖，而是那些被他骗了的租客，空手套白狼的骗子终究逃不过法网，与还逍遥国外声称下周回国的假药停比，高靖或许还不够高明；这些也不过茶余饭后的谈资罢了，这些人掌握过巨额的财富，有过呼风唤雨的时刻，但是终究逃不过苟且，我不认为这也算是辉煌的一生，人类群星闪耀时，也不会出现这些小丑的影子。  

当初入职蛋壳，部门搞java的只有部门的主管，我也算是部门里干活的第一个java工程师了，当时蛋壳的技术团队后端基本是PHP的，他们的搜索竟然是基于一个大字段存储大量关键字模糊来实现的，后来随着架构师的到来，才正式开始搜索引擎的开发，所有的搜索代码几乎都是我一个人搞的，当时还用了`JestClient`，现在GitHub上这个库也早已不维护了；

现在重拾ES，继续感受下ES带来的无与伦比的搜索体验；  
使用Elasticsearch的版本：`7.17.1`  

[JEST - This project is no longer being actively developed](https://github.com/searchbox-io/Jest)  

## 安装

一开始直接按照官方文档用`Docker Compose`安装了一个三节点的集群，up之后报错：  
`max virtual memory areas vm.max_map_count [65530] is too low`，参照[max_map_count问题解决](https://segmentfault.com/a/1190000038912881)解决  
然后三个节点总有一个启动失败，又尝试调整docker的内存设置，同时上面那个设置机器重启后失效，也写到下面的文件中：  
修改windows用户根目录下的`.wslconfig`文件：  
```
[wsl2]
memory=4GB
swap=0
localhostForwarding=true
kernelCommandLine = "sysctl.vm.max_map_count=262144"
```

[docker容器 Exited (137)错误代码](https://blog.haohtml.com/archives/18981)  
[Advanced settings configuration in WSL](https://docs.microsoft.com/en-us/windows/wsl/wsl-config#configure-global-options-with-wslconfig)  
[Using Docker-Desktop for Windows, how can sysctl parameters be configured to permeate a reboot?](https://stackoverflow.com/questions/69214301/using-docker-desktop-for-windows-how-can-sysctl-parameters-be-configured-to-per/69294687#69294687)  

后来又需要使用`ik`分词器，于是又修改了docker-compose文件，并添加Dockerfile文件：  
```shell
FROM docker.elastic.co/elasticsearch/elasticsearch:7.2.0

ENV VERSION=7.2.0

# https://github.com/medcl/elasticsearch-analysis-ik/releases
# 中文分词插件下载
ADD https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v${VERSION}/elasticsearch-analysis-ik-$VERSION.zip /tmp/
# 中文分词插件安装
RUN /usr/share/elasticsearch/bin/elasticsearch-plugin install -b file:///tmp/elasticsearch-analysis-ik-$VERSION.zip

RUN rm -rf /tmp/*
```
将elasticsearch和ik打成新的镜像，并进行集群的安装。  


参照：  
[ Install Elasticsearch with Docker](https://www.elastic.co/guide/en/elasticsearch/reference/7.17/docker.html)  
[使用Docker搭建本地ES集群加kibana和IK中分分词插件](https://www.jianshu.com/p/ed66670e9e7a)  
[Elasticsearch —— docker部署+ik分词器(**未使用**)](https://www.jianshu.com/p/d8b0c736070f)  


## Removal of mapping types

尝试那之前的创建索引语句去执行创建，一执行就报错，经过一番查找：  

In Elasticsearch 7, **mapping types have been deprecated**, which is causing the breaking change at the source of this problem.  

它删除映射类型从版本6开始，6只能创建一个type，到7版本完全弃用掉，纠正了原先将`type`对应为`表`的错误类比，根源在于ES的索引中，不同`type`中具有相同名称的字段在内部由相同的`Lucene字段`支持，这样的话，当你想把一个字段在同一个index中的一个type中创建为`date`类型，另一个type中创建为`boolean`类型的时候，这是无法实现的；（这里绝大多数的中文翻译将原文中的`deleted`翻译为删除，实际这个单词为字段的名称，并无删除之意，还是得自己看官方文档，此解释引用如下）；  

最重要的还不是上面的原因，而是将很少或者没有相同字段的实体存储在同一个index中，会导致数据稀疏并影响Lucene压缩文档的能力。  

ES的这个修改还是挺有意思的，纠正了错误的类比，ES并不是SQL数据库那样的结构。  

文档引用：  
> This can lead to frustration when, for example, you want deleted to be a date field in one type and a boolean field in another type in the same index.

[Removal of mapping types](https://www.elastic.co/guide/en/elasticsearch/reference/7.17/removal-of-types.html)  
[Elasticsearch 参考指南（删除映射类型）**含有错误翻译**](https://segmentfault.com/a/1190000019911538)  


## ES原理

## Java调用ES的方式

- Spring Data Repositories
- RestClient(WARNING: Deprecated in 7.15.0.)
- Java API Client

第一种，需引入：`Spring Data Elasticsearch`，可继承`CrudRepository`、`PagingAndSortingRepository`等，直接通过方法名语义来实现功能：

比如：**查询 findByFirstnameLike** findBy+需要查询字段名+操作 

也可以使用`@Query`注解实现高级查询，就像使用MyBatis时在mapper上用`@Select`注解一样，个人很不喜欢这样的方式，那些反斜杠看的我头大。  

[Spring Data Elasticsearch 官方文档](https://docs.spring.io/spring-data/elasticsearch/docs/current/reference/html/#preface.versions)  

第二种，一般在做高级搜索的时候使用，但是在7.15版本，这种方式被弃用了，取而代之的是官方的Java API Client，解释是：  
> The Java REST Client is deprecated in favor of the Java API Client.  

这个解释也算是让人看了很蛋疼的，虽然它也提供了一些解决方法从`HLRC`迁移到Java API Client上来，但不改代码是不存在的，这总是让人恼怒，就跟刚学会了`hystrix`Spring Cloud就把它淘汰了一样，直到使用了后者之后，我才改变了这个看法。  

第三种，基于`elasticsearch-java`包以及`jackson`，我使用这个API做了个组合查询，代码如下：  
```java
SearchResponse<BossJobIndex> search = client.search(s -> s
                        .index(BOSS_JOB_INDEX)
                        .query(q -> q
                                .bool(b -> b
                                        .must(must -> must
                                                .multiMatch(m -> m
                                                        .fields("name", "companyName")
                                                        .query(bossJobSearchDto.getKeyword())
                                                )

                                        )
                                        .must(must -> must
                                                .term(t -> t
                                                        .field("finance.keyword")
                                                        .value(v -> v.stringValue(bossJobSearchDto.getFinance()))
                                                )
                                        )

                                )
                        ),
                BossJobIndex.class);
```

看到这样的流式编程代码不禁心情愉悦起来，比之前的RestClient不知道高到那里去了，逻辑也更加清晰，不得不说，真香~


[Elasticsearch8.0版本中Elasticsearch Java API Client客户端的基本使用方法](https://blog.csdn.net/anjiongyi/article/details/123391835)  

Search微服务源码：
[搜索引擎服务 search](https://github.com/zhangxx0/search)  