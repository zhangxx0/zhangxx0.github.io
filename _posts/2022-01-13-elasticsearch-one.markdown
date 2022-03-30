---
layout:     post
title:      "Elasticsearch 昨日重相逢"
subtitle:   ""
date:       2022-01-13 00:00:00
author:     "Zhangxx"
header-img: "img/post-bg-wotplus-nine.png"
catalog: true
tags:
    - Java进阶
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

![残存的适配不了2k的蛋壳官网](http://zhangxx0.gitee.io/blog_image/elasticsearch/elasticsearch-danke.png)  


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
[Install Elasticsearch with Docker](https://www.elastic.co/guide/en/elasticsearch/reference/7.17/docker.html)  
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

Elasticsearch是实时(**Near Realtime**(NRT))全文搜索和分析引擎，提供搜集、分析、存储数据三大功能；是一套开放REST和JAVA API等结构提供高效搜索功能，可扩展的分布式系统。构建于Apache Lucene搜索引擎库之上。

  ### 倒排索引
  
  倒排索引（Inverted Index）也叫反向索引，有反向索引必有正向索引。通俗地来讲，正向索引是通过key找value，反向索引则是通过value找key。

  倒排索引服务于es查询操作，对数据的聚合、排序则需要使用正排索引。  

  建立倒排索引前，会根据分词器对每个单词进行`normalization`（时态转换，单复数转换）；  

  注意：
  对于分词的field进行聚合（aggregation）操作，需要将fielddata设置为true，否则会报错提示你打开fielddata、将正排索引加载到内存中

  ### 一个field索引两次来解决字符串排序问题
  
  将一个string field建立两次索引，一个分词，用来进行搜索；一个不分词，用来进行排序  

  ```java
    "finance": {
            "type": "text",
            "fields": {
              "keyword": {
                "ignore_above": 256,
                "type": "keyword"
              }
            }
          }

  ```

  ### 相关度评分TF&IDF算法
  
  Elasticsearch使用的是 term frequency/inverse document frequency算法，简称为TF/IDF算法；
  - Term frequency：搜索文本中的各个词条在field文本中出现了多少次，出现次数越多，就越相关；
  - Inverse document frequency：搜索文本中的各个词条在整个索引的所有文档中出现了多少次，出现的次数越多，就越不相关；


  ### 分布式架构原理
  
  **shard和replica**：
  
  单台机器无法存储大量数据，ES 可以将一个索引中的数据切分为多个 shard，分布在多台服务器上存储。有了 shard 就可以横向扩展，存储更多数据，让搜索和分析等操作分布到多台服务器上去执行，提升吞吐量和性能。每个 shard 都是一个 lucene index。  

  任何一个服务器随时可能故障或宕机，此时 shard 可能就会丢失，因此可以为每个 shard 创建多个 replica 副本。replica 可以在 shard 故障时提供备用服务，保证数据不丢失，多个 replica 还可以提升搜索操作的吞吐量和性能。primary shard（建立索引时一次设置，不能修改，默认 5 个），replica shard（随时修改数量，默认 1 个），默认每个索引 10 个 shard，5 个 primary shard，5 个 replica shard，最小的高可用配置，是 2 台服务器。 shard 分为 primary shard 和 replica shard。而 primary shard 一般简称为 shard，而 replica shard 一般简称为 replica。

  ![](http://zhangxx0.gitee.io/blog_image/elasticsearch/es-cluster.png)  

  ES 集群多个节点，会自动选举一个节点为 master 节点，这个 master 节点其实就是干一些管理的工作的，比如维护索引元数据、负责切换 primary shard 和 replica shard 身份等。要是 master 节点宕机了，那么会重新选举一个节点为 master 节点。

  如果是非 master 节点宕机了，那么会由 master 节点，让那个宕机节点上的 primary shard 的身份转移到其他机器上的 replica shard。接着你要是修复了那个宕机机器，重启了之后，master 节点会控制将缺失的 replica shard 分配过去，同步后续修改的数据之类的，让集群恢复正常。

  就是说如果某个非 master 节点宕机了。那么此节点上的 primary shard 不就没了。那好，master 会让 primary shard 对应的 replica shard（在其他机器上）切换为 primary shard。如果宕机的机器修复了，修复后的节点也不再是 primary shard，而是 replica shard。

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

之前写蛋壳的房屋搜索一般先按照需求写出DSL，验证之后，再按照DSL实现java代码，实现后打印Java生成DSL与原DSL进行对比，对于复杂的查询这样编写可以减少出错；  



[Elasticsearch8.0版本中Elasticsearch Java API Client客户端的基本使用方法](https://blog.csdn.net/anjiongyi/article/details/123391835)  

[**Elasticsearch顶尖高手系列**笔记1-4篇](https://blog.vchar.top/dcs/1618405200.html#!)  

Search微服务源码：
[搜索引擎服务 search](https://github.com/zhangxx0/search)  

## Boss爬虫的简单尝试

- java的okhttp3+jsoup实现
- java爬虫webmagic（GitHub）
- java执行破解js获取cookie
- python的爬虫（代理ip）

爬取Boss数据主要作为ES的搜索练习使用，同时，近期也准备看机会，一举两得。  

Boss直聘的反爬手段还是很到位的，不登录情况下，它的js自动生成了一个`cookie`，这个cookie复杂冗长，在GitHub上搜索了一下爬虫方案，用java实现的也有几个，如上述使用webmagic的一个项目，但已经年久失修并不能正常工作；  

我自己则编写了第一种，使用jsoup解析了html，cookie则是直接从浏览器的headers中考的，其实网上绝大多数的实现都是这样，并没有几个真正破解了cookie，就算有人用python的selenium模拟，但基本上结果就是IP被封掉，我这个实现只能获取4页120条的数据，之后的就变成返回“请稍后”了，暂时用这些数据做测试；  

从GitHub中发现了JS破解和爬虫逆向的一个项目，其中就有Boss直聘的js破解，然后这个算法我又不能搬到java里来，就想在java中跑这段js获取cookie，于是找到了`Rhino`这个东西，然而一番努力也没解决运行js时浏览器函数找不到的问题，导致js无法运行，获取cookie也就无从谈起，于是暂时搁置。

还有一大批的python爬虫，不想做深入的研究了，感觉已Boss的反爬能力，再多努力估计也是竹篮打水，况且爬虫这个东西，，，还是少碰吧。  

![boss_job_index索引数据](http://zhangxx0.gitee.io/blog_image/elasticsearch/elasticsearch-boss-job-index-data.png)

[用 java 作为爬虫抓取表情包斗图](https://www.javanorth.cn/2021/12/26/java-doutu-2022-01-07/#)  

[Crack-JS-Spider](https://github.com/LoseNine/Crack-JS-Spider)  
[JS破解专题 | 某直聘__zp_stoken__算法](https://mp.weixin.qq.com/s/Nxh5uR7Dr0ajO_6CdFVBAQ)  
[Rhino: JavaScript in Java](https://github.com/mozilla/rhino)  