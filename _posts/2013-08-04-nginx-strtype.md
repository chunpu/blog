---
title: nginx基本数据类型
layout: post
---

我的参考来自这里<http://tengine.taobao.org/book/chapter_02.html#id3>

`ngx_str_t`
-----

```c
typedef struct {
    size_t      len;
    u_char     *data;
} ngx_str_t;
```

声明文件: `core/ngx_string.h`

最小的数据结构, 就是一个字符串,跟传统以`\000`来判断字符串结尾不同,`ngx_str_t`通过长度来判断字符串结尾.这样做有什么好处呢?

- 减少计算字符串长度的操作,不再需要老是用strlen来算长度了.

- data是指针, 使用的时候指过去就行了, 不需要copy字符串.

- 坏处是glibc提供原生字符串的api, 使用时需要先吧`ngx_str_t`变成原生字符串才能用,略显啰嗦.

### `ngx_str_t`的api

```c
ngx_string(str)

// 本质就是

#define ngx_string(str) {sizeof(str) - 1, (u_char *)str}
```

宏定义,构造函数, 通过一个标准字符串构造出一个nginx字符串.

只能用来初始化.

```c
ngx_str_t str = ngx_string("hello");

// error
ngx_str_t str;
str = ngx_string("hello");
```

这是因为结构体初始化后不能直接赋值

我实在不理解为何要这么做, 直接写有多打几个字么?可能是c开发者都喜欢建立自己的世界观吧.

```c
ngx_null_string
```

宏定义`ngx_str_t`空字符串,也就是`{0, NULL}`


```c
ngx_str_set(&str, text);

// ---

define ngx_str_set(&str, text) str->len = sizeof(text) - 1; str->data = (u_char *)text
```

此为宏定义,str为`ngx_str_t`的指针, 用法是把标准字符串赋值给`ngx_str_t`这个str指针.

这个text是常量字符串,而不是指针,否则len永远是3.

然后ngx就提供了很多ngx前缀的字符串方法.比如`ngx_atoi`, `ngx_strcmp`等, 不知道这样重复造轮子有啥真正的好处..
