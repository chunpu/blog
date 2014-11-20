---
layout: post
title: autoproxy rule
tags:
- proxy
---

### `example.com`

匹配：http://www.example.com/foo
匹配：http://www.google.com/search?q=www.example.com
不匹配：https://www.example.com/
用于表明字符串 example.com 为 URL 关键词。任何含关键词的 http 连接（不包括 https）皆会使用代理。

### `||example.com`

匹配：http://example.com/foo
匹配：https://subdomain.example.com/bar
不匹配：http://www.google.com/search?q=example.com
匹配整个域名（含子域名）（不论是 http 还是 https），一般用于该站点的 IP 被封锁的情况。

### `@@||example.com`

这种规则的优先级最高，表示所有匹配 ||example.com 规则的网址均 禁止 代理。一般用于特殊情况，比如禁止对国内的网站误用代理。

###


### `!`

注释

未完成
