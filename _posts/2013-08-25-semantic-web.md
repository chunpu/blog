---
layout: post
title: Web语义化
date: 10:24 2013/8/25
tags:
- html
- web
---

Web语义化主要分两个

1. `HTTP`语义化

2. `HTML`语义化

HTTP语义化
-----

HTTP语义化是针对HTTP协议来说的,最有代表性的是`Rest API`

### path

首先是path,路径,对于有大量接口的服务器,我们需要仔细的设计自己的API

比如看新浪微博的API<http://open.weibo.com/wiki/%E5%BE%AE%E5%8D%9AAPI>

一般的话path用`/`来表示嵌套关系, 用querystring来表示filter,和属性值

### HTTP method

HTTP method也具有语义化

一般来说普通服务器只用`GET`和`POST`,但还有`PUT`和`DELETE`来对应数据库操作.不过说实话在实践的过程中,我还是觉得只用`GET`和`POST`好

### 无状态HTTP

HTTP必须要是无状态的,所谓无状态,说细点就是用户带着你给用户配的`token`,可以访问任意的带有自己`私钥`的服务器.

HTTP无状态,说白了就是为了负载均衡.

如果HTTP是有状态的,你在后台保存着用户的session,那用户就只能访问你这台服务器,除非你还做了session同步

另一种方法是在负载均衡中配置会话保持,一般的方法是使用cookie保存server的ID,也可以是保存一张IP HASH表,但很显然这会产生其他问题

可以看出牵连会话的HTTP对于性能会有巨大的损失,而且如果一个服务器坏了,那上面的用户都没法继续会话了

HTML语义化
-------

这才是重头戏,上文说了,Web语义化是为了让机器读懂.我觉得这个说的太玄乎.

系分到HTML语义化,一句话概括就是

> `块元素不出现<DIV>, 行内元素不出现<SPAN>`

有朋友要问了,"现在主流不都是div+css么,你看看xx网站, 各种div,全部div啊"

div确实很全能,但这都是css的功劳,这使得我们连表格也都可以用div来完成

现在谁再用`<table>`标签,可能会被同行认为过时吧

可是HTML语义化在显示二维数据的时候就推荐用table.因为table就是用来显示表格的啊!不然W3C设计这个标签干嘛的.因此我们所说的不用table,完全是指你不该在整个网站上套table.

理解HTML语义化,就是理解HTML标签.

