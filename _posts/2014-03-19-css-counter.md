---
layout: post
title: 博客自动分标题のCSS Counter
date: 8:57 2014/3/19
tags:
- css
---

大标题一
---

### 中标题一

### 中标题二

#### 小标题一
#### 小标题二

段落标题编号CSS实现
---

### markdown写法

```
大标题一
---

### 中标题一

### 中标题二

#### 小标题一
#### 小标题二
```

### CSS Counter

[CSS Counter](https://developer.mozilla.org/en-US/docs/Web/Guide/CSS/Counters)简直就是为了markdown写文章准备的, 不过你同样也可以用来自定义ol序号的样式

我的段落标题CSS:

```css
h1 {
  counter-reset: big;
}

h2 {
  counter-increment: big;
  counter-reset: mid;
}

h2:before {
  content: counter(big);
}

h3 {
  counter-increment: mid;
  counter-reset: small;
}

h3:before {
  content: counter(big)'.'counter(mid);
}

h4 {
  counter-increment: small;
}

h4:before {
  content: counter(big)'.'counter(mid)'.'counter(small);
}

h2:before, h3:before, h4:before {
  padding-right: 10px;
}

/* 缩进 */
h1, h2, h3 {
  text-indent: -10px;
}

```

#### 小标题一
#### 小标题二
#### 小标题三

### 中标题三

#### 小标题一
#### 小标题二
#### 小标题三

