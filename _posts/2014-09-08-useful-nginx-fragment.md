---
layout: post
title: 常用的 nginx.conf 片段
tags:
- nginx
- server
---

记录一些常用的 nginx.conf 片段, 虽然最好的文档始终是 <http://nginx.org/en/docs/>

#### 几个基本点

nginx 的配置层级, `http > server > location`

nginx 的一条配置格式为 `directive value1 value2;`, 如 `listen 80;`, listen 是 directive, 80 是 参数

directive 可以理解为函数, value1 value2 就是他的参数, 可以看出 nginx 的配置都是按空白分隔的, 我个人喜欢用单个空格, 你也可以用 tab, 句末有分号, directive 有其适用的范围, 英文称 context, 比如有些 directive 可以放在 server 下, 有些却只能放在 location 下

nginx 的文件根目录默认是 nginx 安装目录, 也就是说一般是 '/usr/local/nginx', 因此在填写相对路径的时候, 要记住不是 nginx.conf 的相对路径, 比如

```nginx
access_log logs/myapp-access.log;
```

### 按 hostname 分 server, 并进行代理转发

```nginx
upstream proxy_app {
    server 127.0.0.1:8888;
}

server {
    server_name myapp.com;
    access_log logs/myapp-access.log;
    location / {
        proxy_pass http://proxy_app;
    }
}
```

`server_name` 这是极常用的例子, 比如用 nodejs 或者 go 开了一个服务器, 使用的是 8888 端口, 同时又注册了 myapp.com 作为其域名, 那最基本的配置就是开启一个指向该 server 的 upstream, 同时在匹配的域名为 myapp.com 向其转发, 域名匹配值得是 nginx 先解析 http 头部, 如果 host 值为 myapp.com 则判定这个请求属于这个 server, `server_name` 具体的文档说明见<http://nginx.org/en/docs/http/ngx_http_core_module.html#server_name>

location 是用来匹配路径的, `location /` 是指匹配一切路径, 因为默认匹配是指**包含**, `location /` 就是说包含一切带有 `/` 的路径, 而所有的路径都是以 `/` 的开头的, 如果想要只匹配 `/` 路径, 则应该使用 `location = / {}`, 关于 location 匹配更多的可以看官方文档<http://nginx.org/en/docs/http/ngx_http_core_module.html#location>

这个例子中还有一个 directive: `access_log logs/myapp-access.log;` 这一条也是非常的常用, 如果不写的话, 访问日志 nginx 会默认记录在 `logs/access.log`, 而错误日志是记录在 `logs/error.log` 中的, 但是这个server 是专门给 myapp.com 用的, 如果没有和其他访问日志分开, 日志中又没有存 `$hostname` 这样的信息, 如果 server 中出了问题, 我们是很难通过日志分析出来的, 因此, 专门给他一个日志是非常非常重要的, 如果你的 qps 很高的话, 还建议用 logrotate 对这些日志进行按日切割和管理, 如果你的服务器已经有了专门的日志管理, 可以使用 `access_log off;` 来关闭日志

要注意的是, nginx 默认的 `proxy_pass` 用的是 http1.0, 因此你不加 connection 头的话默认是没有长连接的, 如果负载压力大, 这会给后端的服务器造成很大的压力, 往往能看到几千个 `time_wait` 那样的 tcp 连接, 如果想要转发至服务器用长连接, 常用的做法是

```nginx
http {
    proxy_http_version 1.1;
    server {
        server_name myapp.com;
        location / {
            proxy_pass http://myapp.com;
            proxy_set_header Connection "";
        }
    }
}
```

上面多的两个 `proxy_http_version` 显然就是把默认的 1.0 改成 1.1, 可后面的`proxy_set_header`又是什么意思呢? 而且为什么一个在 http 中, 一个在 location 中, 在一起不行吗?

> `proxy_http_version` 写在 http 外面是让所有的 proxy 转发都变成 `http1.1`, 而由于每次到 location 中, Connection 这个头都会被写成 `close`, 因此, 我们不得不在 location 中再次把这个头清除, 清除的方法就是设为空字符串

### 静态文件

比上面用的更加广泛的 nginx 用法是 静态文件服务器, 这也是 nginx 当初的一大卖点

```nginx
location / {
    root html;
    index index.html index.htm;
    autoindex on;
}
```

这三行的意思分别是

- root html: 以 nginx 下的 html 文件夹作为跟目录, html 里的文件是开放的, 可以被访问到的, 而 html 外面的则不可以, 即便是 `../..` 这样也无法被访问到

- index index.html: 指的是如果访问路径, 那就访问尝试匹配 index directive 后面的文件

- autoindex on: 指的是开启目录显示, 在 nginx 是对外的时候最好把它关掉

### 防 DDOS

我们这说的并不是那种 syn flood 之类的 ddos(这种 linux 自己就挡住了), 而是指正常的, 健康的但是又过量的 http 请求

```nginx
http {
    limit_conn_zone $binary_remote_addr zone=conn_limit_per_ip:50m;
    limit_req_zone $binary_remote_addr zone=req_limit_per_ip:50m rate=50r/s;
    server {
        limit_conn conn_limit_per_ip 50;
        limit_req zone=req_limit_per_ip burst=500;
    }
}
```

`limit_conn` 和 `limit_conn_zone` 对应的是 <http://nginx.org/en/docs/http/ngx_http_limit_conn_module.html> 模块
`limit_req` 和 `limit_req_zone` 对应的是 <http://nginx.org/en/docs/http/ngx_http_limit_req_module.html> 模块

http 下面的两个 zone directive 可以理解为声明了这两个字典, 其中一个叫 `conn_limit_per_ip`, 另一个叫 `req_limit_per_ip`, 他们都是以 `$binary_remote_addr` 作为键值记数, 冒号后面的 `50m` 表示占用内存的大小, nginx 配置中的内存都是 shared 内存, 也就是说 n 个 worker 共享的

server 下面的 `limit_conn conn_limit_per_ip 50;` 表示上面定义的字典, 这个 server 记数不能超过 50, 这是啥意思呢? 我们用 remote_addr 为键值, 就是保存了所有的 ip, 一个 ip 对应的值不超过 50, 这就防止了单个地址使用暴力不断请求你的服务器至瘫痪的隐患, 当然你说有人用几十台服务器攻击你咋办, 那就没办法了, 报警吧少年!

理解了 `limit_conn`, 理解 `limit_req` 并不难, `limit_conn` 可以认为是一直开着的长连接, 但是有人一直用短连接请求骚扰你咋办, 这时候就要用上 `limit_req`

今天的 nginx 小片段就到这里, 为什么最近都是这些水文, 唉, 只能说业务繁忙啊, 说技术的怕是跟不上节奏了, 只好说说日常生活中的小经验, 下次会说最佳 `log_format` 实践, 怎么记录日志可以让你用 awk 和 perl 排查日志更方便
