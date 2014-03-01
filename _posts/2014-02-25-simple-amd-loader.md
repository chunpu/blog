---
layout: post
title: 超简易AMD模块加载实现
date: 22:22 14/02/25
tags:
- javascript
---

最近深受打击，认识的几个菊苣工资都高的吓人，何时屌丝才能够钱娶到心爱的妹子啊

之前写网页，都是直接一个inline的style和一个inline的script搞定的

不开玩笑了..模块加载的好处是显而易见的，解决模块依赖的关系，线上联调(加url代理)方便

[AMD](https://github.com/amdjs/amdjs-api/wiki/AMD)就是一个模块加载的方案

AMD全称(Asynchronous Module Definition), 三个单词每个都是关键字，一个是异步，一个是模块，最后是定义

AMD暴露在外的函数只有一个: `define`

用requirejs的可能会说不是`require([], funciton() {xxx})`的么，但我们发现require完全可以改成define，虽然在语义上确实说不大通，我明明是要用这些个库，凭什么让我们用define呢，require才对嘛

但AMD标准就是如此，因此我们在真实环境中也应该用define而不是require

define的重载非常多

比如可以直接用来纯依赖

mod1.js

```javascript
define(['./sub-mod1', './sub-mod2', './sub-mod3'])
```

可以完全不写factory

说到factory可能有人不明白了

这里要说到define函数的三个参数

- `id` 给一个标志，这样别人通过id获取这个模块，而不是通过路径关系获取
- `dependencies` 依赖，这是一个数组，包含自己需要的模块
- `factory` 一个函数，这个函数的参数就是前面依赖的返回值

id这个值是可选的，甚至大部分时候都是不用的，也许我们可以在获取jquery的时候define它为jquery，大多数时候子模块都是不需要id的, 一般顶级模块会用id

dependencies需要注意的就是, 首先它没有`.js`这个后缀。其次是dependencies如果是相对路径，必然以`.`开头，我看了下AMD上面貌似没有定义如果不写`.`是表示绝对还是相对路径还是id，总之，如果是相对路径，那第一个字符必然是`.`，这点可以参照jQuery的文件

关于路径处理，AMD的WIKI举了俩例子

```
a/b/c + ../d -> a/d
a/b/c + ./e -> a/b/e
```

当然dependencies也是可选的,因为可以直接这样

```javascript
define(function() {
  return '0.1.2'
})
```

但这有点说不过去，因AMD的wiki上写了CommonJS wrapping的方式 (就是nodejs那个方式), 它也是一个函数参数

```javascript
define(function(require, exports, module) {
  var a = require('a')
  exports.action = function() {}
})
```

不过反正function有length这个属性，估摸也是可以重载的。。

一个js文件兼容define的写法, 假设我们写一个log函数

```javascript
!function(f) {
  if (typeof define === 'function' && define.amd) {
    define(f)
  } else {
    window.log = f()
  }
}(function() {
  function log(str) {
    if (window.console && typeof console.log === 'function') {
      console.log(str)
    }
  }
  return log
})
```

看着貌似长了一大截(加上commonJS还会更长。。)，但说不定加上这个别人就愿意用你的库惹

我的超简易AmdJS在这里<https://github.com/chunpu/AmdJS>, 可以成功加载jQuery，加载完我就对继续实现AMD失去兴趣了，唉我又是五分钟热度，实现基本功能就跑的那种

原本本文是写其原理的，但其原理实在是简单，就是动态的创建script元素插进去，onload后执行，最后删掉(看起来干净)。反倒是路径处理有点麻烦

因此本文写到这里已经兴趣索然，烂尾结束！
