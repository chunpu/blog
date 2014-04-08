---
layout: post
title: gentoo安装笔记
tags:
- linux
---

选择ISO
---

gentoo的安装分为`stage1`, `stage2`, `stage3`, 这里有他们的[区别解释](http://man.ddvip.com/linux/gentoo/install_man/hb_part1_chap2.html).
大致就是stage1最原始最烦, stage3最脑残. 像我这样的Linux白痴自然选`stage3`

先是选择iso文件, 需要参考这里的[官方文档](https://www.gentoo.org/doc/zh_cn/handbook/handbook-x86.xml?part=1&chap=2)

由于阿里做了开源的镜像和源<http://mirrors.aliyun.com/>(比较信赖大公司维护的源), 找到`/gentoo/releases/amd64/autobuilds/current-iso/`, 注意里面还有hardened版本的, 那是gentoo的专注安全的版本, 不用管它, and64指64位, 不要选x86(32位), 不然还要去了解啥叫`i486`, `i686`, 其实那都是过时货..

下载里面的`.iso`文件(好吧, 其实我不知道怎么在windows下解压缩`.bz`)..


启动ISO
---

启动时选择kernel, 会有如下几个选择

- gentoo
- gentoo-nofb
- memtest86

其中`gentoo-nofb`就是禁用[帧缓冲](http://blog.csdn.net/sdvch/article/details/5941067), 由于我还是不懂啥意思, 就直接输入`gentoo`吧!

进去发现一会儿就能进入熟悉的linux terminal界面了, 貌似连网络都不用设置.. 可以直接ping百度, 还能用鼠标


装到硬盘
---

前面我们只是打开了live cd, 终端前面那个啥还是显示的live, 现在要把gentoo装到硬盘里, 一般硬盘在linux就是`/dev/sda`, 不懂的可以去看一下hda, sda, sdb啥的

### 分区

分区就是`fdisk /dev/sda`, 不过我很懒, 直接分一个区先

### 文件系统

查看自己的文件系统是使用df命令`df -T -h`, 一开始我们看到一大堆根本不知道干啥的分区, 还是先把硬盘格式化成ext4再说吧

```bash
# mkfs.ext4 /dev/sda7
```

格式化之后用df还是看不到这个分区的, 我们需要挂载它

```bash
# mount /dev/sda1 /mnt/gentoo
# df -T -h
```

### 下载stage

这是个很神奇的问题, 我一开始下载的那个不是stage嘛? 反正很神奇就是了, 可能我一开始就搞错了, 既然官方教程说要下那就下吧, 下完解压缩之

```sh
# cd /mnt/gentoo
# tar xvjpf stage3-*.tar.bz2
```

解压缩的时候我突然醒悟原来解压缩就是装系统啊, 因此也要注意由于tar是相对路径, 因此必须要在`/mnt/gentoo`下, 也就是`/dev/sda1`.
解压缩完看到下面各种`usr bin etc`, 俨然一副系统的样子

### 安装Portage

portage目测就是yum和apt-get之流, 我们还是用阿里云的源

```bash
# wget http://mirrors.aliyun.com/gentoo/snapshots/portage-latest.tar.bz2
```

下下来一看, 哇塞! 67M

### 编译选项

新建一个文件`etc/make.conf`(注意是/mnt/gentoo下的)

```sh
CFLAGS="-O2 -pipe"
MAKEOPTS="-j8"
```

要注意, 这个makeopts, 如果不写很可能意味着你的吃零食时间要翻倍, `-j8`表示8倍速同时编译(实际不到), 当然这和你的cpu能力有关

### Chroot

#### 选择镜像host

gentoo很人性化的提供了mirrorselect的小工具, 我们这里直接选择163的就行了, 自己输入阿里云也行(空格是选中, 回车提交)

```sh
# mirrorselect -i -o >> /mnt/gentoo/etc/make.conf
在选一下rsync(不知道是啥玩意..), 我随便选了个中国的
# mirrorselect -i -r -o >> /mnt/gentoo/etc/make.conf
```

#### 挂载/proc和/dev文件系统

不了解具体是啥意思..

```sh
# mount -t proc none /mnt/gentoo/proc
# mount -o bind /dev /mnt/gentoo/dev
```

#### 真正的chroot

```sh
chroot /mnt/gentoo /bin/bash
# env-update
>> Regenerating /etc/ld.so.cache...
# source /etc/profile
# export PS1="(chroot) $PS1"
```

最后一句可以用来改前缀, 以前还真不知道..
