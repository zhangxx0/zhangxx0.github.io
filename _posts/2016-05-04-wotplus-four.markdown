---
layout:     post
title:      "WOTPlus(3) - 开发_1"
subtitle:   "WOTPlus开发历程_1"
date:       2016-05-14 02:00:00
author:     "Zhangxx"
header-img: "img/post-wotplus-four.png"
catalog: true
tags:
    - WOTPlus
---

> 做一件事，无论大小，倘无恒心，是很不好的。

# 承上启下

上篇博文记录了 **WOTPlus** 的启动页、查询页、侧滑菜单与主页、数据的获取以及异步的控制，但到现在为止，其实主页中的第一个页面 **首页** 都还没有完成，只有两个静态的头部Card，还缺少两个比例图以及军团的信息Card，于是就从这两个开始吧。  

## 比率图与军团信息

官网上是有两个比率图的：击杀/死亡率、伤害原因／收到；这俩用的是类似**仪表盘**一样的图表，于是我就开始google Android的图表库，实在是没找到相似的图表，于是我就偷了个懒，使用了 **Pie** 图来展示这个，，，  
之后要研究下那个仪表盘的实现，看了外服的AssistantAPP中确实也是使用的仪表图，这一点是我技术不够，，，抽个时间将这个研究下怎么实现；  

这里的 PieChart 我使用的是：**MPAndroidChart**；需要添加依赖：`compile 'com.github.PhilJay:MPAndroidChart:v2.2.3'`   
看了一个demo之后就拿来用了，之后在  **统计** 菜单中的 **类型与国家** 中也是使用的相同的技术；  

军团的Card中多了一个军团图标的展示，需要动态从网络中下载；
一开始使用的是一个搜的一个方法（继承AsyncTask的一个内部类），性能不是很好，于是后期直接换成：**Glide** 了，简便又好用；  
`Glide.with(mContext).load(mWoter.getClanImgSrc()).into(((ClanViewHolder) holder).clanFlag);`  

