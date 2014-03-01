---
layout: post
title: Promise原理解析与实现
date: 13:38 2014/3/1
tags:
- javascript
---

之前写过一篇`I promise, I resolve`, 但我连微博都没发, 因为我自己都觉得很扯淡, 现在看来果然是在扯淡

这里说的Promise是es6 harmony的Promise, 而非那个DOM Promise.
现在的Chrome两种Promise都支持, 但默认为DOM的Promise, 要想打开harmony模式, 还得要在`chrome://flag`中打开harmony (启用实验性 JavaScript)

分辨dom promise和harmony promise的方法就是在dev中输入`Promise(function(){})`

如果报错了说明是dom的promise, 不报错则为harmony的promise

因为dom的promise标准已经被删除, 而harmony的promise既可以在浏览器中用又可以将来在nodejs中用, 我们当然是选harmony的promise啦

现在我们来尝试用100行左右代码实现一下promise的大概功能

首先写出主要的`Promise`函数

```javascript
function Promise(resolver) {
  resolver(resove, reject)
}
```

我们都知道Promise的参数是一个函数, 其参数是promise内部控制流程的resolve和reject

看到这里, 想必大家觉得很熟悉, 所有流控制的库貌似都是传一个表达 **继续往下传** 的内部函数, 说大白话就是 **我这里搞定了, 你继续** 的回调函数

比如express4之前用到的[connect](https://github.com/senchalabs/connect), 其中的`app.*()`中的function第三个参数就是next, 可以用来移至下一个路由栈继续匹配, 而promise则使用了两个内部函数, 一个表达流程正确的resolve(解决了), 另一个是流程失败的reject(拒绝了)

虽然外观略不同, 但不管是connect还是promise, 其内部都有一个stack或者queue的东西保存着全部的流, 在js中显然也就是一个数组

比如express中可以这么链式的写

```javascript
app.use(function(req, res, next) {
  next()
}).get('/xxx', function(req, res, next) {
  next()
}).use(function() {
})
```

其整个路由栈都被存入一个数组, 在next的时候移到下一个

而Promise的链式用法则为

```javascript
// 先封装一个返回promise的函数
function delay(time) {
  return new Promise(function(resolve) {
    setTimeout(resolve, time)
  })
}

delay(100).then(function() {
  return delay(200)
}).then(function() {
  return delay(300)
})
```

promise的链式由then中的resolve返回值加入, 而非一开始就全部塞入, 这就是promise和express中next的主要区别

继续试着实现promise

```javascript
function Promise(resolver) {
  resolver(resolve, reject)
  var queue = [] // 保存链式调用的数组
  this.then = function(resolve, reject) {
    queue.push([resolve, reject]) // 把then中的resolve和reject都存起来
  }
}
```

我们还没有写resolve和reject这两个内部函数呢, 这俩函数作用完全一样, 只不过一个表示正确, 一个表示错误, 我们完全可以用一个类似connect中的next来表达这两个函数

```javascript
function resolve(x) {
  next(0, x) // 用0来告诉next是resolve
}
function reject(resson) {
  next(1, reason) // 用1告诉next是reject
}
```

那这个控制流程的next到底该啥样呢, 我们都知道, next的作用不过是去调用queue中下一个函数而已

```javascript
function next(i, val) {
  // i 仅仅是用来区分resolve还是reject, val是值
  while (queue.length) {
    var arr = queue.shift() // 移出一个resolve和reject对, 也就是[resolve, reject]
    arr[i](val) // 执行之, val是唯一参数
  }
}
```

为何要while呢? 因为promise可以不停的then下去, 只不过传下来的都是resolve中的值都是undefined罢了, 因此我们用while来用光全部的resolve

问题就来了, 如果我们这么写

```javascript
var p = new Promise(function(resolve) {
  resolve('ok')
})
p.then(function(x) {
  console.log(x)
})
```

因为完全没有延迟, 显然resolve先走了, 而resolve执行的时候, queue中还没有函数去接它, 这个时候就then就不可能触发了

因此要么把resolve的值存起来, 要么就是让resolve肯定晚于后面的then执行

我这里偷一下懒, 用一下setTimeout

```javascript
function(i, val) {
  setTimeout(function() {
    while (queue.length) {
      var arr = queue.shift()
      arr[i](val)
    }
  })
}
```

这样resolve的出栈动作就肯定比进栈晚了, 不过这样写虽然很简洁, 但肯定有隐患(只不过我还没发现)

那如何让Promise支持链式调用呢? 这也不难, 我们只需将其执行结果存起来, 帮它then下去即可

```javascript
function next(i, val) {
  setTimeout(function() {
    while (queue.length) {
      var arr = queue.shift()
      if (typeof arr[i] === 'function') {
        try {
          var chain = arr[i](val)
        } catch (e) {
          return reject(e)
        }
        if (chain && typeof chain.then === 'function') {
          // 一般来说链式的话resolve返回值为一个promise对象
          // 所谓promise对象, 其实不过是 {then: function() {} }
          // 也就是一个含有then函数的对象
          return chain.then(resolve, reject)
        } else {
          // 注意, 此处resolve中同样可以返回一个普通值
          // 我们帮他包装成promise对象即可
          return Promise.resolved(chain).then(resolve, reject)
        }
      }
    }
  })
}
```

上面是一个加上错误处理的next函数, 错误处理在promise中, 就是转成reject即可

---

### 其它函数

Promise还有其它函数, 比如`Promise.all`, `Promise.resolved`等

我至今都不知道是`Promise.resolved`还是`Promise.resolve`

不过我觉得resolved听上去更对一点 (一个已经解决的承诺) , 而且chrome中也是这样的

实现这些附属函数特别简单

```javascript
Promise.resolved = Promise.cast = function(x) {
  return new Promise(function(resolve) {
    resolve(x)
  })
}

Promise.rejected = function(reason) {
  return new Promise(function(resolve, reject) {
    reject(reason)
  })
}

Promise.all = function(values) {
  var defer = Promise.deferred()
  var len = values.length
  var results = []
  values.forEach(function(p, i) {
    p.then(function(x) {
      results[i] = x
      len--
      if (len === 0) {
        defer.resolve(results)
      }
    }, function(r) {
      defer.reject(r)
    })
  })
  return defer.promise
}
```

这里的all用到了一个`Promise.deferred`的函数, 这个函数格外重要

---

### Promise.deferred

deferred的实现同样不难, 但其使用概率则是大大的, 可能比直接用Promise的几率还大

```javascript
Promise.deferred = function() {
  var result = {}
  result.promise = new Promise(function(resolve, reject) {
    result.resolve = resolve
    result.reject = reject
  })
  return result
}
```

deferred的使用方法非常顺手

```javascript
var def = Promise.deferred()
setTimeout(function() {
  def.resolve(222)
}, 1000)
def.promise.then(function(x) {
  console.log(x)
})
```

看到def, 才能看到Promise的精髓, 甚至jQuery反而提供defer作为主对象, promise不过是附属对象

我的完整Promise在[这里](https://github.com/chunpu/promise)

虽然目前Promise还不到100行, 但真正实现起来, 要比[co](https://github.com/visionmedia/co)那样借助yield的异步框架混淆很多, 我已经改了很多次, 但仍有bug, 这当然也跟我最近老打飞机有关, 已经有点神志不清

但是yield估计几年后才能用, 因此趁早学会Promise还是有必要滴
