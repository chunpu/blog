---
layout: post
title: 使用国外vps简单科学上网
date: 7:47 14/02/01
tags:
- life
---

注意：本文只为了看一些被墙的文献，18岁以下请主动关闭此页

首先是vpn翻墙，简直没有比这个更简单快速的方法了，就3步

1 - 下载并运行脚本

```sh
wget http://soft.yzs.me/pptpd.sh;sh pptpd.sh
```

2 - 按脚本提示填写ip和网卡

```sh
Which IP is your server IP:
IP:(就是你服务器IP)

Please input the netdriver of your server:
Net Driver:(你用了哪张网卡，一般都是eth0，阿里云貌似是eth1)
```

再填上用户名和密码，比如都用vpn

运行`lsof -i:1723`

看到如下结果(有pptpd和1723)就算对了

```sh
COMMAND  PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
pptpd   1614 root    6u  IPv4   9988      0t0  TCP *:1723 (LISTEN)
```

为了感谢该[脚本](http://yzs.me/1881.html)(主要是防止以后链接失效), 我决定把脚本贴进来。。

```sh
#!/bin/bash

get_char()

        {

        SAVEDSTTY=`stty -g`

        stty -echo

        stty cbreak

        dd if=/dev/tty bs=1 count=1 2> /dev/null

        stty -raw

        stty echo

        stty $SAVEDSTTY

        }

clear

ip=$(ifconfig | grep 'inet addr:'| grep -v '127.0.0.1' | cut -d: -f2 | awk '{ pr                                                                                                                        int $1}')
echo "$ip"

        echo "==========================="

serverip=""
while [ "$serverip" = "" ]; do
        echo "Which IP is your server IP:"
        read -p"IP:"  serverip
done
        echo "==========================="
        echo "Server IP:$serverip"
        echo "==========================="

        echo "==========================="

ifconfig

netdriver=""
while [ "$netdriver" = "" ]; do
        echo "Please input the netdriver of your server:"
        read -p"Net Driver:"  netdriver
done
        echo "==========================="
        echo "Net Driver:$netdriver"
        echo "==========================="

username=""
while [ "$username" = "" ]; do
        echo "Please input the username of PPTP:"
        read -p"Username:"  username
done
        echo "==========================="
        echo "PPTP Username:$username"
        echo "==========================="

password=""
while [ "$password" = "" ]; do
        echo "Please input the password of PPTP:"
        read -p"Password:"  password
done
        echo "==========================="
        echo "PPTP Password:$password"
        echo "==========================="

        echo "Press any key to continue."
        char=`get_char`
apt-get -y update
apt-get -y install pptpd
sed -i "s#\#localip 192.168.0.1#localip 192.168.0.1#g" /etc/pptpd.conf
sed -i "s#\#remoteip 192.168.0.234-238,192.168.0.245#remoteip 192.168.0.234-238,                                                                                                                        192.168.0.245#g" /etc/pptpd.conf
wget http://soft.yzs.me/pptpd-options -O /etc/ppp/pptpd-options
touch /var/log/pptpd.log
sed -i 's/#net.ipv4.ip_forward=1/net.ipv4.ip_forward=1/g' /etc/sysctl.conf
sysctl -p
iptables -t nat -A POSTROUTING -s 192.168.0.0/24 -o $netdriver -j MASQUERADE
iptables -t nat -A POSTROUTING -s 192.168.0.0/24 -o $netdriver -j SNAT --to-sour                                                                                                                        ce $serverip
echo "$username * $password *">>/etc/ppp/chap-secrets
sed -i "1i\iptables -t nat -A POSTROUTING -s 192.168.0.0/24 -o $netdriver -j MAS                                                                                                                        QUERADE;iptables -t nat -A POSTROUTING -s 192.168.0.0/24 -o $netdriver -j SNAT -                                                                                                                        -to-source $serverip" /etc/rc.local
service pptpd restart
```

3 - 在windows上添加vpn连接， 结束！

----

下面介绍普通ssh科学上网方法

ssh的方法不需要在服务器上做任何设置，因为一般vps都会默认开启sshd，我们只需要3步即可科学上网

1 - 下载putty([下载链接](http://the.earth.li/~sgtatham/putty/latest/x86/putty.exe))

2 - 编写一个脚本kexueshangwang.bat, 放在putty同路径下

```sh
putty.exe -D 1080 root@你vps的IP
```

双击执行并登陆

3 - 安装proxy SwitchySharp

[chrome插件地址](https://chrome.google.com/webstore/detail/proxy-switchysharp/dpplabbmogkhghncfbfdeeokoefdjegm)

添加一个情景

![科学上网](http://fimg.qiniudn.com/kxsw.png)

也就是在socks中填上ip: `127.0.0.1`, 端口: `1080`, 刚才-D后面的数字

启用这个情景，看看是不是能跑出去了

----

最后介绍shadowsocks科学上网的方法

shadowsocks也是目前流行所趋，他的优点，嗯 我也不大清楚，至少他不用建一个单独给人上网的linux账号，而且一般shadowsocks都会用上epoll的event库，性能应该也大大提升。也就是说shadowsocks是用来大批量科学上网的方法(卖￥)

方法同样简单，不过此处只介绍nodejs方法，谁让nodejs是最牛x的平台呢，同样是3步

1 - 分别在客户端和vps端安装nodejs

window直接在[nodejs官网](http://nodejs.org/)下载即可

ubuntu更容易`apt-get install nodejs`

2 - 在客户端和vps端分别安装shadowsocks的全局程序

```sh
npm install -g shadowsocks
```

此命令是增加两个全局命令(在命令行可以直接用的命令)，一个是ssserver，另一个是sslocal

我们需要修改服务端和vps端的配置文件，其地址在全局安装的时候会显示出来

修改服务端的`config.json`, 就是把config.json中的ip从`127.0.0.1`改成`0.0.0.0`

在客户端我们也需要修改配置文件`config.json`, 把ip从`127.0.0.1`改成你的vps外网ip

3 - 启动

在服务端直接执行`ssserver`即可，看到如下即成功

```sh
1 Feb 03:51:23 - UDP server listening 0.0.0.0:8388
1 Feb 03:51:23 - server listening at 0.0.0.0:8388
```

在客户端执行`sslocal`

然后做端口代理转发即可，具体步骤同方法2普通ssh上网步骤3 
