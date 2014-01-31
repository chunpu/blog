---
layout: post
title: 做论坛之心不死
data: 9:28 2013/12/22
tags:
- nodejs
- javascript
---

很早前就想做论坛, 不幸的是搁浅了, 现在想来幸好是搁浅了, 当时的技术实在是渣, 正所谓, 士别三日, 当刮目相看, 我现在看我自己一个月前的代码就是一坨大翔

总有人没怎么用nodejs就说nodejs没法应付复杂逻辑, 没法用数据库, 但在我看来, 只有node才能真正发挥数据库的速度, 原因也很简单, 它是异步IO. es6的出现使得异步函数不再是噩梦, 让人想想都兴奋, 使得我周日早早来到办公室, 实践论坛制作

论坛和博客, 这是两个完全不同重量级的东西, 论坛大概比博客复杂一个维度, 做一个博客大概需要100行, 做一个论坛, 预计要1000行左右, 我建议是没做一个项目前都要预计它的复杂度, 代码量, 不然很难有走到终点的力气

这次论坛将参照[小米论坛](http://bbs.xiaomi.cn/)来做(真是小米虐我千百遍, 我待小米如初恋啊), 虽然小米把我拒了, 但论坛绝对是做的一级棒, 特别是刚出来的bbs v5

选小米论坛的另一个原因是它非常的复杂, 在首页上每一条都有

- 浏览量
- 回复量
- 最后回复
- 版块名

看到这里我虚了一口气, 也不是很复杂嘛, 比起新浪微博, 复杂度低太多了, 因为新浪微博还要知道每条微博的点赞数, 评论数, 转发数, 我是否收藏, 我是否点赞, 特别是最后两个功能, 想想都复杂

所幸小米的收藏, 最后谁回复 等都是在点进去才出现的, 不然复杂度又提升一个档次

由于偶完全不懂数据库, 就随意设计数据库结构了..

post的schema

- 标题: title: 
- 内容: content
- 作者id: userid
- 板块id: sectionid
- 发表时间: createtime (不想用created_at)
- 修改时间: updatetime
- 最后回复时间: lastreplytime
- 阅读数: readcount
- 回复数: replycount
- 收藏数: collectcount
- 点赞数: upcount
- 是否加精: isdigest
- 是否置顶: istop
- 是否官方: isofficial
- 是否推荐: isrecommend

最后那几个是否还能像小米那样有数字来表示权重, 这样如果都是置顶也能分出次序来了.

comment的schema

- postid or commentid
- userid
- username
- content


mysql学习
---

还是先学习一下mysql吧...

0. 登陆mysql

```shell
mysql -uroot -p
输入密码
```

1. 首先要选择databse

```shell
create database testdb;
show databases; # 显示全部
use testdb;
show status; # 显示当前状态, 可以看到是否切换数据库了
```

2. 建一个表

```shell
# 如post table
create table post_table(id bigint primary key, title varchar(20), text content, userid bigint, sectionid bigint, createtime datetime, updatetime datetime, lastreplytime datetime);
```

3. 插入数据

```shell
insert into post_table values(1234, 'title', 'this is content', 4321, 2345, now, now, now);
```

4. 查询数据(如第二页: 第20-40条)

```shell
select * from post_table order by lastreplytime desc limit 20, 20;
```

5. 关联查询(查20条而且还要通过userid查到username)

```shell
select * from user_table, post_table where users.id=posts.userid;
```

数据库到这儿就结束啦
