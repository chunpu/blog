---
date: 1:01 13/11/10
title: 我真是弱爆了
layout: post
tags:
  - life
  - Javascript
---

最近不写博客了，原因不过是因为我又开始工作了，一下班就找同学玩，吃饭，自以为还挺充实，现在看来真是弱爆了

在起初的时候，我还觉得自己挺厉害，产生这种感觉不过是因为我可以分分钟写一个微博，写个博客，写个小网站。。

我甚至觉得自己的前端水平是不是再过个一两年就可以碰天花板了。。实在愚蠢之极

今天打完dota，看了下关于京js的一些分享，其中一个[@belleveinvis](http://weibo.com/belleveinvis)的[ppt](http://vdisk.weibo.com/s/544GFEuUPkI)，我看了直接想找个缝钻进去，ppt做的漂亮极了，讲的是ecma6的东西，我除了最简单的let之外什么都没听过，点开微博一看还是我科ustc校友，目前大四. 回想我大四的时候还在为编译原理而苦恼，但人家已经做出一个自己的语言。不禁内牛满面，这是他的[博客](http://typeof.net/index.html)

最近有收到猎头电话，以及两个大公司的面试通知，一开始我妈直夸我厉害，现在我只觉得弱爆了

不得不提一下n久前一道小米的面试题

简化以下大概是这样的

```
完成以下需求
function repeat (callback, interval) {
  ...
}
var repeatedFunc = repeat(console.log, 5000);
repeatedFun("helloworld")
```

初看此题感觉很简单，不过是写一个返回函数的函数(抢票用的？抢票是6秒)

```javascript
var repeatFunc = repeat(console.log, 5000)

repeatFunc('hi')

function repeat(fn, interval) {
  return function (str) {
    setInterval(function() {
      fn(str)
    }, interval)
  }
}
```

非常不妙的是chrome直接报typeError了，我当时真的急哭，又套了几层function，感觉自己是不是哪里弄错了。大约耗了几十分钟，基本都想放弃了，后来试了下加了一句

```javascript
function print(obj) {
  console.log(obj)
}
var repeatFunc = repeat(console.log, 5000)

repeatFunc('hi')

function repeat(fn, interval) {
  return function (str) {
    setInterval(function() {
      fn(str)
    }, interval)
  }
}
```

就可以了，当然这显然无法让面试官满意，面试官还问我为什么，我说可能是因为console是native函数吧(我当然自己也不知道我说的是啥),请原谅我的愚昧

今天手贱在node中试了一下,node出乎意料乖巧的一个一个把hi输出来了，我懵了，尼玛nodejs和chrome难道不都是v8?

这里面包含了不少问题：

- 为什么nodejs可以，chrome不可以

- 为什么把`console.log`放入`print`就可以了

当然这其中的原因还是要看上面提到那个大神的博客<http://typeof.net/s/jsmech/01.html>

首先说我对scope理解不对的地方

前面错误的代码在chrome中报的错是`Uncaught TypeError: Illegal invocation`

也就是说类型不对，错误的调用，其原因就是console.log中使用了this，虽然fn确实是console.log的引用，但this已经不是console了，解决方法是这样写repeat

```javascript
var repeatFunc = repeat(console.log, 5000)

repeatFunc('hi')

function repeat(fn, interval) {
  return function (str) {
    setInterval(function() {
      fn.call(console, str) // 加上console
    }, interval)
  }
}
```

当然这也是权宜之计，我们是知道函数叫console.log因此写上console的

用一个最简单的例子说明这种现象

```javascript
var obj = {
  func: function() {
    console.log(this)
  }
}

var fn = obj.func
obj.func() // obj
fn() // window
```

现在我们至少能猜测第一个问题

> 为什么nodejs可以，chrome不可以

因为宿主环境不同，nodejs输出并不需要用到this,而chrome用到了

改成print为啥可以呢？因为print中是直接调用console.log，它的this就是console

可以推断出如果是`var print = console.log`,肯定也会报相同的错误

至于为什么浏览器中的console.log会用到this,我还是多看看源码和标准再来说吧，一多说就暴露学历
