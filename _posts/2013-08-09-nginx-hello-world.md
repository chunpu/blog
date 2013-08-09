---
layout: post
title: Nginx模块开发之最简单的Hello模块
tags:
  - nginx
---

nginx模块开发并不是那么容易, 从行数上来讲, 淘宝给出的tengine给出的那个所谓hello模块的长度也到了245行, 要想真正独立写出这么多代码, 对于我来说是非常难的.

245行, 如果是nodejs, 已经可以写一个比较完善的文件服务器了. 要想完全理解这个hello模块, 有c基础的也怕是要花不少时间, 像我这样没有c经验的, 更是难上加难.

我决定写一个真正的hello模块,也就是最最简单的那种,能自己写出来,也算是至少nginx模块开发入门了.

这个hello模块作用就是当访问`/test`的时候, 返回一段固定的html代码(一个字符串).

此模块一共不到60行, 理解此模块比理解淘宝教程的模块要简单几倍.

先上全代码

```c
#include <ngx_config.h>
#include <ngx_core.h>
#include <ngx_http.h>

static char *set(ngx_conf_t *, ngx_command_t *, void *);
static ngx_int_t handler(ngx_http_request_t *);

static ngx_command_t test_commands[] = {
  {
    ngx_string("test"),
    NGX_HTTP_LOC_CONF | NGX_CONF_NOARGS,
    set,
    NGX_HTTP_LOC_CONF_OFFSET,
    0,
    NULL
  },
  ngx_null_command
};

static ngx_http_module_t test_ctx = {
  NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL
};

ngx_module_t ngx_http_test_module = {
  NGX_MODULE_V1,
  &test_ctx,
  test_commands,
  NGX_HTTP_MODULE,
  NULL, NULL, NULL, NULL, NULL, NULL, NULL,
  NGX_MODULE_V1_PADDING
};

static char *set(ngx_conf_t *cf, ngx_command_t *cmd, void *conf) {
  ngx_http_core_loc_conf_t *corecf;
  corecf = ngx_http_conf_get_module_loc_conf(cf, ngx_http_core_module);
  corecf->handler = handler;
  return NGX_CONF_OK;
};

static ngx_int_t handler(ngx_http_request_t *req) {
  u_char html[1024] = "<h1>This is Test Page!</h1>";
  req->headers_out.status = 200;
  int len = sizeof(html) - 1;
  req->headers_out.content_length_n = len;
  ngx_str_set(&req->headers_out.content_type, "text/html");
  ngx_http_send_header(req);

  ngx_buf_t *b;
  b = ngx_pcalloc(req->pool, sizeof(ngx_buf_t));
  ngx_chain_t out;
  out.buf = b;
  out.next = NULL;
  b->pos = html;
  b->last = html + len;
  b->memory = 1;
  b->last_buf = 1;

  return ngx_http_output_filter(req, &out);
}
```

这个文件的路径是`src/http/module/ngx_http_test_module.c`.

光是放这个是nginx的makefile是不知道的,它不会去编译新增的模块, 还需要在`auto/modules`这个文件中加入

