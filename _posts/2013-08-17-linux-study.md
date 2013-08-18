---
layout: post
title: Linux学习-1
---

一直不会Linux，有些遗憾，不过我觉得现在最可以胜任的工作可能是运维了。

首先是因为我目前的工作虽叫研发但每天代码量不到50.大量的修复前人的烂摊子。

其次我们经常看集群，网络等信息，反而很接近运维。再加上我本人对各种服务器很熟悉，也算了解各种协议，而且又会shell，perl。活脱脱一个运维嘛。。

运维怎么可能不懂Linux，你可能要问了。原因是我根本不用Linux系统，我是windows的粉丝，一直觉得Linux非常反人类。

我使用Linux的方式全部为在windows中用putty远程登录，因此Linux的图形界面很少接触。

Linux之所以实用性远不如windows，归根到底是Linux的cli太强大了。这让大多数人沉迷在命令行界面无法自拔，不愿去接触图形界面。


这次又入门了一下Linux,我发现不少之前的误区

### shell

首先是shell，什么是shell，shell就是系统的壳，用来和系统交互的。我以前的误区在于认为命令行就是shell。现在看来大错特错了，GNOME同样有shell，GNOME shell就是我们看到的图形界面，这个shel用openGL渲染，它也可以编辑文件，修改配置等。当然bash也是一个shell。

因此shell分两类，一种是GUI(Graphical User Interface)的shell，图形shell，比如GNOME,KDE.

另一种是CLI(Command Line Interafce),命令行shell，有bash，zsh等。

之所以会有这个误区,是因为我们把shell当成了终端.

在很早以前,终端是一个类似打字机的东西,而现在我们在OS X或者Ubuntu上打开命令行仍然叫terminal,是因为现在的terminal是虚拟terminal.

其实终端是包括这两种shell的.终端可以理解为一个设备,`/dev/tty`

```bash
echo "hello" > /dev/tty
```

直接就打印到屏幕了.tty同样也是个命令

man一下可以看到这个:`tty - print the file name of the terminal connected to standard input`

tty会输出目前连接标准输入(键盘)的终端文件.(Linux一切都是文件..)

在一般的Linux发行版中,可以通过ctrl+alt+F1----F7来切换终端.对应tty1-tty6命令模式,tty7对应图形界面.

普通用户和root用户每次启动终端都会自动启动一个shell.这个在`/etc/passwd`中可以看到.

比如root一般就是对应`/bin/bash`这个shell.如果是zsh粉丝的话可能就是`/bin/zsh`

而其他系统用户的shell都是`/bin/false`,也就是没有对应shell.至于为什么没有GNOME这种,可能是因为图形界面也是先进入bash,再启动start X的吧.

### 分区

第二个误区在于分区,在较早接触Linux的时候,碰到分区问题,前辈会告诉你一堆英文和数字,什么`/home`,`swap`,根目录,等等.当时就吓坏了,想想还是跑回去玩玩windows吧,毕竟只要分个C盘D盘就可以了.

知道一个月前我装centos,依然是分四个区才敢装系统.现在看来,唯一需要分区的是根目录,根目录就好比C盘,系统塞里面呢.

不过在分根目录`/`之前,我们一般会先分swap分区,也就是内存不够用硬盘来替代的分区,虚拟内存分区.

有种说法是swap分区是实际内存的两倍,这在以前内存只有几M的时候是对的.但是现在内存最少4g,动辄几十G的环境下,swap早就已经没用了.

我的内存是8g,难道我要分16g给swap嘛?我的固态硬盘才128g.可惜的是我并不知道swap分区不分会怎样.按我看来swap分区应该分1g.

接下来所有分区都给`/`根分区.

> 那些`/home`,`/boot`怎么办呀! 

这问题就跟windows分区分5个盘一样笨,有些朋友硬盘才300多g,却分了5,6个区.连一个30g的dota2或者30g的wow都不知道放哪里.

windows当然就一个C盘就行了,Linux更加是,`/home`和`/boot`不就是在根目录下嘛,干嘛要分开来.我只能说纯属蛋疼.毫无必要.

我们可以通过`mount`命令来查看挂在的分区.

如果只分一个根目录区的话,就会有一行`/dev/sda1 on /`.


### 硬盘和分区

上面的sda1是什么?看完本段就没有这个疑问了.

在Linux中,设备都被显示在`/dev`目录下,硬盘也不例外.

硬盘的名字分两种,一种是`sd`+字母,如`sda, sdb`.

另一种是`hd`+字母,如`hda,hdb`.

