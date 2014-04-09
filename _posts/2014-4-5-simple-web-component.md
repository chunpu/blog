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
      +mod-001(item)
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
function _$(el) {
  return function(str) {
    // 如果是选择器, 就
    return $(el).find(str)
    // 否则还是
    return $(str)
  }
}

$.extend(_$, $.fn)
```

这样这个$只在自己的mod内作用, 不会污染别人, 调用的时候也只需要

```js
$('.mod-001').each(function() {
  var data = {
    $: _$(this)
  }
  with (data) {
    fn()
    ...
  }
})
```

这样每个mod都执行了一遍, 当然以后也可能需要区分全部mod一起执行, 和每个单个执行, 这个等实际写的时候再说

### 共用变量

共用变量是极其重要的一点, 因为不管在less, 还是jade, 还是js中都需要变量, 也许你不理解为何less需要变量, 可以举个例子

一个轮播, 既然作为一个通用组件, 他就可以自由的选择是放多少张图, 我们知道轮播一般都是放在一个很长的`<ul>`中, 然后修改其left值来进行运动的

```css
ul {
    width: @w * (@n+1);
    position: absolute;
    left: 0;
    li {
        float: left;
        height: @h;
        width: @w;
    }
}
```

ul的宽度必然由轮播的数量`n`来决定(+1是因为还有个克隆的元素)

因此这里非常简单的处理一下

```js
var data = {
  n: 7,
  w: '600px'
}
// 如果data中某值为str或number, 自动添加
@n: 7;
@w: 600px;
```

> 这里不得不提到这个css和js不统一的地方, less常常是带着单位比如`px`一起计算的, 但js却是手动拼接`el.css({left: w + 'px'})`.
这里推荐使用600px, 首先是因为less智商太低, 连`@size: unit(@size, px)`都不支持, 而js处理宽度要简单的多, 基本`parseInt`一下就行了, 以后说不定还能提供js版的unit

```js
unit('600px') = [600, 'px']
```

data远不止这些作用, data本身是用来mod数据渲染的

```js
$.extend(data, getDataById(id)) // id为组件id, 唯一的, 比如由img, title等值
```

### 数据结构

```js
grid: {
  width: 980px,
  margin: 20px,
  rows: [row1, row2, ...]
}

row: {
  cols: [col1, col2, ...]
}

col: {
  mods: [mod1, mod2, ...]
}

mod: {
  id: 343534346, // uuid
  data: {}, // 如果没data, 就用id取
  modid: 002, // mod的id, 用mod的id可以取出对应的html, js, css,
  mods: [mod1, mod2, mod3] // 用到的mod
}
```

因此渲染需要三个数据

一个是全部的用到的全部mod(jade, less, js)

```jade
+mod-001()
+mod-002()
+mod-003()
```

我们必须知道其用到的mod才能进行渲染

另一个是全部的data, 和id有关

最后就是grid的结构了, 还是那句话mod只关心它下面的那个mod, 不会管孙子mod

#### 动态mixin

首先我们要知道, 大多数模板, 都不支持`include(#{var})`这种写法的, 一般我们需要预先知道子模板叫啥.
jade的mixin也是如此

```jade
each item in arr
  +mod-001(item)

// 但不能这么写
each item in arr
  +mod-#{mod-id}(item)
```

这儿的处理办法是在jade上再套一个模板

```jade
each item in arr
  +mod-{%raw%}{{= mod-id}}{%endraw%}(item)
```

> 这里犯了一个严重的错误, jade早早支持了[dynamicMixins](https://github.com/visionmedia/jade/pull/1308/commits), 他确实不能这么写`+mod-#{var}`, 但可以这么写`+#{'mod-' + var}`.
要避免这种错误的发生, 就是要紧跟自己常用repo的commit, 版本更新, bug issue等, 不然容易闹笑话:sob:

不过这又有问题了, 谁说每个tab或者每个div都是用一样的mixin/mod啊, 我想第一个用mod-001, 第二个用mod-002, ..不行吗

我们的目标jade是长这样的, 拿tab来举例

```jade
ul.nav
  each item in tabs
    li= item.title
ul.bd
  li
    +mod-#{tabs[0].mod}(tabs[0])
  li
    +mod-#{tabs[1].mod}(tabs[1])
  ...
  li
    +mod-#{tabs[n].mod}(tabs[n])
```

于是我们得这么写

```js
var data = {
  tabs: [{
    title: 1,
    mod: '001'
  }, {
    title: 2,
    mod: '002'
  }]
}
```

```jade
{% raw %}
ul.nav
  each item in tabs
    li= item.title
ul.bd
  {{ for (var i = 0; i < tabs.length; i++) { }}
    li
      +mod-{{= tabs[i].mod}}(tabs[i])
  {{ } }}
{% endraw %}

// 好吧, 上面是我刚开始不知道有dynamicMixin的方案, 现在简洁多了
ul.nav
  each item in tabs
    li= item.title
      ul.bd
        each tab in tabs
          li
            +#{'mod-' + tab.mod}(tabs[i])
```

这种模板套模板的方式实在不优雅, 但是为了mixin动态化, 不得不这么做, 这个时候, 又引出一个问题

### Grid是不是一个组件

这是个很复杂的问题, 如果grid是组件, 那我们真正做到了递归, 看着非常完美.
但问题就在与, grid是可以拖动, 添加行, 如果其他组件也能做到这样, 操作起来会不会很乱?
不过这也很好解决, 每次只能操作主mod和他的一级son mod, 要操作son mod, 必须把主mod切换成son mod, 大小的话可以用transiform的scale变大嘛

那grid写成组件是啥样呢?

```jade
.grid
  each row in rows
    .row
      each col in row
        .col(class="col-#{col.width}")
          each mod in col.mods
            +#{'mod-' + mod.id}
```

grid的数据结构就是上文的结构, 可以看出非常的简洁, 完全可以组件化

因此可以看出整个页面可以彻底数据化了, 注意, 仅仅是数据化, 而不是mvvm中model的意思, 此工具想做的是通过数据快速渲染整个页面, 和mvvm要做的完全无关, mvvm将会在组件中用

### 渲染一个页面需要的数据

- locals: 也就是第一个model的data, 一般根model是grid model
- 全部用到的mod
- 全部用到的data

单个mod的结构是固定的(不按前文了)

```js
mod: {
  dataid: ---uuid---, // 数据id
  modid: 002 // mod-id, 可通过这个获取其_data(自带默认data), jade, less, js
}

通过modid判断一个对象是否为mod对象

渲染的过程如下

1. 现有locals
2. 遍历locals, 找到mod对象, 列出全部用到的mod
3. 遍历用到的mods, 通过dataid把data转换成真实数据, 通过modid获取mod默认的_data, jade, less, js
4. `$.extend(_data, data); mod.data = _data`
5. html渲染: jade.render(jadeStr, locals)
6. css渲染: 通过data增加less变量, `less.parse(mod.less)`
7. js渲染: 这是最麻烦的, 因为data不好处理, 见下一条

```

### data处理方法

### 总结

可以看到, 这些看起来都无比简单, 没有难度, 关键是它在写法上不需要任何专门的学习, 同时也保证了组件的通用性, 不过这还只是我口头说说, 具体是否真能像我想的这么美好这么屌还得等我写出来再看..

需要解决的问题

- 组件宽度, 第一反应是组件宽度由网格css的`col`来定啊, 但没这么简单, 因为有些单图组件, 图片的宽度是定死的
- js单个与全部运行
- 如何使用jade的mixin和block
- 如何做到`include(#{var})`, (预处理模板)