```bash
if [ $HTTP_ACCESS = YES ]; then
    HTTP_MODULES="$HTTP_MODULES $HTTP_ACCESS_MODULE"
    HTTP_SRCS="$HTTP_SRCS $HTTP_ACCESS_SRCS"
fi
# 上面是原有的, 这里才是加上的

HTTP_MODULES="$HTTP_MODULES ngx_http_test_module"
auto是用来生成Makefile的很多shell脚本,Nginx没有用那些构建工具来制作自己的Makefile, 而是自己写了大量的shell脚本, 学习这些脚本对于自己的shell编程也是很有帮助的. nginx的编译的生成文件都在`objs`中, 清晰明了, 因此`make clean`也只是调用`rm -rf objs`即可, 非常简洁.

总之加上上面两句话, nginx就知道你要新增这个模块了, 顺序应该不是很要紧(其实我是没试过).

这样我们的模块依然不起作用, 还需要修改配置文件, nginx启动完全依靠那个`conf/nginx.conf`的配置文件!



```
server {
    listen       80;
    server_name  localhost;

    #charset koi8-r;

    #access_log  logs/host.access.log  main;

    location / {
        root   html;
        index  index.html index.htm;
    }
    location /test {
        test;
    }
```

我们在http中的server中加上`location /test`来插入我们的模块.

运行之, 在浏览器中访问`你的域名/test`就能看到`This is Test Page`几个大字, 因为是`<h1>`的嘛!

解释下这段代码吧.

```c
#include <ngx_core.h>
#include <ngx_http.h>
#include <ngx_config.h>
```

包含三个关键头文件, 这没什么异议, 注意, 我们写的模块基本都是http的.

```c
static char *set(ngx_conf_t *, ngx_command_t *, void *);
static ngx_int_t handler(ngx_http_request_t *);
```

声明两个函数, 这是两个非常重要的函数, 后面主要讲.

### command注册

```c
static ngx_command_t test_commands[] = {
  {
    ngx_string("test"),
    NGX_HTTP_LOC_CONF | NGX_CONF_NOARGS,
    set,
    NGX_HTTP_LOC_CONF_OFFSET,
    0,
    NULL
  },
  ngx_null_command
};
```

这是定义一个配置命令信息数组(为何是数组暂时还真不知道), 数组最后一个元素都是ngx_null_command.

结构体第一个参数尤为重要, 这里是test, 指的是我们在配置文件中输入test(不是路径的`/test`).这样指定后, nginx在读取配置的时候读到test命令, 才会把接管权给我们.也就是把请求转给我们去处理.

第二个参数代表我们的模块注册的是http location的命令, 并且接受0个参数.http location当然就是指这个命令触发是跟路径有关的.

> 事实上大量的模块触发都跟路径相关, 比如php, php就是接管所有后缀是`.php`的location.php的nginx模块配置如下所示

```
location ~ \.php$ {
  root /home/www;
  fastcgi_pass 127.0.0.1:3344;
  fastcgi_index index.php;
  fastcgi_param SCRIPT_FILENAME $document_root/$fastcgi_script_name;
  include fastcgi_params;
}
```

第三个参数是set, 这是一个函数指针,也就是我们一开始声明的两个关键函数中的一个,读到我们注册的`test`这个命令的时候触发的, 我们一般在set中写上托管http请求的handler函数.

第四个参数是offset类型, 用来结构体的偏移的, 不再hello模块的讨论范围, loc conf类型就直接是`NGX_HTTP_LOC_CONF_OFFSET`就行.

第五个参数就是具体offset的值, 我们这里只有一个命令, 没有参数, 输入0即可.

第六个参数NULL即可, 作用未知.

### 回调函数

写完命令注册, 我们还需要一个context,至于为何叫上下文我也不知道, 但这种把函数作为参数的方法在nodejs中一般叫回调函数,内核程序员喜欢叫它hook, 钩子函数, 我觉得也很形象.但是在hello模块中,我们可以完全不用这些回调机会.

```c
static ngx_http_module_t test_ctx = {
  NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL
};
```

nginx一共提供8个回调机会, 具体是什么时候用不在hello模块讨论范围, 这里都设置为NULL即可.

### 模块结构体

最后写上真正暴露给外面的模块结构体

```c
ngx_module_t ngx_http_test_module = {
  NGX_MODULE_V1,
  &test_ctx,
  test_commands,
  NGX_HTTP_MODULE,
  NULL, NULL, NULL, NULL, NULL, NULL, NULL,
  NGX_MODULE_V1_PADDING
};
```

其中`NGX_MODULE_V1`和`NGX_MODULE_V1_PADDING`都是宏定义, 不必去管

只需知道第二个放上回调函数数组, 第三个是注册命令, 第四个是模块类型即可.后面7个NULL.

### set函数

前面说到, set函数就是给配置信息挂上托管http请求的handler函数的.

```c
static char *set(ngx_conf_t *cf, ngx_command_t *cmd, void *conf) {
  ngx_http_core_loc_conf_t *corecf;
  corecf = ngx_http_conf_get_module_loc_conf(cf->pool, ngx_http_core_module);
  corecf->handler = handler;
  return NGX_CONF_OK;
}
```

返回类型是`char *`,它有两种返回值, 一个是`NGX_CONF_OK`也就是NULL, 另一个是`NGX_CONF_ERROR`也就是`(void *) -1`.

里面最重要的就是`corecf->handler = handler`了, 这句话把handler函数挂到corecf的handler属性上.

### handler函数

这是最重要的函数, nodejs中写http服务器, 完全只要写个handler就行了.nginx却要写前面一堆废话.

```c
static ngx_int_t handler(ngx_http_request_t *req) {
  char html[1024] = "<h1>This is Test Page</h1>";
  int len = sizeof(html) - 1;
  req->headers_out.status = 200;
  req->headers_out.content_length_n = len;
  ngx_str_set(&req->headers_out.content_type, "text/html");
  ngx_http_send_header(req);

  ngx_buf_t *b;
  b = ngx_pcalloc(req->pool, sizeof(ngx_buf_t));
  b->pos = html;
  b->last = html + len;
  b->memory = 1;
  b->last_buf = 1;
  ngx_chain_t out;
  out.buf = b;
  out.next = NULL;
  return ngx_http_output_filter(req, &out);
};
```

其实主菜才是最直观的, 前面是设置http的返回头部, 后面是设置http body.简单至极.每每写到handler部分都神清气爽, 感觉自己也会用c了..
HTTP_SRCS="$HTTP_SRCS src/http/modules/ngx_http_test_module.c"
```

补充一下. 我在用jekyll写博客的时候又出现的编译错误, 原因是使用了高亮`shell`, 不知道是不是因为没有shell, 反正它居然报错了, 修改成bash后显示正确, 这实在让我费解, 我仅仅是写一个博客而已, 你居然来个编译错误.不支持你就不管呗. jekyll用的液体模板并不能报出模板哪里错了, 总之出错了,不得不用排除法去猜, 让人郁闷.

牛逼的是我发现github貌似也使用的`redcarpet`,见github blog<https://github.com/blog/832-rolling-out-the-redcarpet>

github的[GitHub Flavored Markdown](https://help.github.com/articles/github-flavored-markdown),支持的高亮语言有这些:<https://github.com/github/linguist/blob/master/lib/linguist/languages.yml>

非常可惜的是我们没法用这么全的.