![静态首页完成图](https://gitee.com/zhangxx0/blog_image/raw/master/wotplus/mainpageStatic.png)

## 首页Adapter赋值

首页是用 MainFragment 来replace `fl_content`， MainFragment 的布局文件：`fragment_main`代码如下：  

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <android.support.v7.widget.RecyclerView
        android:id="@+id/recyclerview"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />

</LinearLayout>
```

然后在 MainFragment 中设置 `recyclerview` ：  

```
mRecyclerView = (RecyclerView) view.findViewById(R.id.recyclerview);
// 设置RecyclerView的布局管理；
LinearLayoutManager lm = new LinearLayoutManager(mActivity, LinearLayoutManager.VERTICAL, false);
mRecyclerView.setLayoutManager(lm);
mRecyclerView.setItemAnimator(new DefaultItemAnimator());
mRecyclerView.setHasFixedSize(true);
```

然后在 MainFragment 中调用 `getData();` 函数，从 **Preference** （非第一次查询）中或者 **网络**（第一次查询、或者在首页的查询）中获取数据并赋值给Woter：  

```
// 尝试获取页面信息
private void getData() {

    // 添加限制，如果SharedPreference中存在了woter信息，则不再从网络中获取
    // 由查找页面进入需要一个flag，查找页面进入时不管存不存在SharedPreference都需要从网络获取；
    if ("".equals(queryFlag)) {
        // 从CharedPreference中获取woter
        String woterString = PreferenceUtils.getCustomPrefString(getActivity(), "woter", "woterString", "");
        Gson gson = new Gson();
        if (!TextUtils.isEmpty(woterString)) {
            woter = gson.fromJson(woterString, Woter.class);
        }
        handler.sendEmptyMessage(1);

    } else {
        if (!HttpUtil.isNetworkAvailable()) {
            Snackbar.make(getView(), "网络不可用！", Snackbar.LENGTH_LONG).show();
        } else {
            getDataFromWeb();
        }
    }

}
```

取得Woter对象之后，使用 `WoterAdapter` 来给RecyclerView赋值：  

```
woterAdapter = new WoterAdapter(getActivity(), woter);
---
// 添加载入动画
AlphaAnimatorAdapter animatorAdapter = new AlphaAnimatorAdapter(woterAdapter, mRecyclerView);
mRecyclerView.setAdapter(animatorAdapter);
```

关于 **WoterAdapter**，继承自`RecyclerView.Adapter<RecyclerView.ViewHolder>`，这也是RecyclerView的Adapter的标准的写法，由于首页的RecyclerView中包含的是四个 **CardView** ，这里重写了 `getItemViewType`，根据itemType展示不同的布局；

```
@Override
public RecyclerView.ViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
    if (viewType == TYPE_ONE) {
        View view = layoutInflater.inflate(R.layout.item_main_top_0, parent, false);
        return new AccountViewHolder(view);
    }
    if (viewType == TYPE_TWO) {
        View view = layoutInflater.inflate(R.layout.item_main_top, parent, false);
        return new MainInfoViewHolder(view);
    }
    if (viewType == TYPE_THR) {
        View view = layoutInflater.inflate(R.layout.item_main_charts, parent, false);
        return new ChartsViewHolder(view);
    }
    if (viewType == TYPE_FOR) {
        View view = layoutInflater.inflate(R.layout.item_main_clan, parent, false);
        return new ClanViewHolder(view);
    }
    return null;
}
```

在 `getItemCount` 方法中控制军团信息的展示与否：  

```
@Override
public int getItemCount() {
    // 可以在这个地方判断军团信息是否存在，控制返回的视图数量；
    if (mWoter.getEnterClanFlag().equals("0")) {
        return 3;
    } else {
        return 4;
    }
}
```
上面这两段就是 **WoterAdapter** 中的核心代码了，赋值的代码比较简单，对应好相应的数据与控件就可以了；

对了，WoterAdapter 还有个重要的方法（*使用Gson把实体类转换为字符串，然后放到CharedPreference中；别的Fragment加载时从CharedPreference中读取并再次转换为实体类；*）：  

```
// 将Woter放入Preference
public void putWoterToPreference(Woter woter) {
    Gson gson = new Gson();
    String woterString = gson.toJson(woter);
    // Log.d("woter", woterString);

    PreferenceUtils.putCustomPrefString(mContext, "woter", "woterString", woterString);
}
```

这个方法是在 WoterAdapter 的构造方法中调用的，用作数据的暂存；

### 首页信息的网络请求结构

这个其实在上一篇中已经说过了：

1. 根据昵称和大区获取玩家的基本信息，其中最重要的就是：**account_id**;
2. 获取到玩家ID后，用其ID和NAME做参数，获取玩家战绩的html源码String；
3. 将2中获取的String交给 JsoupHtmlUtil.handleWotPage方法处理，返回缺少军团信息的Woter对象；
4. 判断是否有军团信息，若有：获取军团信息，并处理返回的json，将军团信息set到3中的Woter，获取数据完毕，开始将Woter交给 WoterAdapter绘制首页列表；若无：则获取数据完毕，开始将Woter交给 WoterAdapter绘制首页列表；

由于我使用的是Volley，所以产生了以下 **迷之嵌套** ，（*容易使人头晕，慎看*）：  
`getDataFromWeb()`方法：

```
/**
 * 从网络中获取数据
 */
