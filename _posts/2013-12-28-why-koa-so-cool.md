---
layout: post
title: KOA框架为何这么屌
date: 23:49 13/12/28
tags:
- javascript
---

最近逢人就说KOA用着好爽啊，但每次都被泼冷水

首先是看不起Nodejs派：“能跟我大structs比?”。我不得不低头，这真比不起，express就因为太小没法和那些"大框架"比，KOA比express还小几倍，核心部分就100来行，说出来岂不是让java程序员笑死，自讨没趣

其次是express派：“express就够屌了，我看KOA也就和express一样嘛”，我马上回到，不是有generator么，多方便。express派说：那不过是用了点ES6的特性，KOA本身没啥进步的。我一时语塞

不管别人怎么看，KOA确实用着爽，忍不住扯KOA的两点好

### 1. yield数组

yield处理逐个执行的异步函数上次已经说过了

yield后面还可以跟数组，如下

```javascript
var fs = require('fs')
var app = require('koa')()
var readFile = function(dir) {
  return function(fn) {
    fs.readFile(dir, fn)
  }
}
app.use(function* () {
  var arr = yield ['1.txt', '2.txt', '3.txt'].map(function(path) {
    return readFile(path)
  })
  this.body = arr.join(',')
})
app.listen(8000)
```

其实这不过是[上上篇]({{ page.previous.previous.url }})中提到的不太难的多个平行异步函数获取总的回调函数的yield写法，不过也已经简洁到令人赞叹的地步了

试想下如果我们同时要获取数据库中多个内容，代码也会非常简洁

可问题来了，如果这其中有异步函数出错怎么办？

### 2. KOA中的错误处理

KOA中的错误处理才是真正让人叹为观止的地方，牛逼的同学可以看看[generator’s throw feature](http://wiki.ecmascript.org/doku.php?id=harmony:generators#methodthrow)

直接上写法

```javascript
var fs = require('fs')
var app = require('koa')()
var readFile = function(dir) {
  return function(fn) {
    fs.readFile(dir, fn)
  }
}
app.use(function* (next) {
  try {
    yield next
  } catch(e) {
    return this.body = e.message || "I'm dead"
  }
})
app.use(function* () {
  var arr = yield ['4.txt', '2.txt', '3.txt'].map(function(path) {
    // 4.txt不存在
    return readFile(path)
  })
  this.body = arr.join(',')
})
app.listen(8000)
```

可以看到访问网页的返回结果`ENOENT, open '4.txt'`

我们在app的匹配栈上直接给`yield next`套上`try catch`

当然KOA本身也是会处理这些错误的，比如还提供了`app.onerror`这些东西，甚至自动做了404或者500的判断，源码如下

```javascript
// delegate
this.app.emit('error', err, this);

// force text/plain
this.type = 'text';

if ('ENOENT' == err.code) err.status = 404;

// default to 500
err.status = err.status || 500;

// respond
var code = http.STATUS_CODES[err.status];
var msg = err.expose ? err.message : code;
this.status = err.status;
this.res.end(msg);
```

但我们往往需要自己处理错误: 处理数据库错误，处理文件读取错误，读取解析错误... 如果用自带的错误处理肯定会有人不解为什么在`app.onerror`中就不能用酷炫的`this.body=`了呢？也会疑惑`ctx.res`用了之后怎么会二次出错

因此我觉得在最前面加上next函数的`try catch`包裹非常有必要，这样我们可以统一的处理这些错误，任何yield错误都会被抓到，我们只需分析`e.message以及e.code`即可，我们甚至可以玩出很多花样来，利用`this.throw()`和`new Error()`来定义自己的错误，简化错误管理，还能对错误进行归类。

### 代码要写在try catch外面

虽然用try catch 捕捉next错误很爽，但我们知道不管是try块中的代码还是catch块中的代码，都是无法让V8引擎进行任何优化的(具体原因忘了)，不能优化的函数比优化的函数会慢上好几倍，也就是说try和catch中的代码比外面的慢很多倍。但是如果是调用try catch块外面的函数就不会有这个问题了。因此那段头部代码应该这么写

```javascript
function errHandler(e) {
  ...
}
app.use(function* (next) {
  try {
    yield next
  } catch(e) {
    errHandler(e)
  }
})
```

不要小看这个，不信的话可以做这个试验

```javascript
console.time(1)
try {
  throw new Error()
} catch(e) {
  for (var i = 0; i < 1000000000; i++) {
    var p = i % 2
  }
}
console.timeEnd(1)

// 取出来放在外面
console.time(2)
try {
  throw new Error()
} catch(e) {
  run()
}
console.timeEnd(2)
function run() {
  for (var i = 0; i < 1000000000; i++) {
    var p = i % 2
  }
}
```

结果是

```
1: 8352ms
2: 1294ms
```

差距还是相当大的

有了`错误管理`和`yield`这俩大杀器，Nodejs中被人诟病的缺陷就足以解决一大半!
