---
layout: post
title: 简易MVVM
date: 17:29 2014/3/1
tags:
- javascript
---

`MVVM`虽然用到的地方很少, 但对于特定网页应用, 如复杂表单, todo等尤其实用. 
`MVVM`的概念早就吵的很火了, 根本不需要多解释, 今天我们来试着实现一下一个最基本的mvvm小框架

MVVM最大的功劳在于可以减少大量的dom操作, 因为mvvm框架早就帮你处理好了, 因此, mvvm的速度就在于dom操作的快不快

dom操作快不快, 取决于mvvm操作的dom大小, 粒度越细, 操作越快, 也就是说mvvm应该操作的是不可再分的attribute(nodeType = 2)和text(nodeType = 3)节点

先看一下mvvm一般怎么用的, [我的例子](http://codepen.io/ftft1885/pen/slfhF)

```html
<div id='demo'>
  Hello <span class='{{css}}'>{{name}}</span>!
</div>
```

这是一个最简单的html, 直接写在dom中, 不是写在script中的那种

其中css和name就是两个变量, 对应model中的css和name

mvvm的js一般这么写

```javascript
var vm = mvvm('#demo', {
  model: {
    css: 'green',
    name: 'Monkey'
  }
})
```

这样, model中的css和name就自动跑到html中了

我们甚至可以用`vm.model.name = 'Tony'`, 自动的将html中的那部分替换掉, 而且依然保持html的简洁 (没有任何用于选择的标示)

那这个是怎么实现的呢

我们先尝试实现双向绑定中的`model -> view`

### model -> view

```javascript
function renderDOM(dom) {
  each(dom.attributes, function() {
    render(this)
  })
  each(dom.childNodes, function() {
    if (this.nodeType === 1) {
      return renderDOM(this)
    }
    render(this)
  })
}
```

这个函数很清楚, 先render属性节点, 再render子节点, 如果子节点有dom节点, 则递归执行

render函数是什么呢?

```javascript
function render(node) {
  var arr = node.textContent.split(start)
  if (!arr.length) return
  var ret = ''
  for (var i = 0; i < arr.length; i++) {
    var two = arr[i].split(end)
    if (two.length === 1) ret += arr[i]
    else {
      ret += model[two[0]] + two[1]
    }
  }
  node.textContent = ret
}
```

render函数的目标就是找到最小节点的`textContent`中含有{&#123;key&#125;}这样的变量, 并替换它

当然我们可以用正则来直接匹配, 但可能会影响性能, 而且不方便我们以后扩展

通过renderDOM函数, 我们大概完成了model -> view的最基本功能

可是节点中的{&#123; &#125;}被替换了, 之后model变了我们怎么再同步model到view呢?

这也是mvvm中一个小难点

我的方法是:

> 如果节点中用到了某个model值, 我们就把这个节点存起来, 他的原始textContent也存起来, 这样以后就能根据变化的model找到这个节点并按照原始textContent更新了

```javascript
var model2sync = {} // obj to save nodes
for (var k in model) {
  model2sync[k] = []
}
function render(node) {
  var arr = node.textContent.split(start)
  if (!arr.length) return
  var ret = ''
  for (var i = 0; i < arr.length; i++) {
    var two = arr[i].split(end)
    if (two.length === 1) ret += arr[i]
    else {
      ret += model[two[0]] + two[1]
      // two[0] 正是用到的model值, 把他存起来
      model2sync[two[0]].push({
        node: node,
        raw: node.textContent
      })
    }
  }
  node.textContent = ret
}
```

好了, 存好了node和原始值, 我们怎么修改`model.name = xxx`就更新视图(html)呢?

这里要用到es5中的`dsfineProperty`, 当然, 这也是不手动update model的唯一方法

我们还要定义一个代理的model, 他的key和model一模一样

```javascript
var proxyModel = getProxyModel()
function getProxyModel() {
  var obj = {}
  each(Object.keys(model), function(i, k) {
    Object.defineProperty(obj, k, {
      set: function(v) {
        model[k] = v
        // 获得和这个key绑定的节点
        var arr = model2sync[k]
        each(arr, function() {
          // 更新节点的text
          this.node.textContent = renderStr(this.raw)
        })
      },
      get: function() {
        return model[k]
      }
    })
  })
  return obj
}
```

关于model到view的基本实现就到这儿, 具体例子可以看刚才[demo页面](http://codepen.io/ftft1885/pen/slfhF)的demo1

---

### view -> model

前面提到, mvvm还有一个重要使用场景是表单, 我们希望在用户在修改表单的时候, 表单所对应的数据也跟着变

这也就是双向绑定的view到model功能, 我们知道, 修改表单, 无非是通过鼠标和键盘,
所以我们绑定一个元素的keyup和click事件即可, 如果您已经高端到用语音了, 那还是得老老实实绑change事件

不过这个实现起来太过简单

```javascript
// 先写一个事件的on函数, 可以无视
function on(el, events, handler) {
  if (Array.isArray(events)) {
    each(events, function() {
      on(el, this, handler)
    })
  }
  else el.addEventListener(events, handler, true)
}
// 对root元素绑定keyup, click事件
on(this.root, ['keyup', 'click'], function(e) {
  var name = e.target.name
  // 如果目标元素有name
  if (name) {
    // 如果值变了
    if (e.target.value != model[name]) {
      // 更新代理model
      me.model[name] = e.target.value
    }
  }
})
```

简直一气呵成, 简单无比, 具体可见刚才[demo页](http://codepen.io/ftft1885/pen/slfhF)的demo2

---

当然MVVM要考虑的太多了, 完整的MVVM能轻易的实现列表的增删改查 (专业做todo), 但这显然不在我们100行MVVM的探讨范围之内

上述代码完整版在[这里](https://github.com/chunpu/mvvm2/blob/master/mvvm.js)
