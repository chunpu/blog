---
layout: post
titile: amd bundle
---

转换无非三种

### 1.

```javascript
define('/var/arr', [], function() {
  return []
})

// to

var arr = [];
```

非常简单

### 2.

```javascript
define('core', [], function() {
  var jQuery = {}
  return jQuery
})

// to

var jQuery = {}

```

同样简单


### 3.

```javascript
define('foo', ['./core'], function(jQuery) {
  jQuery.foo = {}  
})

// to

jQuery.foo = {}

```

甚至没有return要删


### 4.

```javascript
(function(window) {
  var Sizzle = {}
  // EXPOSE
  if (typeof define === 'function' && define.amd) {
    define(function() {
      return Sizzle
    })
  }
  // EXPOSE
})(window)

// to

(function(window) {
  var Sizzle = {}
})(window)
```
注意, 有expose, define就肯定在里面
也就是把EXPOSE中的东西删掉



规则一下子就出来了

1. 如果只有return语句, 那就是定义

2. 否则, 如果是多行且有return, 就去掉return, 取函数部分

3. 否则, 如果是多行且没有return, 直接取函数部分
