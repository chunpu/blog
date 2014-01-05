---
layout: post
title: mysql折腾笔记
date: 10:58 2014/1/5
tags:
- mysql
---

经常吹嘘自己玩过各种数据库. redis, mysql, sqlite, mongodb..常用数据库都不在话下,不料今天却在远程连接mysql上栽了跟头,于是记一下这个教训

mysql默认是不开启远程连接的,要让mysql支持远程连接需要两个步骤

### 第一步

首先是打开`/etc/mysql/my.cnf`, 如果发现自己的mysql配置文件不是这个路径,强烈建议重装mysql(我就是因为装的mysql和别人不一样才鼓捣很久), 重装失败可以后面详解

将`bind-address = 127.0.0.1`那一行配置改为`bind-address =  0.0.0.0`

bind-address表示监听时绑定的IP地址,为什么要这么改呢?

因为网卡收到请求,会把每个socket的目标地址和端口解析出来,并对照进程监听的IP和端口,如果IP和端口吻合,就传给进程

端口我们都知道mysql是3306,错不了. 而IP默认是127.0.0.1,是一个环回地址,只有本机请求本机,目标地址才会是127.0.0.1,因此我们需要将绑定IP改为网卡地址, 也就是外网IP, 因为远程连接数据库目标IP肯定是公网IP. 不过还可以改成`0.0.0.0`,这样就表示监听所有IP, 只要端口一样, 网卡都会把请求传给进程


### 第二步

进入mysql

`mysql -uroot -p`, 输入密码

添加一个可以远程访问的用户

```mysql
GRANT ALL PRIVILEGES ON *.* TO  'USERNAME'@'IP'  IDENTIFIED  BY  'PASSWORD';
```

注意此处有三个变量, IP就是指远程client的ip地址,如果希望所有IP都可以访问, 就把`IP`改为`%`

重启mysql, `service mysql restart`

然后就可以通过navicat, 或者`mysql --host ip地址 -uUSERNAME -p`来访问了

如果你发现依然不能访问,首先确定是否打开了iptables(防火墙), (随便做个网页,用外网访问一下如果可以就不是iptables的问题)

这个时候就需要重装mysql了

千万不要以为精通mysql的安装(`apt-get install mysql-server`)就同样精通mysql的卸载了(`apt-get remove mysql-server`), (yum党退散

如果出现卸载不完全, 还出现了如下提示

> Unable to set password for the MySQL “root” user  An error occurred while setting the password for the MySQL administrative user. This may have happened because the account already has a password, or because of a communication problem with the MySQL server.

可以进行如下步骤:

```sh
dpkg --get-selections | grep mysql

libdbd-mysql-perl                               install
libmysqlclient18                                install
mysql-client-5.5                                install
mysql-client-core-5.5                           install
mysql-common                                    install
mysql-server                                    install
mysql-server-5.5                                install
mysql-server-core-5.5                           install
```

把这些都删了

```sh
apt-get remove libdbd-mysql-perl libmysqlclient18 mysql-client-5.5 ... --purge
```

收尾清理垃圾

```sh
apt-get autoremove
apt-get autoclean
rm /etc/mysql/ -fr
rm /var/lib/mysql/ -fr
```

然后再执行

```sh
apt-get install mysql-server
```

就成功啦







