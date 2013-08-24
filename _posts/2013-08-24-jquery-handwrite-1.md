---
layout: post
title: 手写jQeury-1
tags:
- jQuery
- javacsript
---

jQuery是一个伟大的库,但我一直没有去专门学习它.不过jQuery非常的人性化,导致于我们不需要像backbone那样去看说明书.

不过不看说明书的后果就是我对jQuery产生很多误区.

这些误区直接暴露在我几个月前的面试中,比如

> 我会原生Javascript,同时也会jQuery

[@朴灵](http://weibo.com/shyvo)大大听后立马纠正我,jQuery也是原生Javascript.

这点我在此次学习中算是了解了,jQuery确实是完全的Javascript,除了两个全局变量`$`和`jQuery`外,做到了完全没有污染其他空间.

也就是说如果引进了jQuery,照样可以写原生的js,完全不妨碍.

> jQuery自已定义了dom,用起来不舒服

我现在想起自己的回答都想找个缝钻进去,当时的我根本没意识到所有的jQuery DOM对象都是一个类数组(Arraylike)

选择器
-------

jQuery从名字上讲,是一个js的dom选择器.我们熟知的dom选择器是`document.getElementById`和`document.getElementsByTagName`

但我们知道CSS的选择器非常实用,比如`div h1.title`,他一下子表达了很多意思,想一下子选出这些dom在css中很快,但是在JS中却非常难办.

jQuery本身主要是为了解决这个问题.要注意的是现代浏览器都提供了`document.querySelectorAll`让我们开发者也能用css选择器的接口

让人大跌眼镜的是很多同学,甚至是一些jQuery讲师,居然不知道这个接口.jQuery和css选择器接口的返回值都是一个NodeList,他们都是类数组

> 类数组是指键是从0,1,2,...这样像数组那样排列,可能有length属性,但没有数组中prototype的比如push,pop等方法的对象

jQuery和css选择器返回的NodeList都不是动态的(getElementsByTagName是动态的),jQuery返回的NodeList有大量的proto方法,比如bind,click等等,正是这些proto方法让jQuery变得如此简单易用.


jQuery的nodeList
----

正是这个nodeList让我刚学js的时候困惑不解.我天真的以为在选id的时候就是调用`getElementById`而其他的时候是调用其他高级选择器.

就是说我根本无法理解我明明是选定一个id,它怎么可能返回一个类数组.

因此我才会在面试的时候说jquery返回的不是原生的dom对象,比如不能用`myDiv.innerHTML`来获取innerHTML.

我们知道jQuery获取innerHTML的方法是`$('xxx').html()`.

看到上面这个api,我当然是以为`$('xxx')`是返回一个自己封装的dom对象,我当时抱怨为何把innerHTML的属性给删了.真是怎么也想不到我们这样是对一个列表调用`.html()`

`$('xxx').html()`返回的当然是字符串,不是数组,因此实际上`$('xxx').html()`的返回值就是`$('xxx')[0].innerHTML`

但我敢肯定,新人在碰到这个api的时候都会想一下:"为什么jQuery不直接提供`$('xxx').html`"的值啊,非要做成一个函数.jQuery只能哭着说"臣妾做不到啊."

而相反的,`$('for').html('bar')`又是对整个nodeList进行innerHTML赋值


jQuery的链式编程
---

jQuery的链式(chainable)编程一直是被人称道的,他可以这么写

```javascript
$('div').find('h3').eq(2).html('Hello')
          .end().eq(4).css(xxx)
```

但原理说穿了不值一提,就是每个proto方法返回`return this`罢了.

```javascript
$('div1').insertAfter($('div2')) // return div1
$('div2').after($('div1')) // return div2
```

虽然很简单,但我们依然要注意每条链主语,上面两句作用一样,但主语不同,同理还有`appendTo`和`append`

上面还有一个`.end()`这个函数可以返回上一个主语.

jQuery事件触发
-----

jQuery事件触发也和我们常用的不同,平常的我在注册click事件的时候都是

```javascript
myBtn.onclick = function() {
  doSomething()
}
```

这样的问题是如果有别人也想用到这个onclick事件,就会把之前注册的函数覆盖了.

jQuery的做法是

```javascript
$('myDiv').click(function() {
  doSomething()
})
```

这样的好处是不会覆盖之前的click事件函数,我们可以猜测它内部应该是使用了`addEventListener`

这样只是说对了一半,如果单单使用`addEventListener`,那我们在webkit inspector的event listener中应该能看到绑定的所有函数.

但事实上我们用jQuery绑定事件永远只看到一个奇怪的函数.

据说这就是jQuery处理多事件高效的原因..(一个函数托管所有事件)

那他是怎么实现的呢,我看了一下源码,发现根本看不懂..一个大大说过,源码看不懂,就自己实现,在对比源码,如果不一样,那不是你傻逼了,就是坐着傻逼了.

```javascript
var fn = {}
$.fn = fn
var eventList = []
function handler(e) {
  for (var i = 0; ev = eventList[i]; i++) {
    if (ev.target === e.target && ev.type === e.type) {
      // event match
      if (ev.handler(e) === false) {
        e.preventDefault()
        e.stopPropagation()
      }
    }
  }
  return this
}

fn.__proto__.bind = function(evType, fn) {
  for (var i = 0; i < this.length; i++) {
    this[i].addEventListener(evType, handler)
    eventList.push({
      handler: fn,
      type: evType,
      target: this[i]
    })
  }
  return this
}

;['click', 'dbclick', 'mousedown', 'mousemove', 'mouseover', 'mouseout', 'mouseup'].forEach(function(evType) {
  fn.__proto__[evType] = function(fn) {
    return this.bind(evType, fn)
  }
})
```

随手写了一下,简单的说就是注册一个事件的时候,不管是什么事件,直接dom绑定handler函数,然后把事件的类型和dom对象以及其函数加到一个内部的数组eventList中. 这样所有的事件函数都在这个数组中了.

当事件触发的时候,从头遍历eventList,如果`e.type`和`e.target`都相同了,那就调用此函数.

提一下jQuery中如果事件函数返回值是`return false`就是取消冒泡事件和默认事件,也就是上面代码中这一段

```javascript
if (ev.handler(e) === false) {
  e.preventDefault()
  e.stopPropagation()
}
```

实现的干净利索,方便实用,居家旅行必备

