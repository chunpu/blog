---
layout: post
title: jQuery初步介绍
tags:
- jQuery
- js
---

chainable

.show().hide();

$('div').draggable();

why $('document').ready(); not window.onload = function()

$('div').click() we can add so many  but how to delete

short => $(function)

$.map(array, fn); // oo? 不污染array

函数式编程

each
----------

$.each(arr, function() {alert(this)}) // value

$.each(dict, function(k, v){})

return what

$ doms has each function


event
----
change, onclick moveover

触发条件

疑问?  document.querySelectors?

$对象是否可呼吸

最难得不分 this的环境

why css

because we can handle nodelist and css hash

嵌套

$("div", "ul")

尝试输出所有function的this环境

$('div').onclick(function() {
  $('ul', $(this)) ...this is click ul  相对选择器

css table:gt(3):lt(2)  this is css?

$('select :selected') ??  没空格  why?
})



css selector
-----
:header  => h1 - h6

:input => input textarea select button

:text => input[type=text]

:password => input[type=password]

还有:radio :checkbox  :image

attr
-------
removeAttr  => display!

创建节点!
-------
var link = $('<a href="xxxx">baidu</a>')
$('div').append(link);

$('wqqwwdq') will return what

删除节点
----
remove();

如何从$ dom中选择dom  remove 会返回被删除节点

$('div').click(function() {
  $(this).text(xxx)
});

dom replace

replaceWith

包裹
wrap()


filter!
----

$('div').has('p')
$('div').not('.myClss')
$('div').filter('.myClass')

$('div').find('h3').eq(2).html('Hello')

next(),parent(),children();


.end() // 回退一级


$('div').find('h3').eq(2).html('Hello')
          .end().eq(4).css(xxx)

html() html('xx')  get and set

move
---

$('div1').insertAfter($('div2')) // return div1
$('div2').after($('div1')) // div2

appendTo and  append

copy
----

clone

小工具
---

trim()

  bind('click mouseover', function(){})

  return false可以组织冒泡..厉害

toggle(function() {}, function() {}); // click?

css3效果!
----
fadeIn, fadeOut, slideDown, slideUp, animate, stop