private void getDataFromWeb() {

    // （1）使用HttpURLConnection 报403错误，应该是一些请求头的设置错误；

    // (2)使用Volley
    //  使用mActivity做参数时报错：修改为getActivity
    // java.lang.NullPointerException: Attempt to invoke virtual method 'java.io.File android.content.Context.getCacheDir()' on a null object reference
    final RequestQueue mQueue = Volley.newRequestQueue(getActivity());

    // 第一个请求 请求官网获取用户信息
    // 拼接请求串
    String name = PreferenceUtils.getCustomPrefString(getActivity(), "queryinfo", "name", "");
    final String region = PreferenceUtils.getCustomPrefString(getActivity(), "queryinfo", "region", "");

    if (QueryActivity.REGION_NORTH.equals(region)) {
        Constant.USER_JSON_URL = Constant.USER_JSON_BASE_URL_NORTH + CommonUtil.urlEncodeUTF8(name) +"&name_gt=";
    } else {
        Constant.USER_JSON_URL = Constant.USER_JSON_BASE_URL_SOUTH + CommonUtil.urlEncodeUTF8(name) +"&name_gt=";
    }

    JsonObjectRequest jsonObjRequest = new JsonObjectRequest(Request.Method.GET, Constant.USER_JSON_URL, null,
            new Response.Listener<JSONObject>() {
                @Override
                public void onResponse(JSONObject response) {

                    Gson gson = new Gson();
                    //final XvmUserInfo xvmUserInfo = gson.fromJson(response.toString(), XvmUserInfo.class);
                    final KongzhongUserInfo userInfo = gson.fromJson(response.toString(), KongzhongUserInfo.class);

                    if (userInfo.getResponse().size() == 0) {
                        Snackbar.make(getView(), "玩家信息不存在！", Snackbar.LENGTH_LONG).show();
                        backToQuery();
                    } else {
                        // 保存woterId
                        PreferenceUtils.putCustomPrefString(getActivity(), "woterId", "woterId", userInfo.getResponse().get(0).getAccount_id());

                        if (QueryActivity.REGION_NORTH.equals(region)) {
                            Constant.WOTER_URL = Constant.WOTER_BASE_URL_NORTH + userInfo.getResponse().get(0).getAccount_id() + "-" +
                                    CommonUtil.urlEncodeUTF8(userInfo.getResponse().get(0).getAccount_name()) + "/";
                        } else if (QueryActivity.REGION_SOUTH.equals(region)) {
                            Constant.WOTER_URL = Constant.WOTER_BASE_URL_SOUTH + userInfo.getResponse().get(0).getAccount_id() + "-" +
                                    CommonUtil.urlEncodeUTF8(userInfo.getResponse().get(0).getAccount_name()) + "/";
                        }

                        // 第二个请求
                        // 这里的URL需要在第一次请求之后获得，因此初始加载的是空的，会报Bad url错误；
                        // 还是得级联，但是级联实在是太不美观了；2016年3月31日23:17:15
                        final StringRequest stringRequest = new StringRequest(Constant.WOTER_URL,
                                new Response.Listener<String>() {
                                    @Override
                                    public void onResponse(final String response) {

                                        new Thread(new Runnable() {
                                            @Override
                                            public void run() {

                                                jsoupHtml(response, region);

                                                // 第三个请求
                                                // 获取军团信息的json
                                                String clan_url;
                                                if (QueryActivity.REGION_NORTH.equals(region)) {
                                                    clan_url = Constant.CLAN_URL_BASE_NORTH + userInfo.getResponse().get(0).getAccount_id() + "&time_token=" + new Date().getTime();
                                                } else {
                                                    clan_url = Constant.CLAN_URL_BASE_SOUTH + userInfo.getResponse().get(0).getAccount_id() + "&time_token=" + new Date().getTime();
                                                }
                                                Log.d("clan_url", clan_url);
                                                final JsonObjectRequest jsonObjectRequest = new JsonObjectRequest(clan_url, null,
                                                        new Response.Listener<JSONObject>() {
                                                            @Override
                                                            public void onResponse(JSONObject response) {
                                                                Gson gson = new Gson();
                                                                ClanInfo clanInfo = gson.fromJson(response.toString(), ClanInfo.class);

                                                                handleClaninfo(clanInfo);

                                                                // 执行耗时的方法之后发送消给handler
                                                                handler.sendEmptyMessage(1);

                                                            }
                                                        },
                                                        new Response.ErrorListener() {
                                                            @Override
                                                            public void onErrorResponse(VolleyError error) {
                                                                Log.e("TAG3", error.getMessage(), error);
                                                                Snackbar.make(getView(), "获取军团信息出错！", Snackbar.LENGTH_LONG).show();
                                                                backToQuery();
                                                            }
                                                        });

                                                // 此处判断是否要请求军团信息，设置EnterClanFlag
                                                // 从官网获取的userInfo中没有ClanId
                                                // 从http://north.warsaga.cn/clans/2083/?utm_campaign=wot-portal&utm_medium=link中截取
                                                String clanUrl = userInfo.getResponse().get(0).getClan_url();

                                                if (TextUtils.isEmpty(clanUrl)) {
                                                    woter.setEnterClanFlag("0");
                                                    handler.sendEmptyMessage(1);
                                                } else {
                                                    woter.setEnterClanFlag("1");
                                                    mQueue.add(jsonObjectRequest);
                                                }
                                            }
                                        }).start();

                                    }
                                },
                                new Response.ErrorListener() {
                                    @Override
                                    public void onErrorResponse(VolleyError error) {
                                        Log.e("TAG2", error.getMessage(), error);
                                        Snackbar.make(getView(), "获取战绩页面出错！", Snackbar.LENGTH_LONG).show();
                                        backToQuery();
                                    }
                                });
                        mQueue.add(stringRequest);
                    }

                }
            }, new Response.ErrorListener() {
        @Override
        public void onErrorResponse(VolleyError error) {
            Log.e("TAG1", error.getMessage(), error);
            Snackbar.make(getView(), "获取userinfo出错！", Snackbar.LENGTH_LONG).show();
            backToQuery();
        }
    }) {
        @Override
        public Map<String, String> getHeaders() throws AuthFailureError {
            HashMap<String, String> headers = new HashMap<String, String>();

            headers.put("Accept", "application/json, text/javascript, */*; q=0.01");
            headers.put("X-Requested-With", "XMLHttpRequest");

            return headers;
        }
    };

    mQueue.add(jsonObjRequest);
}
```

这个代码确实看的人头晕眼花的，所以，赶快的学习**RxJava和Retrofit**来重写它吧！  
其中使用的jsoup解析html的工具方法就不再赘述了，已经在第一篇中阐述；  

### 拼接动态的URL遇到的错误

之前测试用的都是静态的URL，所以要把真正的参数（昵称大区）进行URL的拼接；  
当时很困惑的一个问题（曾在qq群发问）：  
*请教个问题：我用chrome抓包，一个请求的返回是json，requestcode是200  
但是用volley获取json请求的时候，却返回个404  
把链接放到浏览器也是404页面  
可是抓包的时候是好好的返回的json啊  *

这个问题是在进行第一个请求的时候碰到的，即根据昵称和大区获取 **acount_id** 等信息的请求：  
于是一开始我选择了 **曲线救国**，那就是使用 **XVM** 的一个获取用户信息的请求链接，能正确获取到json数据，而且 **acount_id** 都是通用的，于是前期是这样不伦不类的实现的，，，  

到了后期，**XVM**的这个接口直接挂掉了，因为他们在改版，这个接口直接废弃掉了，于是，我又回来啃官网的接口，也就是解决上面的这个问题，这个问题在之后的 **获取等级信息** 和 **获取战车战绩信息** 的时候也遇到了，这两个地方是解决了的，解决点就在 **header** 的设置上！  
但是呢，这个 **header** 的设置也不是完全相同，这点空中网就比较坑爹了，当时我是一个一个的实验的，花了不少的时间；  

上面的 `getDataFromWeb` 方法使用的是后期的方法，两者返回的对象是不同的，前者是 `XvmUserInfo `，后者是 `KongzhongUserInfo` ;  

解决办法：  
Volley中设置 header :  

```
@Override
public Map<String, String> getHeaders() throws AuthFailureError {
    HashMap<String, String> headers = new HashMap<String, String>();

    headers.put("Accept", "application/json, text/javascript, */*; q=0.01");
    headers.put("X-Requested-With", "XMLHttpRequest");

    return headers;
}
```

*最近在实验Retrofit对这个请求的获取，发现只设置一个 `"X-Requested-With: XMLHttpRequest"`注解就可以了，很让人费解，，，*

### 自定义加载Dialog-死亡之轮

这其实是后期实现的一个小的自定义控件，但是写在这个地方貌似比较合适：  
这个控件写的糊里糊涂的，内部的原理我现在并不是很明晰，先写下来：  

`DeathWheelProgressDialog` 的使用：  

```
deathWheelProgressDialog = DeathWheelProgressDialog.createDialog(getActivity());
deathWheelProgressDialog.show();