hd是n年前IDE接口的硬盘,口很大针很多.现在几乎看不到了.

sd是现在的硬盘,比如sata串口硬盘.sd可以理解为small disk.

因此现在都是sda,sdb.非常好理解,sdb就是双硬盘的时候第二块硬盘

这些文件在硬盘插上去就会显示,即便他们还没被格式化,还没被分区.

分区是使用fdisk命令(只能MBR),我们分出来的区就成为sda1, sda2这样的.

一般的话我们只有一块硬盘, 一个根分区,因此就只显示`/dev/sda1 on /`


### 文件权限

Linux有着完善的文件权限管理.

一般使用`chmod`来进行权限管理

文件权限可以分别对user grounp other三个分类进行设置

习惯上user和group一样,因为group就是user同名的.暂时看来很少会用到group.

权限有三个,read, write, 和execute.

在`ls -l`中我们可以看到文件的权限信息`drwxr-xr-x`.

分开看就是`d rwx r-x r-x`.第一个d是指目录,如果是l就是链接,如果是`-`就是普通文件.

同样第一个`rwx`表示user对文件有读,写,执行,全部三个权限.

第二个`r-x`表示表示group对文件有读和执行两个权限,第三个同理是对other说的.

```bash
chmod a+x # 所有用户加上可执行权限
chmod u+wx # user用户加上写和执行权限
```

我们可以这样给文件加上权限, 一般我们写脚本,如果想直接运行,都会用`chmod +x xxx`来使其变成可执行文件.

注意这儿的a表示all user.即包含了user+group+other.如果不写就是默认为a.


#### 数字模式

我们还看到别人输入`chmod 777 xxx`,那这个数字代表什么呢.

其实这个数字是8进制表示法.

```
r = 4 (2^2)
w = 2 (2^1)
x = 1 (2^0)

rw = 4+2 = 6 = 110
rwx = 4+2+1 = 7 = 111
r-x = 4+1 = 5 = 101
```

常用mod的有可执行文件775 普通文件660 文件最高也是666

我们可以通过umask命令查看默认权限配置

umask root 0022  普通是0002

root限制了group权限,新建的时候是减法计算.

还有另一个模式叫s, g, t，(suid guid sticky)

比如sticky,

```
root@ft:~# ll /usr/bin/passwd
-rwsr-xr-x 1 root root 42824 Apr  9  2012 /usr/bin/passwd*
```

首先要知道用户执行的进程拥有和用户一样的权限.

passwd命令所有用户都可执行,但我们知道passwd本质是修改`/etc/shadow`文件,这个是一般用户无法查看的.那怎么办呢?

这时候就需要suid了,passwd不管是什么用户执行,都会以所属用户即root执行,这样该进程就拥有查看并修改`/etc/shadow`的权限了.

sticky是别人无法删除不是自己的东西.guid当然和suid一样啦.

### Linux根目录下的目录

Linux下的目录都有些是成对的.

比如mnt media

他们都是空的,用来挂在新的设备,一般是存储设备.

media用来挂在自动播放的存储设备.

还有bin和sbin

他们都是可执行文件的目录,sbin表示`super binary`.里面放的都是危险的可执行程序,都需要root权限,比如fdisk.

最后是usr和opt

usr是放用户自己安装的程序的,而opt也是放自己的程序的,但opt一般习惯放大文件.

### `/proc`

`/proc`是虚拟文件系统,是一部分内存的映射,这个是Linux提供的接口,在UNIX比如苹果`OS X`上并没有这个.

`/proc`的接口是Linux比其他系统简单的原因,我们可以轻松获取硬件信息,比如常用的`/proc/cpuinfo`, `/proc/meminfo`

还有电池信息, 鼠标信息, 我们仅仅修改文件就可以控制鼠标和键盘,这是多么爽啊.

因此,在Linux中查看实时的硬件信息,比如网卡状况啊,尽量使用`/proc`查看,显得更加专业.

### `/etc`

和查看信息用`/proc`一样,在Linux中修改配置,我们尽量的使用`/etc`直接修改文件,这样更加方便我们了解本质,比如新增一个用户,你需要纠结到底是`useradd`还是`adduser`,但实际上新增一个用户就是在`/etc/passwd`中加一行而已.

同样,修改ip,我们需要纠结很多网络命令,但其本质也是修改`ifcg-ethx`这种文件.多用`/etc/`修改配置,我只能说分分钟涨姿势.




乱七八糟的小知识
------

### `cal`

cal命令可以显示可爱的日历

