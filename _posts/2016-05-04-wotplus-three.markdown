---
layout:     post
title:      "WOTPlus(2) - 开发_0"
subtitle:   "WOTPlus开发历程_0"
date:       2016-05-04 22:00:00
author:     "Zhangxx"
header-img: "img/post-wotplus-three.jpg"
catalog: true
tags:
    - WOTPlus
---
> 假如我要写什么，我根本就不管他格调不格调，正如谈恋爱时我绝不从爱祖国开始谈起。

# 写在前面

以下计划有几篇开发相关的文章，我并不是想做一个教程，而只是想记录下我开发 **WOTPlus** 时所一步步走过的路；  
文章的大体走向按照时间线，开发的顺序除数据获取外基本上是按照功能模块次第进行的；  
文章参照，我的印象笔记 - **坦克世界战绩查询WOTPlus开发日志**

---

# Let's Rock

首先还是 **数据** 的问题，毕竟坦克世界官网不是个开放平台，也没有给开发者使用的数据接口，所以只好自己来找了；  
抽象数据之前，先把工程建立出来，实现类似 **知乎日报** 的放大图片动画的启动页：

### 启动页

放大动画动画效果代码

```
iv_begin.setImageResource(R.mipmap.begin);
final ScaleAnimation scaleAnim = new ScaleAnimation(1.0f, 1.2f, 1.0f, 1.2f,
        Animation.RELATIVE_TO_SELF, 0.5f, Animation.RELATIVE_TO_SELF,
        0.5f);
scaleAnim.setFillAfter(true);
scaleAnim.setDuration(3000);
scaleAnim.setAnimationListener(new Animation.AnimationListener() {
    @Override
    public void onAnimationStart(Animation animation) {
    }

    @Override
    public void onAnimationEnd(Animation animation) {
        startActivity();
    }

    @Override
    public void onAnimationRepeat(Animation animation) {

    }
});
iv_begin.startAnimation(scaleAnim);

```

实现的效果图（*我先学习下怎么制作gif*）：

*弄了好一阵子，命令行、录屏软件之类的都试了，但是在我的MX2上就是不好使，统统闪退，所以先来个静态图先看着，等我有空了再弄动态演示图*

