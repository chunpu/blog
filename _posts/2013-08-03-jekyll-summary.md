---
title: jekyll小结
layout: post
tags:
  - jekyll
---

又玩了一会jekyll, 感觉挺不错的,但功能不全.

首先是因为Liquid这个模板并不是一个全功能的模板.使用起来并不是很舒畅.

然后就是它的`Front-matter`,也就是每篇markdown中最开头的头部信息.layout是必须写的,我们无法通过修改_config.yml来设置默认的layout.比如我设置

```yaml
layout_defaults:
 -
  layout: page.html
 -
  path: _posts
  layout: post.html
```

这使得我每次写文章都要加上`layout: post`这句废话, 尽管我就是在_post文件夹下写的.这不是一个好的设计,而且破坏人们写作的心情,不少人提出了这个问题, 比如这个issue<https://github.com/mojombo/jekyll/issues/453>, 作者注意到了却没去实现.

jekyll允许通过插件来自定义新功能, 可惜github的jekyll是不支持的,这肯定不能支持, 不然写个死循环就要跑坏github的服务器了.所以jekyll虽然很纯净, 但少了一些基本的功能.

写完这篇我在手机上阅读时发现有横向滚动条,原来是没有断词

```css
word-wrap: break-word;
```

加上自动断词即可解决.

横向滚动条在移动设备的响应式网页上是大忌.话说今天在写media query的css的时候,偶才知道屏幕分辨率不等同于浏览器分辨率

浏览器的分辨率通过`window.screen.height`和`window.screen.width`获取.

关于设备宽度和浏览器宽度的区别可以参照<http://stackoverflow.com/questions/5716066/difference-between-width-and-device-width-in-css-media-queries>

可以通过`window.devicePixelRatio`获取设备和浏览器像素密度的比例.

比如小米2的像素比是2, 小米2的分辨率是`720 x 1280`, 于是它的浏览器分辨率就是`360 x 640`.640 x 960的iphone也是2.但768 x 1024的ipad就是1, 因此用ipad横屏浏览桌面版网页是不会有x轴滑动条的.但ipad3则是2, 因为超过1000的分辨率是没有意义的.这是因为大多数定死宽度的网页都采用940-980px之间的数值.所以不要担心手机分辨率太高用media-query分辨不出来.屏幕越小, 分辨率越大, 则`window.devicePixelRatio`越大, 这样保证了使用viewport的时候字体大小是正常的.
