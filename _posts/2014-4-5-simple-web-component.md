---
layout: post
title: 超简易网页视图组件
date: 17:46 2014/4/5
tags:
- html
- javascript
---

最近一直在想一个web组件到底应该是啥样的

说到web组件, 大多数第一反应都是`<template>`, shadowDOM这样标准化的组件, 也会想到像ng这样的模块化组件.
但这些组件牢牢依靠polymer, angular这些库, 无法重用.
我一直想做个组件, 它啥都不依赖, 直接复制进html就可以跑, 但同时他又不会影响其他组件. 这组件的原理极度简单, 任何人一看就明白原理

有个ppt说到, shadowDOM和iframe最像, 我一想确实啊, iframe相当于是完美沙盒, shadowDOM同样非常独立, 但iframe无法使用宿主的css, 因此用iframe当组件略显丑陋, 而且性能有点蛋疼, 当然我不排斥iframe这个想法

要说组件, 必然还是有`css, js, html`三部分


### CSS

其中css是最简单的, 因为css自己本身就能重用, 同时我们使用less的话, 给用户编辑的css外面套一个`.mod-001 {}`就轻松做到了不污染他人, 这也有个问题, 就是用户无法自己使用.mod本身, 如果也要定义一个整体, 只能自己加一个box(比如由图片顶宽的组件)

```jade
h1 mod

// 自己套一个壳的组件

.box
  h1 mod
```

### HTML

html我依然选择了jade, 因为jade支持轻松编译pretty(带缩进)和不带缩进两种模式, 这也使我不需要再去自己压缩html了, 同时使用jade能保证代码的完美, 不会有直接复制过来的垃圾html混进来.
html如何组件化呢, 其中最最重要的一点就是组件是可以嵌套的, 就是说

```jade
ul.box
  each item in arr
    li.son
      +mixin(item)
```

没错, mixin就是组件, 这和less的mixin不同, css的mixin是常用样式, 不像jade的mixin是类似一个一段小html, 有了jade的mixin, 生活如此愉快, 前端可以很爽的预览模板输出了

### JS

最后是js, 我曾想过用amd那样

```js
define('mod-001', function() {
  ...
})
```

这样看着很美, 很简洁, 但其实js连这都不需要, 只需要这么写

```js
$(function() {
  alert('hi')
})
```

直接就按jQuery那么写是最好的, 没有任何门槛, 也可以直接沿用以前写好的代码.
唯一不同的就是这个`$`

```js
function _$(str) {
  // 如果是选择器, 就
  return $('mod-xxx').eq(n).find(str)
}

$.extend(_$, $.fn)
```

调用的时候也只需要

```js
for mods-in-same-type {
  var data = {
    $: _$(one-mod) // 对
  }
  with (data) {
    fn()
  }
}
```

这样每个mod都执行了一遍, 当然以后也可能需要区分全部mod一起执行, 和每个单个执行, 这个等实际写的时候再说

需要解决的问题

- 组件宽度, 第一反应是组件宽度由网格css的`col`来定啊, 但没这么简单, 因为有些单图组件, 图片的宽度是定死的
- js单个和全部运行
