---
layout:     post
title:      "WOTPlus(4) - 开发_2"
subtitle:   "WOTPlus开发历程_2"
date:       2016-05-14 22:00:00
author:     "Zhangxx"
header-img: "img/post-wotplus-five.jpg"
catalog: true
tags:
    - WOTPlus
---

> 人在年轻时，最头疼的一件事就是决定自己这一生要做什么。

# 承上启下

昨天把首页的基本写完了，现在来继续阐述 **成就** **统计** **等级**页面的编写过程；

## 成就页面  

前期已经获取到了成就页所需要的数据 **Woter.getAchievements**，然后创建 `AchieveMentFragment` 来replace 主页的空白区域；  

![手绘成就页设计图](http://7xsvfv.com1.z0.glb.clouddn.com/AchievementsHandDesign.jpg)  

成就的页面采用的是一个嵌套的 **RecyclerView** ，外层使用 `LinearLayoutManager (VERTICAL)`，内层使用 `GridLayoutManager`，因此这里也使用了两个 **Adapter**：`AchieveMentAdapter` 和 `AchieveAdapter` ，前者是分类的成就信息块，后者是单个的成就信息；  
在 `AchieveAdapter` 中提供点击的回调：  

```
// 通过adapter中自己去提供回调
public interface OnItemClickLitener {
    void onItemClick(View view, int position, String description);

    void onItemLongClick(View view, int position);
}

private OnItemClickLitener mOnItemClickLitener;

public void setOnItemClickLitener(OnItemClickLitener mOnItemClickLitener) {
    this.mOnItemClickLitener = mOnItemClickLitener;
}
```

在 `onBindViewHolder` 方法中：  

```
// 如果设置了回调，则设置点击事件
if (mOnItemClickLitener != null)
{
    holder.itemView.setOnClickListener(new View.OnClickListener()
    {
        @Override
        public void onClick(View v)
        {
            int pos = holder.getLayoutPosition();
            // 成就的描述信息
            String description = achieve.getAchivementDes();
            mOnItemClickLitener.onItemClick(holder.itemView, pos, description);
        }
    });

    holder.itemView.setOnLongClickListener(new View.OnLongClickListener()
    {
        @Override
        public boolean onLongClick(View v)
        {
            int pos = holder.getLayoutPosition();
            mOnItemClickLitener.onItemLongClick(holder.itemView, pos);
            return false;
        }
    });
}
```

然后在 `AchieveMentAdapter` 中设置：`subadapter.setOnItemClickLitener` ，然后调用 `showDesDialog(description);` 方法来展示成就的描述，不再粘贴这个方法代码；

这个页面的编写比预计的要顺利，初步的编写完成样子：  

![成就页面原型](http://7xsvfv.com1.z0.glb.clouddn.com/achievementpage.jpg)

由于比较顺利，所以写完这个页面我就去玩坦克了，，，这毅力也是弱的一笔，然后当时还在看 **Game of Thrones** 前五季的第二遍，那天晚上看到了第三季，有一个镜头是Jon Snow和亿格瑞特的great wall的拥抱，感觉镜头好美，还截了图；  

至此，成就页面基本编写完毕；  
开始统计页面和等级页面；  

## 统计页面 

统计页面的数据获取也是在之前的工具方法中完成，不再赘述；

统计页面使用 `StatisticsFragment` 填充，在里面我使用了 `TabLayout` 和 `ViewPager`，并在代码中实现关联 `tabLayout.setupWithViewPager(viewPager);`，这种实现方式比较简便，在页面中实现了两个滑动或者点击切换的页面 **徽章与战绩** 和 **类型与国家**，同样使用两个Fragment来填充，分别为 `BadgeRecordFragment` 和 `TypeCountryFragment`；

下面是 `StatisticsFragment` 的布局

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <android.support.design.widget.TabLayout
        android:id="@+id/tabs"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:background="?attr/colorPrimary"
        android:theme="@style/ThemeOverlay.AppCompat.Dark.ActionBar"/>

    <android.support.v4.view.ViewPager
        android:id="@+id/viewpager"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        app:layout_behavior="@string/appbar_scrolling_view_behavior" />


</LinearLayout>
```

### 徽章与战绩页面

这个页面使用了三个 `CardView` 来分别展示：**通用徽章**、**全部战绩**、**战斗统计**，由于太长了，所以一屏是放不下的，一开始使用的是 `ScrollView`，但是不能实现上滑隐藏toolbar的效果，这里我有不想使用 RecyclerView，感觉有点重了，于是又开始了搜索，，，

这个地方也查找了挺长的时间，阅读了好多东西，终于在一篇文章中找到了 NestedScollView 才是与 CoordinatorLayout 配套使用的： 

[http://www.jianshu.com/p/1078568e859f](http://www.jianshu.com/p/1078568e859f) 

![ NestedScollView ](http://7xsvfv.com1.z0.glb.clouddn.com/nestedScollView.png)

关于ScrollView内部空间不能完全填充的问题：
[http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2014/0704/1629.html](http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2014/0704/1629.html)

添加属性： `android:fillViewport="true"`  

```
<android.support.v4.widget.NestedScrollView
        android:layout_width="fill_parent"
        android:layout_height="fill_parent"
        android:fillViewport="true">
        <!--内部控件match_parent需添加上面的fillViewport属性-->
```

初步效果图：  
![徽章与战绩](http://7xsvfv.com1.z0.glb.clouddn.com/badgeandrecordpage2.jpg)

### 类型与国家页面

同样使用 **MPAndroidChart**，不再赘述；

初步效果图：  
![类型与国家](http://7xsvfv.com1.z0.glb.clouddn.com/typeandcountrypage2.jpg)

## 等级页面

等级页面的数据获取遇到些问题，等级的数据在html中没有**直接显示**的数据，使用的是js的databind的方式，如图：  
![等级数据html源码图](http://7xsvfv.com1.z0.glb.clouddn.com/gradedata.png)

也就是说，等级数据是Ajax异步获取的；

于是开始查找js文件，忘了当时具体是怎么找到的了，反正最后找到了类似的Url：`url = window.ACCOUNT_RATING_URL;`，他们这个url是可以传递不同的参数的，使玩家可以查看**全部、4周、7天、1天** 的等级数据；这里我提取出一个4周的Url：[http://ncw.worldoftanks.cn/zh-cn/community/accounts/1509154099-/account_ratings/?timerange=28&group=all](http://ncw.worldoftanks.cn/zh-cn/community/accounts/1509154099-/account_ratings/?timerange=28&group=all)  

这时候就产生了一个状况：F12抓包（在战绩页面点击4周排名）返回json，而将这个url直接放入浏览器中显示404；

这也就是之前获取用户信息时所遇到的那个问题，正是在这个地方解决的，我的做法是：  
先比较两个请求的信息，如图，上面是页面点击，下面是放入浏览器中：  

![请求对比](http://7xsvfv.com1.z0.glb.clouddn.com/graderequest.jpg)

比较发现 **Request Header** 中的 Accept 不一样（当时还没意识到其实 `X-Requested-With` 是很重要的），下面的请求中是不包含 **json** 类型的；尝试用代码设置Accept；搜索：volley设置header
这里也耗费了好多时间，google了好久，才渐渐找到答案，参考链接：[Android Volley - BasicNetwork.performRequest: Unexpected response code 400](http://stackoverflow.com/questions/26796965/android-volley-basicnetwork-performrequest-unexpected-response-code-400)

于是重写 `getHeaders()` 方法：  

```
@Override
public Map<String, String> getHeaders() throws AuthFailureError {
    HashMap<String, String> headers = new HashMap<String, String>();

    headers.put("Accept", "application/json, text/javascript, */*; q=0.01");
//                headers.put("Accept", "application/json; charset=utf-8");
    headers.put("Accept-Encoding", "gzip, deflate, sdch");
    headers.put("Accept-Language", "zh-CN,zh;q=0.8,en;q=0.6");
    headers.put("Cache-Control", "max-age=0");
    headers.put("Connection", "keep-alive");
//                headers.put("Host", "ncw.worldoftanks.cn");
//                headers.put("User-Agent", "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/47.0.2526.111 Safari/537.36");
//                // 这个是会变化的；
//                // headers.put("X-CSRFToken", "LmLnQ3QEldNwXEWZhdg2nC0L8YkGtU7y");
    headers.put("X-Requested-With", "XMLHttpRequest");

    Log.d("统计", "headers=" + headers);
    return headers;
}
```

这里的代码之所以这么乱，是因为进行了好多次的测试造成的，各种组合都实验一下，忽然就能返回 json 了，有种豁然开朗的感觉；

这里我还想重写 `getParams()` 方法来传递参数，但是这个方法在执行的时候不被调用，google原因说是 JsonRequest  会忽略这个方法，具体见：[https://github.com/mcxiaoke/android-volley/issues/82](https://github.com/mcxiaoke/android-volley/issues/82)  
于是只能用拼接url的方式来传递参数了；

获取到 json 之后就好办了，使用 RecyclerView 展示，不再赘述；

初步效果图：  
![ 等级列表 ](http://7xsvfv.com1.z0.glb.clouddn.com/badgepage.png)

#### 一个`ViewPagerAdapter`问题

nav切换之后，再切换回统计页面时，两个viewpager都为空白；

第一次点击的时候回调用viewpager内的两个Fragment的createView方法；  
第二次点击的时候没有调用？？？

看文档：
     [http://stackoverflow.com/questions/25574234/viewpager-oncreateview-is-not-always-called](http://stackoverflow.com/questions/25574234/viewpager-oncreateview-is-not-always-called)  
     [http://developer.android.com/intl/zh-cn/reference/android/support/v4/app/FragmentPagerAdapter.html](http://developer.android.com/intl/zh-cn/reference/android/support/v4/app/FragmentPagerAdapter.html)  

![](http://7xsvfv.com1.z0.glb.clouddn.com/fragmentproblem.png)

这个问题的原因，应该就是出在这个Adapter上；
     更换为：`FragmentStatePagerAdapter`
这里 `FragmentPagerAdapter` ，  
> That is because a FragmentPagerAdapter keeps in memory every fragment. Hence, when you visit the first time the fragment, onCreate will be invoked but the second time Android will looking for in memory, so it not need invoke onCreate

也就是说，`FragmentPagerAdapter` 第二次加载Fragment的时候是不调用 `onCreate` 方法的，同样应该也是不调用 `onCreateView` 方法，但是我的逻辑是在 `onCreateView` 方法中加载数据的，不调用是不行的，于是换为`FragmentStatePagerAdapter`，使之每次都调用。

贴下 `ViewPagerAdapter` 代码：  

```
/**
 * Created by xinxin on 2016/4/4.
 * 统计页面ViewPager数据适配器
 *
 * FragmentStatePagerAdapter替换FragmentPagerAdapter
 */
public class ViewPagerAdapter extends FragmentStatePagerAdapter {

    private final List<Fragment> mFragments = new ArrayList<>();
    private final List<String> mFragmentTitles = new ArrayList<>();

    // 添加Fragment方法
    public void addFragment(Fragment fragment, String title) {
        mFragments.add(fragment);
        mFragmentTitles.add(title);
    }

    public ViewPagerAdapter(FragmentManager fm) {
        super(fm);
    }

    @Override
    public Fragment getItem(int position) {
        return mFragments.get(position);
    }

    @Override
    public int getCount() {
        return mFragments.size();
    }

    @Override
    public CharSequence getPageTitle(int position) {
        return mFragmentTitles.get(position);
    }
}
```

## 成就、统计、等级页面基本完成

2016年5月15日13:26:26 by zhang.xx







 

