[kejunz](http://weibo.com/kejunz)大大曾说过,最好的html代码是这个

```html
<div class="mod">
  <div class="hd"></div>
  <div class="bd"></div>
</div>
```

[玉伯](http://weibo.com/lifesinger)大大发[ISSUE](https://github.com/lifesinger/lifesinger.github.com/issues/105)表示赞成

其中也说道为何不用`<header>/<section>/<footer>`,却没有明确说明

在我看来,仍然觉得`<header><footer>`这样的好,为什么呢?

因为越是外层的块元素,选择器权重应该越低,我们知道id,class,标签对应的选择器权重分别是`00100`, `00010`, `00001`

如果我们用`id="hd"`来写的话,显然会导致选择器灾难,因为ID应该用在嵌套最深处的标识元素

但使用`class="hd"`就会好很多

### Section Content

Section标签有4个

- article

- aside

- nav

- section

section content是用来表示从头到尾的部分,也就是这样用

```html
<header>
</header>
<section>
</section>
<footer>
</footer>
```

在section内部我们一般是放头信息和概要,比如

```html
<section>
  <h1>title</h1>
  <p>outline</p>
</section>
```

### Heading Content

就是我们常用的`h1,h2,h3,h4,h5,h6,hgroup`

### Phrasing Content

其他大部分标签都是此类(语法标签?),如`a, b, canvas, cite, code, time, textarea`

### Embedded content

嵌入标签,有`audio,canvas,embed,iframe,img,math,object,svg,video`

### Interactive content

有`a, audio, button, input, select, video`等

一些语义化标签
------

- `<nav>` 导航标签,用来放入超链接列表

- `<article>` 最常用就是里面包含一个h1和p

- `<aside>` 用来放置于正文无关的内容,比如广告,侧边栏

- `<header> | <footer>` 只需要知道他们并不是section content

- `<address>` 用来放置联系信息,如作者名字,电话等.

试想如果有了这些标签,那爬虫就能干的事就多了,比如可以轻易通过`<article>`找出正文,自动过滤`<aside>`广告,根据`<nav>`来继续爬,根据`<address>`来显示作者和联系方式.

因此与其说Web语义化是为了让机器明白,其实就是让爬虫更加轻松.

另一个明显的语义化是`rel`属性

`rel`用在链接标签中(`<a>`, `<area>`, `<link>``)

我们知道链接标签最常用的属性是`href`

而`rel`属性的值很多,比如常见的是css的`stylesheet`,还有RSS的`alternate`

哦,对了,rel的全称是什么?其实就是`relationship`啦

因此,rel是用来表示href所指向位置和当前位置的关系的

rel的还有license,help,author,tag

如果链接是作者个人页面,那我们就应该写上`rel="author"`

还有的属性有`next`, `prev`, `nofollow`

这个也非常实用,比如对于很多有阅读模式的浏览器,他们会自动找next和prev的链接,帮用户设置好上一页,下一页

浏览器甚至可以智能的帮用户预加载next页面.同样nofollow可以告诉爬虫不要接着爬了

可以这么说,只有`<link>`的`stylesheet`rel值是对普通浏览器有用的(它会使浏览器自动下载href指向的css文件),其他rel值都是无用的

没用我们还要写,完全是为了让爬虫,让阅读器,特殊浏览器,浏览器插件更加方便的读取我们的信息

我想这大概就是语义化的一部分了

### 块标签

- `dl` 全称是description list

- `dt` 全称是description term

- `dd` 全称是description definition

- `figcaption` 用于代码,图片,表格的标题,一般

- `figure` 用于包含带有标题的代码,图片

> 
```html
<figure>
 <img src="bubbles-work.jpeg"
      alt="Bubbles, sitting in his office chair, works on his
           latest project intently.">
 <figcaption>Bubbles at work</figcaption>
</figure>
```

- `div` 万恶的无语义元素,W3C明确指出,只有你想不出其他标签的时候才应该用div

### 文本级标签

- `em` emphasis强调,会被搞成斜体

- `strong` 也是强调,比em还强,一般是粗体(话说我今天才知道粗体会使元素变长)

- `i` 全称是italicized,

- `small` 常用来免责声明

- `abbr` abbreviation

>
```html
<abbr title="Web Hypertext Application Technology Working Group">WHATWG</abbr>
```
> title为text的全称

- `dfn` defining instance of a term,表示一个术语

- `time` 属性datetime放置时间格式

> 
```html
<span class="summary">Web 2.0 Conference</span>:
<time class="dtstart" datetime="2005-10-05">October 5</time>
```

- `span` 行内元素的div,本身毫无语义,也就是实在没合适的行内元素才会选这个,一般放在`<code>`中显示代码高亮



### 如何让IE6-8支持语义新标签?

首先是JS

>
```javascript
(function(){
    if(!/*@cc_on!@*/0) return
    var html5 = "abbr,article,aside,audio,bb,canvas,datagrid,datalist,details,dialog,eventsource,figure,footer,hgroup,header,mark,menu,meter,nav,output,progress,section,time,video".split(',');
    for(var i = 0, len = html5.length; i < len; i++ ) {
        document.createElement(html5[i])
    }
})();
```

代码很容易,唯一蛋疼的是`/*@cc_on!@*/`是啥玩意儿

这个还得看MSDN的手册<http://msdn.microsoft.com/zh-cn/library/vstudio/eb0w91wa(v=vs.100).aspx>

然后是CSS

```css
article,footer,header... {
  display: block;
}
```

总结
----

Web语义化除了HTTP的语义化外,主要就是HTML语义化

HTML语义化只要是反对使用无语义化的`<div>`和`<span>`而使用HTML定义好的语义化标签

语义化的好处是方便爬虫,浏览器,插件,特殊浏览器这些软件可以方便的把HTML中的信息归类
