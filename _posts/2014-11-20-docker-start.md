---
layout: post
title: docker 入门教程指南
tags:
- docker
---

关于 docker
---

今天云平台的同事提到, 现在的运维就是恶性循环, 因为大家都在申请机器, 显然的是, 大家申请机器都是按照自己的峰值申请的, 而为了保证突发状况, 如 ddos, 双11 等, 申请者甚至会两倍于自己的峰值去估算自己的服务. 由于资源紧张, 云平台会对其削减, 因为云平台也要为公司减少运维成本, 可能原来申请10台机器最后变成给5台, 而最终有经验的申请者会直接申请20台, 然后等削减成10台(正如鲁迅所说, 如果不说把屋顶掀了, 中国人根本不会同意在屋里装个窗户)

申请容易而回收难, 云平台并没有能力去监控每台机器的使用量, 就算他们找到使用资源大大浪费的机器, 他们也难以说服拥有机器的团队回收其机器, 最终, 大量超高配的机器(比如64g 内存, 16核), 就贡献着几百的 qps, 造成极大的浪费

也许自动化运维是每个云平台的目标, 将运维自动化, 私有化, 最后公有化, 而当今大热 docker 就是解决运维困境的一剂良药

什么是 docker? 为了更好理解, 你可以直接和别人说它是虚拟机, 实际上 docker 并不是虚拟机, 它做的是 linux 的隔离, 但它的隔离做的如此之真实以至于让人觉得自己拥有可以一台完整的 linux 系统

那 docker 和 虚拟机具体是什么区别呢, 虚拟机在底层模拟出各种硬件, cpu, 硬盘之类的, 而 docker 是在软件层面给资源分组, docker 性能无限接近原生, 因为 docker 用的就是系统自己的进程, 而虚拟机做的再好, 也做不出原生的感觉

docker 的隔离技术源自于 Linux 容器 LXC(linux container), 听起名字就知道, 这和沙箱应该差不多, 可以把东西分开放, 也就是隔离的意思, 甚至可以在某些仓库中看到 docker 的名字叫 lxc-docker

而 LXC 又是基于 cgroup 的 namespace, chroot 等, 由于我并不懂这些, 但 namespace 可以帮助我们理解, 就像我们写程序一样, 这是一个命名空间, 与其他 namespace 相区别

cgroup 对于 docker 是至关重要的, 了解它才会觉得 docker 不神秘, cgroup 全称为 control group, 是 linux 内核提供的功能, 简单的说, 它的作用就是把系统运行的进程按用户自定义的群组区分, 也就是说 一个 docker, 一个 group

cgroup 有限制使用资源的能力

- blkio -- 这个子系统为块设备设定输入/输出限制，比如物理设备（磁盘，固态硬盘，USB 等等）

- cpu -- 这个子系统使用调度程序提供对 CPU 的 cgroup 任务访问

- cpuacct -- 这个子系统自动生成 cgroup 中任务所使用的 CPU 报告

- cpuset -- 这个子系统为 cgroup 中的任务分配独立 CPU（在多核系统）和内存节点

- devices -- 这个子系统可允许或者拒绝 cgroup 中的任务访问设备

- freezer -- 这个子系统挂起或者恢复 cgroup 中的任务

- memory -- 这个子系统设定 cgroup 中任务使用的内存限制，并自动生成由那些任务使用的内存资源报告

- net_cls -- 这个子系统使用等级识别符（classid）标记网络数据包，可允许 Linux 流量控制程序（tc）识别从具体 cgroup 中生成的数据包

- ns -- 名称空间子系统

看到上面的 cgroup 配置属性真的是恍然大悟, docker 成为当今最热的运维神器一点都不奇怪, 它简直就是为了运维而生, **弹性计算**, **资源报表**, **暂停服务**, **资源限制** 根本不在话下


入门教程
---

帮 docker 吹了这么多牛, 得说说具体怎么用

### 安装

不管你是什么系统, 你应该首先尝试脚本一键安装, 也是官方推荐的方法

```sh
sudo curl -sL https://get.docker.io/ | sh 
```

如果你很不幸的失败了, 那你也许是苦逼的 centos 或者 redhat 使用者, 那建议你使用

```sh
yum search docker
```

运气好的话你可以找到一个叫 docker.io 的程序, 没错就是他

如果再不行, 恩, 那就只能上官网看英语安装文档了

不得不承认我至今都没有成功的从源码安装过


### 运行

docker 也需要一个 daemon(后台), 所以首先要启动 docker 服务(不知为啥很多教程根本不提这一步)

```
sudo docker -d
# 或者
sudo docker -H tcp://127.0.0.1:4243 -d
```

如果没报啥错误, 并且 `docker images` 也能看到一个列表目录而不是报错, 那少年你一定是成功了

> docker 依赖系统模块 bridge, 可以用 `modprobe bridge` 来查看有没有

> 阿里云默认是启动不了 docker 的, 输入 `route del -net 172.16.0.0 netmask 255.240.0.0` 可解决


### 使用

使用前你不得不了解两个概念, 一个叫 image, 一个叫 container, 对初学者来说这俩可能意思有点接近或者混淆, 看[这些比喻](https://docker.cn/p/a-discussion-on-the-relation-between-image-and-container)也许你一下子就明白了, image 是只读的模板, 用来生成你需要的 container, 而 container 也可以变成新的 image

使用 docker 就是使用 container, 而 container 来自于 image, 因此你需要先有个 image, docker 的操作像极了 git, 你可以这样下载一个 image

```sh
docker pull ubuntu
```

这样你就有了一个 ubuntu image, 可以用 `docker images` 看到这个新的 image

使用这个 image, 可以向它发送一个命令, docker run ubuntu echo hello docker

当然, 没人会这么无聊, come on! 我们需要像虚拟机一样使用它, 搭建属于我们自己的环境, 自己的系统, 把搭建完成的 container 变成一个新的 image, 这才是我们的目标!

```sh
docker run -it ubuntu /bin/bash
```

没错, 这条神奇的命令终于让 docker 变得有趣起来, 它让我们像 ssh 进入虚拟机一样操作, 翱翔

> 退出可以用 exit, 或者 CTRL + D

可惜的是, 一旦退出, container 不在维持了, 我们不可能一直在 container 中不出来, 但我们也要保持 container 的状态, 那怎么办呢

```sh
docker run -itd ubuntu /bin/bash # 后台执行 container

docker ps # 找到后台执行的 container id 或昵称

docker attach <container id> # 重新 attach 这个 container
```

> 注意, 这时候如果 exit 依然会终止这个 container, 要想 detach 跳出一个 container, 你需要使用 `CTRL + P, CTRL + Q`, 这样我们就又能用 attach 重新进入 container, 真是来去自如

也许你不了解这些命令参数的意思, 不要紧, 本文并不准备当一个解释参数的大妈, docker 像 git, svn 那样可以用 `docker help <command>` 来非常方便的查看这些帮助

觉得实用就赶紧试试吧!


