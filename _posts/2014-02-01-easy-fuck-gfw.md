---
layout: post
title: 使用国外vps简易翻墙
date: 7:47 14/02/01
tags:
- life
---

首先是vpn翻墙，简直没有比这个更简单快速的方法了，就3步

### 1. 下载并运行脚本

```bash
wget http://soft.yzs.me/pptpd.sh;sh pptpd.sh
```

### 2. 按脚本提示填写ip和网卡

```bash
Which IP is your server IP:
IP:(就是你服务器IP)

Please input the netdriver of your server:
Net Driver:(你用了哪张网卡，一般都是eth0，阿里云貌似是eth1)
```

再填上用户名和密码，比如都用vpn

运行`lsof -i:1723`

看到如下结果(有pptpd)就算对了

```bash
COMMAND  PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
pptpd   1614 root    6u  IPv4   9988      0t0  TCP *:1723 (LISTEN)
```

### 3. 在windows上添加vpn连接， 结束！

----

下面介绍shadowsocks翻*墙的方法