![启动图](http://7xsvfv.com2.z0.glb.clouddn.com/wotplus_001.jpg)

启动图片的尺寸：1080x1920

### 查询页与主页

这里的主页架构参照：Chris Banes 的轮子 [**cheesesquare**](https://github.com/chrisbanes/cheesesquare)  
使用侧滑菜单罗列功能模块；  
![ 侧滑菜单图 ](http://7xsvfv.com2.z0.glb.clouddn.com/wotplus_bolg_01_01.png)

**Navigation的图标问题**   
全部取自 Google 的 [**Material icans**](https://design.google.com/icons/)，免去了自己找或者自己画的痛苦，当然我也不会画，，，

然后开始写首页的UI，首页是一个Card的RecyclerView，所以先写了几个静态的CardView：  
![两个Card](http://7xsvfv.com2.z0.glb.clouddn.com/wotplus_blog_01_02.jpg)  

### 数据的获取

空中网并没有对外暴露玩家信息的json的接口，我用chrome抓了无数遍的包，分析了好久，始终没找到相应的接口， 所以只好从html中获取想要的数据了，这与json接口相比显然是慢了好多，但是这也是没办法的事情。  

大体的思路是：

* 获取玩家信息html字符串；
* 使用jsoup处理字符串并将相应信息对应到实体类 **Woter**
* 使用Woter来进行赋值操作；

首先获取html：  
一开始使用的是httpUrlconnection，但是返回的却是：**getErrorStream  ---错误 403: 权限不足**
应该是header之类的需要设置，但是我比较不习惯使用这个，于是我开始寻找网络框架来做请求；

试用了Volley、 [android-async-http](https://github.com/loopj/android-async-http)，最后选择了Volley，毕竟是Google的官方产品，莫名有种靠谱感；

Volley的使用教程，来自郭神：[Android Volley完全解析(一)，初识Volley的基本用法](http://blog.csdn.net/guolin_blog/article/details/17482095)  

获取到源码时候使用jsoup进行分析：
添加jsoup的依赖： `compile 'org.jsoup:jsoup:1.8.3'`  

jsoup的官方文档：[jsoup: Java HTML Parser](http://www.open-open.com/jsoup/parsing-a-document.htm)  
[使用jsoup解析HTML之获取html源码的几种方法](https://liuzhichao.com/p/1490.html)  

（*2016年5月5日22:47:20 今天先写到这里；*）

#### 概要信息抽取

2016年5月14日20:07:34 继续；  
创建了一个工具类 `JsoupHtmlUtil` 来解析html为 Woter ；  
其中 `handleWotPage` 方法：  

```
public static Woter handleWotPage(Document doc,String region) {
        Woter woter = new Woter();
        Elements elements;
        Element element;

        try {

            /**
             * (1)概要信息
             */
            element = doc.getElementById("js-profile-name");
            woter.setWoterName(element.text());

            // 创建时间戳
            element = doc.select(".js-date-format").first();
            woter.setTimeStamp(element.attr("data-timestamp"));

            // 主要信息栏
            elements = doc.select(".t-personal-data_value");
            woter.setPersonRanking(elements.get(0).text());
            woter.setPersonWin(elements.get(1).text());
            woter.setPersonFight(elements.get(2).text());
            woter.setPersonExp(elements.get(3).text());
            woter.setPersonDmg(elements.get(4).text());

            // 击杀／死亡率 伤害原因／收到
            elements = doc.select(".b-speedometer-weight");
            woter.setKillDeathRate(elements.get(0).text());
            woter.setDmgRecRate(elements.get(1).text());

            elements = doc.select(".b-speedometer-ratio");
            woter.setKillDeathNum(elements.get(0).text());
            woter.setDmgRecNum(elements.get(1).text());
      }
}
```  

其实就是在html中找到需要的数据，并根据ID、class等将数据提取出来放置到自己构建的Woter类中；  
麻烦就麻烦在数据要一个一个的找，并且要找出html的规律以便更好的提取数据；  

#### 军团信息获取  

军团信息是临时获取的，并没有固定的显示，因为有的人没有加军团，没法找到其url；  

因此，调用获取军团信息的网络请求之前需要进行一个判断：  

```
String clanUrl = userInfo.getResponse().get(0).getClan_url();

if (TextUtils.isEmpty(clanUrl)) {
    woter.setEnterClanFlag("0");
    // 没有clanUrl则跳出
    handler.sendEmptyMessage(1);
} else {
    woter.setEnterClanFlag("1");
    // 调用获取军团信息的请求
    mQueue.add(jsonObjectRequest);
}
```

从html中找到动态获取军团信息的js：  
![军团信息获取](http://7xsvfv.com1.z0.glb.clouddn.com/claninfo.jpg)  
这个 url 返回的是 **json** ，但是，这个json里的数据也是html代码，用于直接在html页面上显示军团块的，因此还需要用 jsoup 对里面的信息进行提取；  
实现方式：  
使用Gson将获取到的json数据转换成对应的类-ClanInfo；
然后使用 `jsoupClanInfo`方法来处理 `clanInfo.getData().getClan_block().toString()`  

```
/**
     * 使用jsoup处理军团信息
     * @param s
     */
    private void jsoupClanInfo(String s) {
        Document doc = Jsoup.parse(s);
        Element link = doc.select("img").first();

        Element clanPosition = doc.select(".number").first();
        Element clanDays = doc.select(".number").last();

        // 赋值给woter，但是这个地方要考虑赋值的顺序，应该是在获取解析的主页面信息之后，再设置这几个军团信息；
        woter.setClanDescription(link.attr("alt"));
        woter.setClanImgSrc(link.attr("src"));
        woter.setClanPosition(clanPosition.text());
        woter.setClanDays(clanDays.text());

    }
```  
至此，军团信息获取完毕；  


#### 抽取成就信息

成就信息比较复杂，有相当长的html代码块是用来展示成就信息的；  
首先，抽象成就信息为model - `Achievements`，作为Woter的一个属性；  
然后是 Achievements 将成就信息分为7个list及其相对应的数量，以及一个总数量：  

```
public class Achievements {

    // 总成就数量 获取/总数
    public String totalNum;

    /**
     * 战斗英雄
     */
    public List<Achieve> warheroList;
    /**
     * 荣誉排行
     */
    public List<Achieve> honorList;
    /**
     * 史诗成就
     */
    public List<Achieve> epicList;

    public List<Achieve> teamList;
    public List<Achieve> commemorateList;
    public List<Achieve> stageList;
    public List<Achieve> otherList;

    // 获取与总数数量  例如 9/14
    public String warHeroNum;
    public String honorNum;
    public String epicNum;
    public String teamNum;
    public String commemorateNum;
    public String stageNum;
    public String otherNum;
}
```

然后是 jsoup 提取成就信息的代码（*你可能不想看，因为太多了*）：  

```
/**
 * （2）成就信息
 */

// 提取成就息
// 总数
Element total = doc.select(".js-achievements-header").first();
// totalNum，在下面赋值
// System.out.println(total.text());

// 最外层div js-full-achievements
Element root = doc.select(".js-full-achievements").first();

// 成就分类数量
Elements subroots = root.getElementsByTag("h4");

// （1）七个分类的名称及数量，在下面赋值
//            String subNum1 = subroots.get(0).text();
// ...

// （2）七个分类的内容
Elements subrootContents = root.select(".b-achivements"); // 7个内容

Achievements achievements = new Achievements();
for (int i = 0; i < subrootContents.size(); i++) {

    List<Achieve> tempList = new ArrayList<Achieve>();

    Elements contentLis = subrootContents.get(i).select(".b-achivements_item");
    Elements contentDivs = subrootContents.get(i).select(".b-tooltip-main");

    for (int j = 0; j < contentLis.size(); j++) {

        Achieve achieve = new Achieve();

        Element contentLi = contentLis.get(j);
        Element contentDiv = contentDivs.get(j);

        // （1）from contentLi
        // ID和图片地址
        String img;
        Element imgContent = contentLi.getElementsByTag("img").first();
        if (QueryActivity.REGION_NORTH.equals(region)) {
            img = "http:" + imgContent.attr("src"); // //ncw.worldoftanks.cn/static/3.34.7/encyclopedia/tankopedia/achievement/geniusforwarmedal.png
        } else {
            img = "http://scw.worldoftanks.cn" + imgContent.attr("src"); // /static/3.35.7/encyclopedia/tankopedia/achievement/victorymarch.png
        }

        String id = imgContent.attr("alt");
        // 获得数量
        Element numContent = contentLi.select(".b-achivements_num").first();
        // 数量存在有或者没有两种情况，需要进行判断
        String num = "0";
        if (numContent != null) {
            num = numContent.text();
        }

        // （2）from contentDiv
        // 成就名称
        Element nameContent = contentDiv.select(".b-bold").first();
        String name = nameContent.text();
        // 成就描述
        String discription = "";
        Element contentDis = contentDiv.getElementsByTag("p").last();
        // 描述（1）
        discription = contentDis.text();
        // 附加描述 可能为空
        Element contentDisAdd = contentDiv.getElementsByTag("li").first();
        if (contentDisAdd != null) {
            discription += contentDisAdd.text();
        }
        // 另：几个特殊的情况
        // 巴顿猎手
        if ("pattonvalley".equals(id)) {
            Elements contentDiss = contentDiv.getElementsByTag("p");
            discription = "";
            for (Element p : contentDiss) {
                discription += p.text();
            }
        }
        // 猎人
        if ("beasthunter".equals(id)) {
            Elements contentDiss = contentDiv.getElementsByTag("p");
            discription = "";
            for (Element p : contentDiss) {
                discription += p.text();
            }
        }
        // 西奈雄狮
        if ("sinai".equals(id)) {
            Elements contentDiss = contentDiv.getElementsByTag("p");
            discription = "";
            for (Element p : contentDiss) {
                discription += p.text();
            }
        }

        achieve.setAchivementId(id);
        achieve.setAchivementImg(img);
        achieve.setAchivementNum(num);
        achieve.setAchivementName(name);
        achieve.setAchivementDes(discription);

        tempList.add(achieve);
    }

    achievements.setTotalNum(total.text());
    if (i == 0) {
        achievements.setWarheroList(tempList);
        achievements.setWarHeroNum(subroots.get(i).text());
    }
    if (i == 1) {
        achievements.setHonorList(tempList);
        achievements.setHonorNum(subroots.get(i).text());
    }
    if (i == 2) {
        achievements.setEpicList(tempList);
        achievements.setEpicNum(subroots.get(i).text());
    }
    if (i == 3) {
        achievements.setTeamList(tempList);
        achievements.setTeamNum(subroots.get(i).text());
    }
    if (i == 4) {
        achievements.setCommemorateList(tempList);
        achievements.setCommemorateNum(subroots.get(i).text());
    }
    if (i == 5) {
        achievements.setStageList(tempList);
        achievements.setStageNum(subroots.get(i).text());
    }
    if (i == 6) {
        achievements.setOtherList(tempList);
        achievements.setOtherNum(subroots.get(i).text());
    }

}

woter.setAchievements(achievements);
```

![ 解析完的成就类 ](http://7xsvfv.com1.z0.glb.clouddn.com/Achievements.png)

这个地方是比较复杂的，因为要根据html的结构一层层的抽丝剥茧，到最后才能将所有的数据一个个的抽离出来，这里使用了嵌套循环，使之最后直接构建出一个 **Achievements ** ，并set到Woter中去；

做完这一块之后，我的信息大增，毕竟页面要使用的数据源有了将近一半了！  
于是我就拿着这些能提取出来使用的数据去写页面了，，，

#### 延时加载  

首先阐明下输入玩家昵称和大区之后，点击查询之后：

1. 根据昵称和大区获取玩家的基本信息，其中最重要的就是：**account_id**;  
2. 获取到玩家ID后，用其ID和NAME做参数，获取玩家战绩的html源码String；
3. 将2中获取的String交给 `JsoupHtmlUtil.handleWotPage` 方法处理，返回缺少军团信息的Woter对象；  
4. 判断是否有军团信息，若有：获取军团信息，并处理返回的json，将军团信息set到3中的Woter，获取数据完毕，开始将Woter交给 `WoterAdapter` 绘制首页列表；若无：则获取数据完毕，开始将Woter交给 `WoterAdapter` 绘制首页列表；  

以上这个流程，刨去4先不说，123是肯定要执行的，123中有两个嵌套的网络请求，并且第二个网络请求返回的是500k左右的String，而且需要将这个String用jsoup进行解析为Woter，这是需要耗费相当长的时间的，当然这个东西不能在主线程中进行了，我这里使用一个 **Handler** 来进行延时加载的控制：  

```
private Handler handler = new Handler() {
        @Override
        public void handleMessage(Message msg) {
            super.handleMessage(msg);
            switch (msg.what) {

                case 1:
                    try {
                        woterAdapter = new WoterAdapter(getActivity(), woter);
                        // 添加载入动画
                        AlphaAnimatorAdapter animatorAdapter = new AlphaAnimatorAdapter(woterAdapter, mRecyclerView);

                        mRecyclerView.setAdapter(animatorAdapter);
                        deathWheelProgressDialog.dismiss();

                    } catch (Exception e) {
                        e.printStackTrace();
                    }

                    break;
            }
        }
    };
```

当执行完耗时操作后，这个操作可能是在第二个网络请求后（无军团信息的情况），也有可能是在第三个网络请求之后（有军团信息的情况），调用 `handler.sendEmptyMessage(1);` 方法；
一开始使用的时候我还犯了个错误，就是在网络请求之后直接调用了 sendEmptyMessage 方法，而解析html的耗时方法还没有执行完，这样就起不到控制异步任务的作用了，因此，应该在所有的耗时方法都执行完毕之后再调用 sendEmptyMessage；  

第一篇暂时写到这里，篇幅太长不宜阅读，下一篇继续；
2016年5月14日21:24:07 by zhang.xx
