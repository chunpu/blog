---
tags: jekyll
date: 22:36 2013/8/1
title: jekyll学习
---

jekyll真的是太棒了,特别是代码显示部分,我指的是jekyll官网的css效果,我直接复制粘贴了他的css文件.

一开始我觉得jekyll所有post要求统一命名,比如`2013-08-01-hello-world.md`感觉很蠢.现在觉得挺好的,因为对于一篇随笔,你可能只是要一个时间.

我已经在考虑是否要完全使用jekyll写博客了,因为github的pages就是用jekyll渲染的,不用白不用.

放弃自己写博客生成是因为已经开始工作,这些东西看起来有点幼稚,而且不再有大量的时间放在自己的兴趣上了.需要赶紧补一下C语言.

挑选更纯粹的工具,才能更加高效.下面是学习jekyll的心得, 好的话以后就不折腾博客了..

文件夹结构
-------

```
.
├── _config.yml
├── _drafts
|   ├── begin-with-the-crazy-ideas.textile
|   └── on-simplicity-in-technology.markdown
├── _includes
|   ├── footer.html
|   └── header.html
├── _layouts
|   ├── default.html
|   └── post.html
├── _posts
|   ├── 2007-10-29-why-every-programmer-should-play-nethack.textile
|   └── 2009-04-26-barcamp-boston-4-roundup.textile
├── _site
└── index.html
```

`_posts`文件夹就是放markdown格式的博客的.

`_includes`用于放子模版, 一般不需要.

`_layouts`用于放父模板,默认有`default.html`和`post.html`

`_sites`是自动生成的,需要用`.gitignore`忽略

`_drafts`你可以建一个草稿文件夹放还没写完的文章.

可以新建其他文件夹,并且像静态服务器那样访问他们,
比如可以新建一个about文件夹,里面放一个`index.html`放置自己的简历或者自我介绍,
同样你也可以直接在跟目录下放一个about.html
我试了下放markdown文件,结果报错了,不知道哪里出了问题.

_config.yml
---------

配置文件,配置文件完全可以不管,这是非常好的,因为它提供默认值.

其中默认的键值pygments是代码高亮,我特别喜欢.

不得不重提一下markdown解析器,`markdown: redcarpet`, 这是能像Github-Flavored-Markdown解析出的结果那样.更重要的是code也能用python那种注释来包含,当然也是github的风格.这让我觉得爽极了,我以前的博客代码高亮是靠marked这个插件自己猜的..非常不好.

jekyll的redcarpet配合highlight样式以及Monokai(sublime默认配色),我觉得已经完美.看起来非常舒服.
平时上网搜问题,看到别人排版拙鸡的博客和挤在一坨的代码,不满的同时有点小小的优越感..

要注意解析器不能乱写,redcarpet是github自带的.github支持的jekyll插件在<https://help.github.com/articles/using-jekyll-with-pages>列全了,而且有版本号, 本地测试的时候最好也要下一样的版本.

其他配置等了解后再补充

front-matter
-----

```
---
layout: post
title: Blogging Like a Hacker
```

这种放在最前的博客信息我一看就明白,因为我已经也是这样做的,原因是因为单靠ctime,mtime啊来判断时间并不靠谱.

jekyll的头部信息强制要求是yaml格式的.我觉得这点很棒.规范了格式.可以这么写

```
---
layout: post
title: Jekyll is Cool
tags:
  - jekyll
  - github
  - blog
categories:
  - blog
---
```

要注意的是这个头部信息是可选的,可写也可以不写.

预设的博客格式属性有如下几个

- layout: 这个不多说.就是父模板

- pemalink: 访问该博客的连接, 默认是像这样子`/2013/08/01/hello/world.html`

- published: boolean类型,默认是true

- category: 分类

- categories: 可以是列表,就是属于多个分类.

- tags: 标签.(说实话我搞不清categories和tags的区别, 觉得他们功能重合了..)

- date: 日期, 可以覆盖外面的文件名日期, 日期写法很丰富,反正notepad中F5出来的日期是可以用的.

文章模板
-----

文章模板是用一个叫Liquid template language做的,初步看来这不是一个全功能模板.不过语法属于大众语法,
`{{ xxx  }}`表示插入, `{% some code %}`表示执行.没有太多学习成本.

提供的默认变量很多

### 大变量site.

- `site.posts`: 所有博客的hash.

- `site.pages`: 所有page, 元素的属性page和post应该没啥区别

- `site.related_posts`: 显示相关联的10个博客,不知道这相关联是怎么取的, 目测是时间..

- `site.categories.CATEGORY`: 显示这个分类下的博客列表

- `site.tags.TAG`: 显示这个标签下的博客列表

- `post`: site.posts单个值, 有url, title, excerpt等子属性, 注意excerpt就是预览文本,取前面一小段.

- `page`: 这是针对单个文章的全局变量, 作用等同于post, 同样拥有url, title, excerpt等属性.

### Paginator

貌似中文叫页码.其中提供了获取上一篇 和 下一篇的功能

- paginator.per_page

- paginator.posts

- paginator.total_posts

- paginator.total_pages

- paginator.page

- paginator.previous_page: 上一页

- paginator.previous_page_path: 上一页路径

- paginator.next_page: 下一页

- paginator.next_page_path: 下一页路径

注意, 这个paginator只能在index文件中使用, 坑爹啊.

paginator原来是总的页数的意思, 可以在_config.yml中设置, 比如

```yaml
paginate: 5
```

那就是5篇文章一页.

上一篇和下一篇分别是用`page.preivous.url` 和 `page.next.url`获取的.

jekyll的文档中貌似并没有说这俩个page的自属性.

暂时就看到这些文档, 感觉够用了, 剩下的就是继续去了解一下那个模板了.






