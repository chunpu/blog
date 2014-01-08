---
layout: post
title: co中的yield的n种重载
date: 14:07 2014/1/8
tags:
- javascript
---

yield在generator中不过是中断函数, 并且返回一个`return.value`的值, 但是到了[co](https://github.com/visionmedia/co)中, yield简直是无所不能, 简直是专治各种异步

来看这几个例子

还是准备之前一直用的delay函数(用setTimeout模拟延迟就是为了浏览器上也可以实验)

```javascript
function delay(time) {
  return function(cb) {
    setTimeout(function() {
      fn(null, time)
    }, time)
  }
}
```

####重载1. 首先是普通的thunk函数,也就是用的最多的情况

```javascript
co(function* () {
  yield delay(100)
  yield delay(200)
})(function(err, d) {
  console.log(err, d) // null, 200
})
```

####重载2. thunk数组

```javascript
co(function* () {
  yield [delay(300), delay(200), delay(250)]
})(function(err, d) {
  console.log(err, d) // null, [300, 200, 250]
})
```

####重载3. thunk字典

```javascript
co(function* () {
  var o = {
    first: delay(300),
    second: delay(200),
    third: delay(250)
  }
  yield o
})(function(err, d) {
  console.log(err, d) // null { second: 200, third: 250, first: 300 }
})
```

这里一直在提thunk这个词,那到底thunk是啥呢

thunk是think的过去时,表示计算机一直在think(计算或者等待),等到结果出来了,就表示think结束了,说白了就是事件机制

thunk函数又是啥呢? delay函数就是典型的thunk函数, 来看它的特点, thunk函数的参数不带回调, 但是返回值却是一个参数为回调函数的函数, 也就是说, 只要是thunk函数, 都可以这么用

`thunk(args)(next)`

next也是一个函数, 但co会托管所有的next, 它一般长这样

```javascript
function next(err, data) {
  if (err) cb(err)
  else doSomething()
}
```

因此可以看出co其实也是一个thunk函数, 因为co都是这么用

```javascript
co(function* () {
  ...
})(function() {})
```

显然, 接受thunk函数的yield也同样能接受GeneratorFunction这种参数


####重载4. GeneratorFunction

```javascript
var gen = function* (time) {
  yield delay(time)
  yield delay(time + 100)
}
console.time(1)
co(function* () {
  yield gen(140)
  yield delay(100)
})(function() {
  console.timeEnd(1) // 1: 685ms
})
```

接受他们的混合也是可以的

```javascript
var gen = function* (time) {
  yield delay(time)
  yield delay(time + 100)
}
console.time(1)
co(function* () {
  yield {
    first: gen(200),
    second: [delay(200, 100, 150)]
  }
})(function(err, data) {
  console.log(err, data) // null { second: [ 200, 100, 150 ], first: { key2: 200, key1: 300 } }
  console.timeEnd(1) // 1: 508ms
})
```

我也不是有意搞这么奇怪的(逃..

有人也许会说[async](https://github.com/caolan/async)也能满足这些个需求

但这其实是不可能的, 因为yield能真正中断函数, 而普通函数不行

```javascript
co(function* () {
  var a = delay(200)
  var b = delay(a + 100)
})
```

在普通函数中`a + 100`, 早就在delay没执行完前被计算了, 但是在generator中, `var b = delay(a + 100)`是真真切切的被中断了

因此yield真正实用之处就是中断表达式求值

那co是怎么实现这么多重载的呢?

co多种重载的实现
---

实现这些重载非常简单, 个人觉得比TJ菊苣的co要好懂太多(捂脸), 代码如下

### co核心

```javascript
function co(Gen) {
  var gen = Gen
  if (isGeneratorFunction(gen)) {
    gen = gen()
  }
  return function(cb) {
    next()
    function next(err, arg) {
      if (err) {
        cb(err)
      } else {
        if (gen.next) {
          var ret = gen.next(arg)
          if (ret.done) {
            cb(null, arg)
          } else {
            // 新增toThunk
            toThunk(ret.value)(next)
          }
        }
      }
    }
  }
}
```

其中`toThunk`函数是之前没有的

toThunk做的就是分辨类型, 按参数类型把`return.value`变成真正的thunk函数

```javascript
function toThunk(o) {
  var fn
  if (o.constructor && o.constructor.name.indexOf('GeneratorFunction') === 0) fn = co
  else if (Array.isArray(o)) fn = arr2Thunk
  else if (Object.prototype.toString.call(o) === '[object Object]') fn = obj2Thunk
  if (fn) o = fn(o)
  return o
}
```

thunk数组转成thunk函数

```javascript
function arr2Thunk(arr) {
  return function(cb) {
    var len = arr.length
    var result = []
    arr.forEach(function(a, i) {
      var fn = toThunk(arr[i])
      fn(function(err, d) {
        if (err) cb(err)
        else {
          len --
          result[i] = d
          if (len === 0) cb(null, result)
        }
      })
    })
  }
}
```

thunk字典转成thunk函数

```javascript
function obj2Thunk(o) {
  return function(cb) {
    var arr = Object.keys(o)
    var len = arr.length
    var result = {}
    arr.forEach(function(a, i) {
      var fn = toThunk(o[arr[i]])
      fn(function(err, d) {
        if (err) cb(err)
        else {
          len --
          result[arr[i]] = d
          if (len === 0) cb(null, result)
        }
      })
    })
  }
}
```

co中的yield还可以重载promise对象等

不过只要能理解co中的yield的值永远是thunk对象就足够了
