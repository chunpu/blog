---
layout: post
title: 深入理解jQuery中的data(data_user, data_priv)
date: 17:39 2014/2/22
tags:
- javascript
---

data函数在jQuery中看起来很不起眼, 就像沙滩上一颗平凡的沙子, 但仔细一瞅, 却惊讶的发现data是jQuery中无比重要的一环, 甚至jQuery中各种事件都基于此

---

data在jQuery中有两种

一个是用来存数据的, 对应的对象分别是

- 存储对象: `data_user`

- 获取与设置方法: `$.data(el, key, value)`

另一个是用来存事件的, 如click事件那种, 对应的对象分别是

- 存储对象: `data_priv`

- 获取与设置方法: `$._data(el, key, value)`


`data_user`和`data_priv`, 就如其名, 一个是用户用的, 一个是jQuery私有的, 他们都是一个叫Data的实例对象

---


### 存数据的 *data_user*

data的使用方法简单无比

```javascript
$('body').data('key', 'value')
// 或者
$.data(document.body, 'key', 'value')
```

你可能会说, 这函数也太弱了吧, 自己分分钟就可以写一个

```javascript
$.fn.data = function(k, v) {
  if (arguments.length === 2) {
    return this.each(function() {
      this[key] = v
    })
  }
  return this[0][k]
}
```

这样写看上去没啥问题, 其实却有很多隐患, 比如这样

```javascript
// case1
$('body').data('tagName', 'somevalue')
// case2
$('body').data('key', $('body'))
```

直接套在node对象上碰到原生的属性只会失效. 又如果值存为自己, 谁保不准会引用来引用去出问题呢, 而且直接放在节点对象上, 容易暴露逻辑, 被轻易修改

我们来看看jQuery怎么做的

```javascript
$(document.body).data('aaa', 'value-aaa')
console.dir(document.body)
```
![document.body](http://fimg.qiniudn.com/jquery-data-0.png)

我们看到多出来暗色的属性-->jQuery加一串随机数

首先是暗色, 我们就知道此属性不能被枚举(enumerable为false), 是用definePropert*定义的属性

其次可以看出这个属性和jQuery.expando很像

expando是一个jQuery的唯一标示, 这个多出来的属性就是jQuery.expando再加上一串随机数产生的

> 为啥费这么大劲, 直接叫`jquery-data`啥的不好嘛? 这是因为一个网页能共存多个jQuery, 用[$.noConflict](https://api.jquery.com/jQuery.noConflict/)来实现不冲突共存

同时我们能看到这个属性的值为`3`

我们再试试修改body上的data值

```javascript
$(document.body).data('aaa', 'value-bbb')
$(document.body).add(document).data('bbb', 'value-xxx')
```

再看看这个属性的值, 还是3, 根本没有变

聪明的同学一定知道了, jQuery存data是用一个数组一样的东西存的, dom上只存这个数据的位置

而这个数组(array-like), 就是本文一开始提到的data_user的一个属性

先打印一下data_user存的东西

![data_user](http://fimg.qiniudn.com/data_user.png)

可以看到这里面存了2个属性, 一个就是expando, 也就是节点元素上多出来的那个奇怪属性, 另一个是cache, 不像数组, 但存着刚才我们看到的`3`, 3属性的值就是之前存入的data

知道这些我们就可以写出一个最简单的类似jQuery的data函数

```javascript
;(function() {
  $.fn.data = dataFn
  var data = {
    expando: jQuery.expando + Math.random(),
    cache: []
  }
  var eo = data.expando
  function dataFn(k, v) {
    if (arguments.length === 2) {
      return this.each(function() {
        if (this[eo] >= 0) {
          data.cache[this[eo]][k] = v
        } else {
          var len = data.cache.length
          this[eo] = len
          data.cache[len] = {}
          data.cache[len][k] = v
        }
      })
    }
    var o = data.cache[this[0][eo]] || {}
    if (arguments.length === 0) {
      return o
    }
    return o[k]
  }
})()
```

---

### 存事件的 *data_priv*

那为啥jQuery的事件也和它有关呢?

了解jQuery事件的同学一定知道, 不管怎么给dom加事件, 在chrome的调试板上只能看到dom被绑了一个很奇怪的匿名函数

因为所有事件都被jQuery托管了, 但是都绑同一个函数, jQuery是怎么知道哪个handler对应哪个dom呢?

我们不妨再打印一下dom元素

```javascript
$('a').click(alert)
console.dir($('a')[0])
```

![](http://fimg.qiniudn.com/data_priv.jpg)

我去, 怎么看上去和存了data一模一样啊, 也多了一个暗色奇怪字符串, 值也是3

我们也给a存下data看下

`$('a').data('aaa', 222)`

![](http://fimg.qiniudn.com/data-user-priv.png)

这下有俩奇怪字符串了!

其实它们就是`data_user`和`data_priv`这俩货各自的expando

事实上data和events本来就是完全一样的东西, 只是一个存在`data_user`中, 一个存在`data_priv`中

打印一下`data_priv`

![data_priv](http://fimg.qiniudn.com/data-priv-obj.jpg)

可以看到`data_priv`确实和`data_user`一样, 只是存放的数据比较复杂, 是一些jQuery的events标示

jQuery提供了`$._data`这个方法(以后会被删掉), 让我们可以像`$.data`那样轻松的获取绑定在某节点上的handlers

比如我们看一下github在document上绑定的事件

![github-data-priv](http://fimg.qiniudn.com/github-data-priv.png)

知道jQuery的event handlers存放和data一样, 借助data, 那我们自己模仿jQuery实现一个最简单最简陋的`$.fn.on`函数就像玩一样

```javascript
;(function() {
  $.fn.on = onFn
  function onFn(ev, sel, handler) {
    return this.each(function() {
      this.addEventListener(ev, _handler)
      var o = {}
      o[ev] = {
        sel: sel,
        handler: handler
      }
      $.data(this, o)
    })
  }
  function _handler(e) {
    var _event = $.data(this)[e.type]
    var sel = _event.sel
    var elements = $(this).find(sel)
    var handlerQueue = []
    elements.each(function() {
      var cur = e.target
      if (cur === this || $(this).has(cur).length) {
        _event.handler.call(this, e)
      }
    })
  }
})()
```