```
    August 2013
Su Mo Tu We Th Fr Sa
             1  2  3
 4  5  6  7  8  9 10
11 12 13 14 15 16 17
18 19 20 21 22 23 24
25 26 27 28 29 30 31

```

### Linux很多rc文件的rc啥意思

rc = run command


### 跟踪日志信息

tail --follow 用来实时跟踪查看日志信息


### 快速搜索文件

locate命令,通过文件数据库搜索,文件数据库一天更新一次,可以通过`updatedb`命令手动更新

### 其他小知识

分区跟硬件无关，是软件实现， 主流有MBR和GPT

### 小技巧

`esc键 .` 可以复制上一个参数

`find . -name "a*" -exec ls -l {} \;`管道参数.




网络对应配置文件
----

网卡配置  /etc/sysconfig/network-scripts/ipcfg-eth0

dns /etc/resolv.conf

主机名 /etc/sysconfig/network

静态主机名 /etc/hosts

可以用`mtr`来测试网络质量

mtr = my traceroute. 可以看到丢包率

### 使用文本工具

cut 

cut -d: -f1 /etc/passwd

sed 正则替换

### Linux 启动顺序

BIOS  可启用设备 

扇区前512字节最后两个字符是不是55aa BIOS移交控制权给MBR前446字节

446字节一般用来跳转一个复杂的引导程序， 比如grub

启动顺序 grub中的stage1也就是MBR，然后stage2 最后是内核

加载完linux内核后运行第一个进程 init

我们可以看到init是pid为1的进程, init是所有进程的父进程

他的作用是启动/etc/rc.d/rc.sysinit

init 有0-6 7个运行级别，在/etc/tabinit中可以看到注释，

同时这个运行级别就配置在这个文件中

### shell中的`$`和`#`

`$`表示普通用户user

`#`表示超级管理员root

shell前面每次那一段叫提示符,可以在shell的配置文件中修改.

### root忘记密码

grub是给kernel加上1，可以单用户模式，直接进root不要密码

可以用grub-md5-crypt给grub加密码.

### yum

yum是rpm的前端程序 全程叫yellowdog updater modified 可以解决依赖关系

查询一个文件是哪个程序的

```
yum whatprovides /etc/tabinit
```

centos中所有几乎所有软件都是rpm包装的


待了解
------

Linux加密算法`/etc/shadow`



语义化
---

这是我学习Linux最大的感悟,如果有人问我怎么学Linux,我会告诉他三个字`语义化`.

曾见过前辈教新人Linux, 嘴里飞快的吐着各种命令,比如gdb, 一会儿s,一会儿f,一会儿b.新人呆呆的照着做.

前辈说完,问道会了吧,新人点点头,心里不住咒骂,这sb Linux,调试都这么烦, 还来个vim,能跟我大eclipse比?

其实这就是因为学习Linux没有语义化学习的原因.

比如一个简单的`tar zxvf xxx.tar.gz`不少人敲了十遍还是记不住,就算记住了解压缩释放, 压缩的时候还是得去百度一下.

我是非常反对简写的.

```
tar --gzip --extract --verbose --file  nginx-1.5.2.tar.gz
```

一般cli程序都会用`--`来作为参数的开头,并且加入`-`开头的alias,也就是简便的别称

比如版本是`--version`,帮助是`--help`.一般的别称是`-v`和`-h`

虽然有简写,比如`-h`,但版本就不一定了,因为`-v`可能是`--version`也可能是`--verbose`.所以有些`--version`的缩写是`-V`.

`-h`也未必是`--help`,也可能是`--human-readable`

因此使用全称更加方便我们语义化的记住用法.

比如gdb,如果我们每次都输入step, refresh, finish这些全称的话,估计10分钟就用熟了.

同样vim, 非常不推荐在配置中写上`set ts=2`,谁能想到ts是tabstop呢.没准自己也有一天忘了呢.

之所以提语义化是因为我用这么久命令行,依然不记得如何ls用时间或者大小排序.

其问题就在于`ls -s`根本不是按大小排序,而是另一个功能.

如果我早点知道`ls --sort=time`和`ls --sort=size`,我面试还会煞笔到连ls都说错! 哎,一提到曾经的面试我只能说都是泪.

语义化实在太重要了.

现在的前端招聘,对html的要求一般就一条:

> 熟悉语义化html

这是基本的一条,同时也是要求很高一条.

直接涉及的html标签的使用,嵌套,控件的设计, 以及CSS class命名, 行为和显示分离.一大堆知识.

最后,我只能说,语义化学习Linux,轻松加愉快


