---
layout: post
title: IE6 支持的方法一览
tags:
- javascript
---

整理一下 ie6 可以用的原生的方法, 暂时只记录了 string 和 array 的 prototype

下面写的结果并不代表全部, 而是常用的方法, 那些被遗弃和不常用的函数并没有被记录, 如(String.prototype.sub)这些

### IE6 支持的 `String.prototype`

- charAt, charCodeAt
- concat
- indexOf, lastIndexOf
- match, search
- replace
- slice
- split
- substr, substring
- toLocaleLowerCase toLocaleUpperCase toLowerCase toUpperCase

可以看到只缺失了 trim 这个重要函数, jQuery 的实现如下

```javascript
function trim(str) {
    var rtrim = /^[\s\uFEFF\xA0]+|[\s\uFEFF\xA0]+$/g
    return null == str ? '' : (str + '').replace(rtrim, '')
}
```

### IE6 支持的 `Array.prototype`

- concat
- join
- pop, push, shift, unshift
- slice
- splice
- sort
- reverse

缺少 indexOf, lastIndexOf, 以及那些常用 es5 基本函数, map, filter, forEach

简单的 Array.prototype.indexOf

```javascript
function indexOf(arr, item) {
    var len = arr ? arr.length : 0
    for (var i = 0; i < len; i++) {
        if (item === arr[i]) {
            return i
        }
    }
    return -1
}
```

简单的 Array.prototype.forEach

```javascript
function forEach(arr, fn) {
    var len = arr ? arr.length : 0
    for (var i = 0; i < len; i++) {
        if (false === fn(arr[i], i, arr)) break
    }
}
```

注意此处遇到 false 直接停止了, 在很多其他语言或框架中, 遇到 false 停止一般是一个选项, 比如 jQuery 的 `$.Callbacks` 中是以 `opt.stopOnFalse` 来决定的, 而 [lodash](https://github.com/lodash/lodash) 则是 false 肯定停止遍历, 我也是偏爱 lodash 的决定, 毕竟我们不需要这么多的 options, 这样也可以更简单的实现 es5 中的 every 和 some

简单的 arrayLike -> array 转换函数

在 arguments 和 nodelist 对象转换上尤为常见, 因此也记录一下

```
function makeArray(arr, ret) {
    ret = ret || []
    ;[].push.call(ret, arr)
    return ret
}
```

也是非常实用, 在记录一个常用的类型判断方法

```javascript
;{}.toString.call(arg)
```

可以检测出 `Boolean Number String Function Array Date RegExp Object Error` 这么多类型

之所以记录这个, 是因为我在写一个 mini-jQuery, 希望能兼容 IE6, mini-jQuery 是给没有用 jQuery 的前端项目, 离开 jQuery 就不会写前端 js 代码星人用的


### 测试方法

由于我并不知道怎么遍历 ie6 下的不可枚举属性, 所以用了最笨的方法, 测试代码:

```javascript
var pre = ''
var array = 'concat forEach filter map find join map pop push reduce reverse shift every slice some sort splice unshift indexOf lastIndexOf'.split(' ')
pre += 'array: <br><br>'
for (var i = 0, x; x = array[i++];) {
    if ([][x]) {
        pre += x + '\n'
    }
}
var string = 'charAt charCodeAt concat contains endsWith indexOf lastIndexOf match replace search slice split substr substring toLocaleLowerCase toLocaleUpperCase toLowerCase toUpperCase trim trimLeft trimRight'.split(' ')
pre += '<br><br><br>string: <br><br>'
for (var i = 0, x; x = string[i++];) {
    if (''[x]) {
        pre += x + '\n'
    }
}
document.getElementById('pre').innerHTML = pre
```

