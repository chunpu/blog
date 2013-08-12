---
title: nginx模块开发之handler函数
layout: post
tags:
  - nginx
---

http handler是http模块中最重要的函数, 直接托管http请求.

和前文set不同, set是在nginx启动的时候读取配置的过程中被触发的, 而handler函数是在真实请求到那个路径上时被触发的.

也就是浏览器请求多少次, handler就触发多少次.

### 返回值

handler的返回类型是`ngx_int_t`, 因为一般的http handler定义好body后就能交给http filter函数了, 比如我们hello模块的`ngx_http_output_filter`, 而filter函数都是返回整形数的.比如error就是-1.

### 参数

handler参数只有一个, `ngx_http_request_t *req`, 这个简单的出奇, 不过我用过不少http服务器框架, 也就只有nodejs分了`request, response`两个参数.

req是一个巨大的结构体.

不过这样的后果就是response的信息都写在request这个结构体上, 比如头部信息就是`request.headers_out`中.

请求的头部在`headers_in`中.

我们可以清楚的看到各种请求信息, 比如user_agent:

```
(gdb) p *req->headers_in.user_agent
p *req->headers_in.user_agent
$3 = {hash = 3194399592611459, key = {len = 10, data = 0x6eaa37 "User-Agent"}, v
   data = 0x6eaa43 "Mozilla/5.0 (Windows NT 6.2; WOW64) AppleWebKit/537.36 (KHT
lowcase_key = 0x6ebc69 "user-agentaccept-encodingaccept-languagecookie"}
```
在print`headers_in`的时候我们还发现nginx已经通过user_agent帮你分辨好了浏览器..还专门列出了msie6.. 当然还有很多重要的头部信息, 比如`if_modified_since`, `chunked`, `cookie`等.

在req中也有全部的我们需要的信息, 有`method`但不是保存字符串, 使用数字代表的, 比如get就是2. 还有`http_version`, 比如http1.1就是1001.

但是想uri, request_line, method_name, 都是显示一个request, line, 并没有做解析, 直接是生肉`GET /test HTTP/1.1\r\nHost`

但拥有全部http请求信息的我们也完全可以写出所有http相关的模块了.

### 源代码中是如何触发handler的

第一次使用gdb调试的时候发现调试不了, 马上就想起来nginx是分master和worker进程的, 在调试的时候, 我们不仅需要关掉daemmon后台运行, 以及master进程, 仅仅使用一个worker进程进行调试.

方法是修改nginx配置文件`conf/nginx.conf`,

```
daemon off;
master_process off;
```

这样就可以用gdb调试了.

handler函数在`src/http/ngx_http_core_module.c`中被调用.可以看到我们的handler函数被当成request结构体的`content_handler`属性.

返回`ngx_http_output_filter(req, &out)`的时候, 一次完整的请求+返回就完成了.
