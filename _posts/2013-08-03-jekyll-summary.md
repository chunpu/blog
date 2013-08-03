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
