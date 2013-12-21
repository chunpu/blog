---
layout: post
title: Harmony Generator, yield, ES6, co框架学习
date: 14:41 2013/12/21
tags:
  - javascript
  - nodejs
---

harmony generator是ES6中的新玩具, 为了更好的避免回调金字塔, 学习这个特性是重要的, 这个特性在各大浏览器以及nodejs中已经都可以使用

nodejs中需要`0.11.*`版本, 官网找不到可以用[nvm](https://github.com/creationix/nvm)来下载(nvm install v0.11.9), 同时启用harmony generator还需要使用`node --harmony`, 为了方便, 我们一般会先取个别名`alias node='node --harmony'`

在chrome中使用该特性需要先打开`chrome://flags/`, 搜索`harmony`, enable之, 重启chrome即可

准备工作好了, 进入正题(建议使用chrome, 方便调试玩耍)

最简单的yield用法

```javascript
function* Hello() {
  // 我习惯用大驼峰命名因为这就好比generator的构造函数
  yield 1
  yield 2
}

var hello = Hello() // hello 是一个generator
var a = hello.next() // a: Object {value: 1, done: false}
var b = hello.next() // b: Object {value: 2, done: false} 
var c = hello.next() // c: Object {value: undefined, done: true}
```

可以看到hello的原型链中总是有一个next函数, 每次运行都返回yield后面的值, 只是不是单纯的yield后面的值, 而是放在一个对象的value键中, 同时我们注意到对象中还有另一个键`done`, Hello函数中有两个yield, 因此前两个done的值为false, 表示`我还没有结束呐!`, 最后一个done为true, 表示我已经结束了! 如果继续运行`hello.next()`则会报错`Uncaught Error: Generator has already finished`

很明显的说`yield就是相当于是另一种return`, return使用时函数就结束了, 而使用yield的时候, 函数会卡在那个yield的地方, 等待下一个next

你可能会问,这到底有啥用呢? 难不成像那些教程中求Fibonacci函数那样一个一个next着玩?

代替回调金字塔
---

要理解一个语言,必须知道这门语言为什么会产生. 同理,理解这个新特性,也需要知道它是为了解决怎样的难题. 本文上来就说到harmony是用来解决回调金字塔的, 那究竟如何解决的呢

先写出一个需要耗时的异步函数: delay

```javascript
funciton delay(time, cb) {
  setTimeout(function() {
    cb && cb()
  }, time)
}
```

我们并不是讨论同个耗时函数同时运行, 并取得最终回调的情景, 因为这种情景太简单了

```javascript
var len = 3
function cb() {
  len --
  if (len === 0) {
    console.log('finish')
  }
}

delay(200, cb)
delay(1000, cb)
delay(500, cb)
```

我们讨论的是逐个按顺序运行

```javascript
delay(200, function() {
  delay(1000, function() {
    delay(500, function() {
      console.log('finish')
    })
  })
})
```

上面程序看上去很短,不过是因为还没有加入参数和错误处理, 以及他的回调才3个,如果多起来函数就会斜的像金字塔了

利用harmony generator

首先我们需要改造delay函数

```javascript
function delay(time) {
  return function(fn) {
    setTimeout(function() {
      fn()
    }, time)
  }
}
```

可以看到新delay变成了一个返回刚才delay的函数, 而且参数变成了一个, 就是time, 回调函数成了返回函数的参数

借助co函数(后面详解), 我们可以这样用

```javascript
console.time(1)
co(function* () {
  yield delay(200)
  yield delay(1000)
  yield delay(500)
})(function() {
  console.timeEnd(1) // print 1: 1702.000ms 
})
```

这样就完成了顺序的异步执行, 写法非常简洁

co函数
---

为什么是co函数呢? 嗯, 这篇博文完全是为了看懂TJ大神的[co框架](https://github.com/visionmedia/co), 即将大热的[koa框架](http://koajs.com/)也是基于这个小小的函数

那co函数究竟是怎么写的呢?

最简单的co

```javascript
function co(GenFunc) {
  return function(cb) {
    var gen = GenFunc()
    next()
    function next() {
      if (gen.next) {
        var ret = gen.next()
        if (ret.done) { // 如果结束就执行cb
          cb && cb()
        } else { // 继续next
          ret.value(next)
        }
      }
    }
  }
}
```

短短十行就完成了co的主要功能, 不过这个co是不支持任何参数的, 因此会如此简短

`ret.value(next)`这行代码可以说是co中最重要的地方, 这句话将使得以后大部分异步函数都变成这个鸟样! 比如读文件

```javascript
function read(file) {
  return function(fn){
    fs.readFile(file, 'utf8', fn);
  }
}
```

也就是说使用co的异步函数都是返回一个函数, 且该函数的参数是异步执行后的回调, co通过不断的执行, 传递next, 执行, 传递next, ... , 结束

看起来异步函数都被包了一层, 换来的好处是参数少了一个(不用把回调函数写里面了), 但是回调的结果在哪里呢?

大多数时候需要顺序调用的异步函数都不会这么简单, 每一步都需要来自上一步的返回值

比如最简单的就是这样:

```javascript
function delay(time, cb) {
  setTimeout(function() {
    cb && cb(time)
  }, time)
}

delay(200, function(time) {
  delay(time+100, function(time) {
    delay(time+100, function(time) {
      console.log(time) // print 400
    })
  })
})
```

那yield如何传值呢? 先来看这个小demo

```javascript
var gen = (function* (num) {
  console.log(num) //  print 11
  var a = yield 1
  console.log(a) //  print 22
  var b = yield 1
  console.log(b) // print 33
})(11)

gen.next()
gen.next(22)
gen.next(33, 44)
```

可以看出, yield的返回值和yield后面的东西毫无关系(但不能没有, 不然是undefined), 返回值就是next函数中第一个参数

我们稍稍改一下co函数就能完成回调使用之前回调参数的需求

```javascript
function co(GenFunc) {
  return function(cb) {
    var gen = GenFunc()
    next()
    function next(args) { // 传入args
      if (gen.next) {
        var ret = gen.next(args) // 使用args
        if (ret.done) {
          cb && cb(args)
        } else {
          ret.value(next)
        }
      }
    }
  }
}
```

就多了俩args, 于是我们就可以非常愉快的这么写!

```javascript
function delay(time) {
  return function(fn) {
    setTimeout(function() {
      fn(time) // time为回调参数
    }, time)
  }
}

co(function* () {
  var a
  a = yield delay(200) // a: 200
  a = yield delay(a + 100) // a: 300
  a = yield delay(a + 100) // a: 400
})(function(data) {
  console.log(data) // print 400, 最后的值被回调出来
})
```

如何在回调中传err
---

有一种说法: 一个程序, 三分钟一的代码都是在错误处理

nodejs中也有大量的错误处理

在nodejs官方文档中就有大量这样的例子

```javascript
fs.readFile('/etc/passwd', function (err, data) {
  if (err) throw err;
  console.log(data);
});
```

错误都放在回调函数中的第一个参数中, 如果不使用专门的框架, 我们需要大量的写这种代码

```javascript
if (err) {
  throw ..
  next(err) ..
} else {
  next(null, err)
}
```

简直冗长无比, 但也没有办法, 因为回调函数第一个放错误已经从不成文规定变成了大多数项目的明文规定

但有了co函数我们就不必这么麻烦了

首先把delay改成标准的函数

```javascript
function delay(time) {
  return function(fn) {
    setTimeout(function() {
      fn(null, time) // null 表示没有错误..
    }, time)
  }
}
```

改进co函数, 加上错误判断

```javascript
function co(GenFunc) {
  return function(cb) {
    var gen = GenFunc()
    next()
    function next(err, args) {
      if (err) {
        cb(err)
      } else {
        if (gen.next) {
          var ret = gen.next(args)
          if (ret.done) {
            cb && cb(null, args)
          } else {
            ret.value(next)
          }
        }
      }
    }
  }
}
```

使用新的co函数

```javascript
co(function* () {
  var a
  a = yield delay(200) // 200
  a = yield delay(a + 100) // 300
  a = yield delay(a + 100) // 400
})(function(err, data) {
  if (!err) {
    console.log(data) // print 400
  }
})
```

好了! co函数的雏形就这样完成了, TJ大大的真正的co框架还支持promise等写法, 不看看源码绝对可惜了

最后记一下函数是否是generatorFunction的判断方法

```javascript
function isGeneratorFunction(obj) {
  return obj && obj.constructor && 'GeneratorFunction' === obj.constructor.name
}
```
