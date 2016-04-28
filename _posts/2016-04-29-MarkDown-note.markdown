---
layout:     post
title:      "Markdown学习笔记"
subtitle:   "工欲善其事，必先利其器"
date:       2016-04-28 00:00:00
author:     "Zhangxx"
header-img: "img/post-bg-markdown.jpg"
catalog: true
tags:
    - 技能
---

> “深思熟虑的结果往往就是说不清楚”


Markdown 是一种「电子邮件」风格的「标记语言」

参考文章：
[献给写作者的MarkDown新手指南](http://www.jianshu.com/p/q81RER)
[MarkDown语法说明](http://wowubuntu.com/markdown/)
[认识与入门 Markdown](http://sspai.com/25137)

编辑器
MarkPad 可以直接new jekyll page文档

# 标题

# 一级标题
## 二级标题
### 三级标题
#### 四级标题
##### 五级标题
###### 六级标题

*注：# 和「一级标题」之间建议保留一个字符的空格，这是最标准的 Markdown 写法。*

# 列表

## 无序
两种写法
- 文本1
- 文本2
- 文本3

* 111
* 222
* 333

## 有序

1. 文本1
2. 文本2
3. 文本3

# 快速链接
[跳转到引用 ](#build)
*这个在简书不好用啊？*

# 链接和图片

[简书](http://jianshu.io)
![](http://upload-images.jianshu.io/upload_images/734786-fbe354930623d311.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
*注：插入图片的语法和链接的语法很像，只是前面多了一个 ！。*

<p id = "build"></p>
---


#引用

> 一盏灯， 一片昏黄； 一简书， 一杯淡茶。 守着那一份淡定， 品读属于自己的寂寞。 保持淡定， 才能欣赏到最美丽的风景！ 保持淡定， 人生从此不再寂寞。

> 床前明月光
> 疑是地上霜
> 举头望明月
> 低头思故乡

*注：>和文本之间要保留一个字符的空格*



---

# 粗体和斜体

用两个 * 包含一段文本就是粗体的语法，用一个 * 包含一段文本就是斜体的语法

*一盏灯*， 一片昏黄；**一简书**， 一杯淡茶。 守着那一份淡定， 品读属于自己的寂寞。 保持淡定， 才能欣赏到最美丽的风景！ 保持淡定， 人生从此不再寂寞。

# 表格

| Tables | Are | Cool |
| ------------- |:-------------:| -----:|
| col 3 is | right-aligned | $1600 |
| col 2 is | centered | $12 |
| zebra stripes | are neat | $1 |

# 显示链接中带括号的图片

![][1]
[1]: http://latex.codecogs.com/gif.latex?\prod%20\(n_{i}\)+1

# 代码片段


这个```是那个按键？？？

`这个符号是在 Esc 键下面，切换到英文下即可。

只需要用两个 `  把中间的代码包裹起来，如 `code`

`code`

```
// action启动方法
public static void mainActionStart(Context c, String name, String region) {
  Intent intent = new Intent(c, MainActivity.class);
  intent.putExtra("name", name);
  intent.putExtra("region", region);
  c.startActivity(intent);
}
```
```javascript
// Component/index.js
require('./index.scss');
```


```后面的 javascript
 表示此段代码为javascript代码，Markdown会自行使用javascript代码颜色渲染。这里也可以不写。
PS：谁能够提供一个完整的Markdown可以渲染的语言列表啊，比如：linux命令这里写什么？
**应该是下面这个**
[rouge支持的语言列表 ](https://github.com/jneen/rouge/wiki/List-of-supported-languages-and-lexers)
[源头](http://jekyllcn.com/docs/templates/)

测试json：(加和不加效果一样，，，)
```json
{
    "people":[
        {"firstName":"Brett","lastName":"McLaughlin","email":"aaaa"},
        {"firstName":"Jason","lastName":"Hunter","email":"bbbb"},
        {"firstName":"Elliotte","lastName":"Harold","email":"cccc"}
    ]
}
```



使用代码高亮的例子如下:
{% highlight ruby %}
def foo
  puts 'foo'
end
{% endhighlight %}

#### 行内代码
这是`javascript`代码

```hbs
<ul>
  <li *ngFor="#hero of heroes">
    {{hero.name}}
  </li>
</ul>
```
```
Ember     : {{# each}}
Angular 1 : ng-repeat
Angular 2 : ngFor
Knockout  : data-bind="foreach"
React     : 直接用 JS 就好啦 :)
```


# 其他小用法

#### 分割线：
* * *
***
*****
- - ----------------------------------------

#### &符号：
&amp;

#### 自动链接
<377241804@qq.com>

#### 反斜杠
\*literal asterisks\*

Markdown 支持以下这些符号前面加上反斜杠来帮助插入普通的符号:
- \ 反斜线
- ` 反引号
- * 星号
- _ 底线
- {} 花括号
- [] 方括号
- () 括弧
- #井字号
- + 加号
- - 减号
- . 英文句点
- ! 惊叹号

#### 级联列表：
- Document Times
    - Frameworks
    - Style Guide
        - **OOCSS**
        - **SMACSS**
    - **Pre-processer**
    - **PostCSS**


<pre>
  照片Icon     One   One
主屏幕 ⇌ 相簿列表 ⇌ 相簿 ⇌ 单张照片
    Home      Back  Back
</pre>

<pre>
微信
Kant 给你发了一个红包
</pre>



—— Zhangxx 完成于 2016年4月29日02:40:40
查找rouge支持的语言花了一些时间；
