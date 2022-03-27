---
layout:     post
title:      "WOTPlus(7) - 开发_5"
subtitle:   "WOTPlus开发历程_5"
date:       2016-05-15 03:00:00
author:     "Zhangxx"
header-img: "img/post-wotplus-eight.jpg"
catalog: true
tags:
    - WOTPlus
---


> 君不见黄河之水天上来，奔流到海不复回！　　君不见高堂明镜悲白发，朝如青丝暮成雪。

# 最后的修饰阶段

所有的功能基本上已经开发完毕，剩余的就是bug的修正与页面的美化以及优化之类的了；

## bug修改
这个bug我留的还真不少，第一次总结就写了满满的一页：  
![ 第一次bug整理 ](http://zhangxx0.gitee.io/blog_image/wotplus/wotplusbug.jpg)  

上面的这些bug我都是一个接着一个的改的，碰到难的就先放一放，

## 页面美化

页面美化的功臣是外服的两个 Assistant APP，我把它俩 **反编译** 后得到的无数图标，对于我这个不会p图的程序员来说，简直是福音啊，于是我就无耻地拿了一些来用，以至于美化的效果还可以；  

一些图标我使用的是固定宽高，换上之后一边齐，果然好看了许多；  
一些列表前面加上相应的图标也显得好看的多；  

## 一些琐碎的知识点

#### 1、FloatingActionButton 的隐藏

由于各个功能的页面都是以Fragment的形式展现的，但是：  
> You should move your FAB inside the `CoordinatorLayout`  

所以直接将放入Fragment的xml中fab是不会显示的，还得放到Activity的xml中；  

上滑消失参照：  
[FloatingActionButton滚动时的显示与隐藏小结](http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2016/0407/4126.html)

于是，在MainFragment中实现下列的代码，而在别的Fragment中添加 `fab.hide();` ，这样就控制只在 **首页** 显示这个fab，别的页面不显示；

```
// FloatingActionButton
final FloatingActionButton fab = (FloatingActionButton) getActivity().findViewById(R.id.fab);
fab.show();
fab.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View view) {
        // 弹出战绩查询对话框
        Intent intent = new Intent(getActivity(), AtyQueryDialog.class);
        startActivity(intent);

    }
});
```

RecyclerView 上滑隐藏fab：  

```
mRecyclerView.addOnScrollListener(new HidingScrollListener() {
    @Override
    public void onHide() {
        fab.animate()
                .translationY(fab.getHeight() + fabBottomMargin)
                .setInterpolator(new AccelerateInterpolator(2))
                .start();
    }


    @Override
    public void onShow() {
        fab.animate().translationY(0).setInterpolator(new DecelerateInterpolator(2)).start();
    }
});
```

#### 2、军团的 time_token 参数
分析军团数据的时候，由于军团信息的获取是ajax请求，其中有一个 time_token 参数需要传递，其实就是当前的时间，当时以为比较难就一直放着，结果，用一个 `new Date().getTime()` 与js中的参数一对比竟然一模一样，，，


#### 3、查询按钮的点击效果
底图是在 World of Tanks Blitz 上拿过来的，有两张，，，  
于是实现一个点击与松开时展现不同图片的效果：
参照：[改变按钮背景图片的效果](http://www.360doc.com/content/14/0313/20/12928831_360360114.shtml)

设置 `android:background="@drawable/query_button_press"`  
query_button_press 代码：  

```
<?xml version="1.0" encoding="utf-8"?>
<selector xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:state_pressed="false" android:drawable="@drawable/but_login_bg_guest"/>
    <item android:state_focused="true" android:drawable="@drawable/but_login_bg_guest"/>
    <item android:state_pressed="true" android:drawable="@drawable/but_login_bg_guest_tap"/>
</selector>
```

#### 4、MaterialEditText的使用

使用扔物线大神的控件：
[ MaterialEditText教程博客 ](http://www.rengwuxian.com/post/materialedittext)  

弄了个极简的样式：
![查询信息框](http://zhangxx0.gitee.io/blog_image/wotplus/query.png)

主要代码：  

```
<com.rengwuxian.materialedittext.MaterialEditText
                android:id="@+id/nametext"
                android:layout_height="wrap_content"
                android:layout_marginBottom="0dp"
                android:hint="玩家昵称"
                android:text=""
                app:layout_columnSpan="3"
                app:layout_gravity="fill"
                app:met_baseColor="@color/yellow"
                app:met_clearButton="true"
                app:met_floatingLabel="highlight"
                app:met_minCharacters="4"
                app:met_primaryColor="@color/gold" />
```

#### 5、点击fab弹出的自定义dialog

使用Activity创建对话框 `AtyQueryDialog`

![官方文档说明的自定义对话框](http://zhangxx0.gitee.io/blog_image/wotplus/zidingyidialog.png)  

Manifest文件中制定其主题：  

```
<activity    android:name=".activity.AtyQueryDialog"
                  android:theme="@style/DialogQueryTheme" />
```
```
<style name="DialogQueryTheme" parent="@style/Theme.AppCompat.Dialog">
    <item name="windowNoTitle">true</item>
</style>
```

fab点击事件直接写就可以了：  

```
fab.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View view) {
        // 弹出战绩查询对话框
        Intent intent = new Intent(getActivity(), AtyQueryDialog.class);
        startActivity(intent);

    }
});
```

效果图：
![fab query](http://zhangxx0.gitee.io/blog_image/wotplus/fabquery.png)

#### 6、南北区区分
南区账号查询时分析html出现空指针错误  
[北区：http://ncw.worldoftanks.cn/zh-cn](http://ncw.worldoftanks.cn/zh-cn)  
[南区：http://scw.worldoftanks.cn/zh-cn](http://scw.worldoftanks.cn/zh-cn)  

一个字母之差，，，

南区坦克单车成就图片错误：  
http:/static/3.35.7/encyclopedia/tankopedia/achievement/victorymarch.png  
南区的html中图片地址少了一段，因此若是南区则拼接缺失的一段：  
[http://scw.worldoftanks.cn/static/3.35.7/encyclopedia/tankopedia/achievement/medallafayettepool.png](http://scw.worldoftanks.cn/static/3.35.7/encyclopedia/tankopedia/achievement/medallafayettepool.png)

区分南北区示例代码：  

```
// 区分南北区
String region = PreferenceUtils.getCustomPrefString(getActivity(), "queryinfo", "region", "");
String gradeUrl;
if (QueryActivity.REGION_NORTH.equals(region)) {
    gradeUrl = Constant.GRADE_URL_BASE_NORTH + woterId + "-/account_ratings/";
} else {
    gradeUrl = Constant.GRADE_URL_BASE_SOUTH + woterId + "-/account_ratings/";
}
```

#### 7、右滑返回

首先我参照的是 **NBAPlus** 中的代码，但是弄过来之后各种错误；  
于是我就使用了 [SwipeBackLayout](https://github.com/ikew0ng/SwipeBackLayout) 开源库来实现；

引入的库是继承的FragmentActivity `public class SwipeBackActivity extends FragmentActivity`，扩展一个继承自AppCompatActivity 的 `SwipeBackBaseActivity`，以便实现与BaseActivity相同的效果  

想要右滑返回的只要继承 `SwipeBackBaseActivity` 就可以了：  

```
public class AtyAbout extends SwipeBackBaseActivity
```
同时，要指定：

```
<activity
        android:name=".activity.AtyAbout"
        android:theme="@style/AppTheme.TransparentActivity" />
```
```
<!--滑动返回主题-->
<style name="AppTheme.TransparentActivity" parent="AppTheme.NoActionBar">
    <!--<item name="android:windowBackground">@android:color/transparent</item>-->
    <item name="android:windowIsTranslucent">true</item>
</style>

```

#### 8、点击波纹扩散效果
这个特效出自：InstaMaterial  
[实现参考博文](http://android.jobbole.com/81113/)

这个的原理我也说不清楚，只知道怎么来做：  
两个页面：**发起页面--->目的页面  **

先自定义一个 `RevealBackgroundView` ，然后在 **目的页面** xml中添加：  

```
<com.xinxin.wotplus.widget.RevealBackgroundView
        android:id="@+id/revealBackgroundView"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />
```

**目的页面** 的`onCreate` 方法的最后添加：`setupRevealBackground(savedInstanceState);`

```
private void setupRevealBackground(Bundle savedInstanceState) {
    vRevealBackground.setOnStateChangeListener(this);
    if (savedInstanceState == null) {
        final int[] startingLocation = getIntent().getIntArrayExtra(Constant.START_LOCATION);
        vRevealBackground.getViewTreeObserver().addOnPreDrawListener(new ViewTreeObserver.OnPreDrawListener() {
            @Override
            public boolean onPreDraw() {
                vRevealBackground.getViewTreeObserver().removeOnPreDrawListener(this);
                Log.d("location",startingLocation.toString());
                vRevealBackground.startFromLocation(startingLocation);
                return true;
            }
        });
    } else {
        vRevealBackground.setToFinishedFrame();
    }
}
```

**发起页面** 的列表的点击事件中：  

```
adapter.setOnItemClickLitener(new TanksByTypeAdapter.OnItemClickLitener() {
    @Override
    public void onItemClick(View view, int position) {
        Tank tank = finalTanksByTypeList.get(position);

        // 全屏扩散
        // 开始位置
        int[] startingLocation = new int[2];
        view.getLocationOnScreen(startingLocation);
        startingLocation[0] += view.getWidth() / 2;

        Intent intent = new Intent(AtyTanks.this, AtyTank.class);
        intent.putExtra(Constant.START_LOCATION, startingLocation);
        if (tank != null) {
            intent.putExtra(AtyTank.TANK_TITLE, tank.getTankName());
            intent.putExtra(AtyTank.TANK_ID, tank.getTankId());
        }
        startActivity(intent);
        overridePendingTransition(0, 0);
    }

    @Override
    public void onItemLongClick(View view, int position) {

    }
});
```
在目的页面中首先要把相应的控件设置 `INVISIABLE`，类似：`tankScroll.setVisibility(View.INVISIBLE);`  

然后在状态变化时调用目的页面重写的 `onStateChange` 方法，从而使页面展示：  

```
@Override
public void onStateChange(int state) {
    if (RevealBackgroundView.STATE_FINISHED == state) {
//            tanksMainContent.setVisibility(View.VISIBLE);
        mAppBarLayout.setVisibility(View.VISIBLE);
        recyclerView.setVisibility(View.VISIBLE);
//            setStatusBarColor(Color.TRANSPARENT);
    }

}
```

大体上是这么个流程，，，我写的应该是对的，好久不看又生了；

#### 9、Glide 使用
常用：`Glide.with(context).load(tank.getTankBadge()).into(((TankShortInfo) holder).tankBadge);`  

#### 10、AlertDialog 的使用

```
AlertDialog.Builder builder = new AlertDialog.Builder(mContext);
builder.setTitle("成就描述")
        .setMessage(classAceDes)
        .setPositiveButton("了解", new DialogInterface.OnClickListener() {
            @Override
            public void onClick(DialogInterface dialog, int which) {
                dialog.dismiss();
            }
        });
AlertDialog dialog = builder.create();
dialog.show();
```

#### 11、RecyclerView 动画

[RecyclerViewItemAnimators Library](https://github.com/gabrielemariotti/RecyclerViewItemAnimators)  

构建时需要引入一个特定的工厂;

```
// RecyclerView 动画
SlideInRightAnimatorAdapter animatorAdapter = new SlideInRightAnimatorAdapter(adapter, recyclerView);
recyclerView.setAdapter(animatorAdapter);
```

#### 12、APK 瘦身

**Lint** 去除无用资源；  
[使用Android Studio的lint清除无用的资源文件](http://waychel.com/shi-yong-android-studiode-lintqing-chu-wu-yong-de-zi-yuan-wen-jian/)

[Android APK瘦身经验总结](http://www.jianshu.com/p/bfe44ef18aca)

图片压缩：  
[https://tinypng.com/](https://tinypng.com/)  
[https://tinyjpg.com/](https://tinyjpg.com/)

最终由6.5M压缩到4.3M；

## 末

开发的历程基本上结束了，剩余的就是打包上线的事情了，对了还有推广的事情，，，  
这个也早就写了，写的也不是很全，见 [上架记录](http://amx1390.com/2016/05/08/wotplus-n/)

对于第一个正式版本0.6的技术开发历程，如有以后再补充吧，接下来使用新技术、增肌新功能当然会另有博客来阐述，所以，第一个版本，到这里吧！ 哎呀，我写了一天的博客了，，，

2016年5月15日23:11:39 by zhang.xx
