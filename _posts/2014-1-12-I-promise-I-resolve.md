---
layout: post
title: I promise, I resolve
date: 12:59 2014/1/12
tags:
- javascript
---

Promise是当今大热, 很多人选择Promise来控制自己的异步流程, Promise甚至还进了下一代标准, 在最新的浏览器(chrome 32, 最新的Firefox)中已经自带

要注意的是Promise目前属于DOM Object, 虽然是es6的标准, 但只要不进v8, nodejs就不可能内置, (还是yield好

Promise的API可以参照[MDN中的文档](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise), 以及html5rock中的[这一篇](http://www.html5rocks.com/en/tutorials/es6/promises/)

首先要知道Promise最基本的用法, 把原生的Promise替换掉, 效果还一样就说明大致成功了

### 最简单的promise用法

```javascript
new Promise(function(resolve) {
  setTimeout(function() {
    resolve('hello')
  }, 1000)
}).then(function(d) {
  console.log(d) // hi
})
```

可以看出Promise的理念就如其名, 先把异步函数放在Promise对象中, 在then函数中执行

换句话说, 就是我给你这个承诺, 但不马上执行, 先存起来, then的时候就执行, 执行完后告诉你是resolve了还是reject了

### then函数

可以说Promise对象中最重要的属性就是`then`, then是一个函数, 其参数为`then(resolve, reject)`

resolve和reject也是函数, resolve函数中的参数就是上一次执行带来的参数, 注意, 一般来说是没有第一个参数是error的习惯的, 因为promise有reject

Promise当然不会是先存起来再then执行这么简单, 这也太弱了, 它更大的特点是可以不断的then

### 链式then

```javascript
new Promise(function() {
  ...
}).then(resolve).then(resolve).then(resolve)...
```

Promise的魅力就是它可以链式的then下去, 当然像上面这样then只有第一个有用, 后面都是空then

后面的then的执行内容取决于前一个then中resolve的返回值, 看这个例子

```javascript
new Promise(function(resolve) {
  setTimeout(function() {
    resolve('haha')
  }, 1000)
}).then(function(d) {
  console.log(d) // haha
  return new Promise(function() {
    setTimeout(function() [
      resolve('hahaha')
    }, 1000)
  })
}).then(function(d) {
  console.log(d) // hahaha
})
```

上面例子中第一个then返回了一个新的Promise对象, 然后接下来的then就能获取上面resolve的参数了

可以想见, Promise使用如此广泛, 却很少看到, 因为Promise都用在库的内部, 比如jQuery的ajax中, 他能随意中断或者执行, 不过是因为它每一步都返回一个Promise对象

那Promise对象到底有何特征呢? `then()`是如何判断resolve的return值是Promise对象呢?

其实Promise对象特征就一个, 上文也说过了, 就是有then属性

不信来做个试验

```javascript
new Promise(function(resolve) {
  setTimeout(function() {
    resolve('haha')
  }, 1000)
}).then(function(d) {
  console.log(d) // haha
  return {
    then: function(resolve) {
      setTimeout(function() {
        resolve('hahaha')
      }, 1000)
    }
  }
}).then(function(d) {
  console.log(d) // hahaha
})
```

上面例子的第一个resolve仅仅是返回了一个带then一个属性的对象, 也可以链式的then下去

可见不管Promise的库有多少, 不管他用catch还是fail来表示错误, 不管是怎样的接口, then是肯定要有的, 或者说, then就是promise的一切

知道了这个我们就可以动手实现一个自己的Promise了

```javascript
(function(exports) {
  exports.Promise = Promise
  var stack = []
  var resolves = []
  function Promise(fn) {
    stack.push(fn)
  }
  Promise.prototype.then = function(resolve) {
    resolves.push(resolve)
    run()
    return this
  }
  function resolve(val) {
    var fn = resolves.shift()
    var res = fn(val)
    if (res && res.then) {
      stack.push(res.then)
      run()
    }
  }
  function run() {
    var fn = stack.shift()
    if (typeof fn === 'function') {
      fn(resolve)
    }
  }
})(window)
```

去年小米面试说用20行实现一个Promise, 想必我这个还有没精简的地方, 我那时随便写了个不知道啥的东西交上去, 至今想起还有点羞愧

这里面最关键的函数就是resolve, 但功能简单明确, 执行resolve函数, 判断其返回值有没有then属性, push进执行函数数组中, 等待下一个run

闭包中存放两个数组, 一个是stack, 即第一个Promise的参数加上所有的返回值的then. 第二个是resolves参数, 是所有的then后面的resolve函数的集合

### Promise该如何用呢?

我们还是写一个delay函数

```javascript
function delay(time) {
  return {
    then: function(resolve) {
      setTimeout(function() {
        resolve(time)
      }, time)
    }
  }
}
```

这个delay比我们之前看到的toThunk型delay还多套了一层, 光这一点就说明Promise不如thunk

实现一个简单的waterfall流程

```javascript
new Promise(delay(600).then).then(function(d) {
  console.log(d) // 600
  return delay(d + 200)
}).then(function(d) {
  console.log(d) // 800
  return delay(d + 200)
}).then(function(d) {
  console.log(d) // 1000
  return delay(d + 200)
})
```

上述Promise都完全没有涉及reject, 就已经有点复杂了, Promise说到底, 真的是解决了缩进, 也仅仅是解决了缩进, 没有从根本上简化异步流程, 要想真爽, 还得靠yield