deathWheelProgressDialog.dismiss();

```

`DeathWheelProgressDialog` 类中的 `createDialog` 方法：

```
public static DeathWheelProgressDialog createDialog(Context context) {
    deathWheelProgressDialog = new DeathWheelProgressDialog(context, R.style.CustomProgressDialog);
    deathWheelProgressDialog.setContentView(R.layout.death_wheel_progress_dialog);
    deathWheelProgressDialog.getWindow().getAttributes().gravity = Gravity.CENTER;
    deathWheelProgressDialog.setCancelable(false);

    return deathWheelProgressDialog;
}
```

这个自定义Dialog我感觉是最初级的了，只是修改了Dialog的布局文件，而我的实现方式是在布局文件中展示一个gif图片（一个不停旋转的死亡之轮）；  这个我隐隐中感觉在性能上可能赶不上那种好多图实现一个动画效果的实现方式，但是也说不好差在哪儿，还是需要进一步学习下自定义控件；

我的参照自定义ProgressDialog:  
[http://blog.csdn.net/rohsuton/article/details/7518031](http://blog.csdn.net/rohsuton/article/details/7518031)

![死亡之轮](https://gitee.com/zhangxx0/blog_image/raw/master/wotplus/death-wheel.gif)

布局文件（*使用的在github上找的一个gif开源库*）：  
*`compile 'pl.droidsonroids.gif:android-gif-drawable:1.1.+'` ，好像还要添加repositories，具体见github:[android-gif-drawable](https://github.com/koral--/android-gif-drawable)*

```
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <pl.droidsonroids.gif.GifImageView
        android:layout_width="51dp"
        android:layout_height="54dp"
        android:src="@drawable/death_wheel"
        android:layout_centerInParent="true"
        />

</RelativeLayout>
```

## 首页基本完成

开始编写 **成就** 页面吧；  
2016年5月15日00:00:50 by zhang.xx
