---
layout: post
title: 为什么图片下面会多出3-5像素
tags:
- html
date: 13:49 2014/1/22
---

今天给图片加背景的时候, 突然发现底部总是多出来3px(chrome下)

觉得非常奇怪, 还专门把html拎出来, css全删了还是出现底边, 后来把`<!doctype html>`都删了, 那条底边才没. 大家告诉我是没有reset css, 但加上margin和padding的重置后依然没用

不只是chrome, firefox有5px的底边, IE也有4px左右的底边, 具体可以看问题的放大版<http://codepen.io/ftft1885/pen/CpDwa>

问题的原因就是img属于inline元素, inline元素一般都是文字, 而英文的小写字母`y`和`g`底部比别人多出来一段, 因此不管inline元素中有什么, 都会默认为y和g留出底部一段距离, 这段距离和字体大小有关, 比如把字体放大到129px后, 它外层的容易高度已经到了156px

不过知道了img表现同文字一样, 我们就能有很多解决办法

```css
img {
  display: block;
}
```

这样img就不是inline元素了, 不用像文字那样需要留底

```css
.box {
  font-size: 0;
}
```

把img的父元素设置字体大小为0, 字体不占位了, 因此也不会留底, 同理还有`line-height: 0;`

```css
img {
  vertical-align: middle;
}
```

这是最佳的解决方法, 因为这问题就是由`verticle-align: baseline;`引起的
