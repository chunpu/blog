---
layout: post
title: 从零单排C语言-01
tags:
- learn-c
---

突发奇想, 决定重新学习C语言, 虽然已经毕业一年多, 现在只会 `printf`, 但浪子回头, 终究不晚

学习C语言, 我决定从 [coreutils](http://www.gnu.org/software/coreutils/) 入手, 可以看到简单的 C 语言程序, 同时学习最基本的 Linux 命令, coreutils 的 git 地址是 <git://git.sv.gnu.org/coreutils>

在学习其源码之前, 我们还要学会两个看源码神器, 一个是 `CTRL + ]`, 一个是 `SHIFT + K`

### MAN

面对 linux 新手, 老鸟们总会说一句, 不知道就 man 一下, man 是 manual 的简称, 也就是说明书, 在 vim 中查看系统函数的说明非常方便, 按下 `SHIFT + K` 即可, 系统函数一般属于 man(3), 如果你还不了解 man 的数字区域划分, 可以输入 `man man`, 来看 man 自己的说明

![shift-k-1]({{site.static}}learn-c/shift-k-1.png)

按下 `SHIFT + K` 查看系统函数

![shift-k-2]({{site.static}}learn-c/shift-k-2.png)

### CTAGS

其中 `CTRL + ]` 对应的就是 ctags 的快捷键, 什么是 ctags 呢, 用一句话说, ctags 给你的源码打上各种 tag, 使得你可以在函数调用的地方看到函数的定义(宏定义也行), 如果你还是用 eclipse 之类的 IDE, 那就更好理解了, 就是看到一个函数, 按着 CTRL 点进去这个功能.

安装 ctags

ctags 在源中名字可能叫 ctags 也可能叫 exuberant-ctags, 建议用 `apt-cache search` 或者 `yum search` 查找 ctags. 我这儿就是后者, 于是用 `apt-get install exuberant-ctags` 即可

ctags 基本使用方法

1. 建立 tags, `ctags -R`, 在当前目录下递归建立 tags, 输入完后可以看到当前目录下多了个叫 tags 的文件, 这个就是 ctags 产生的标签文件

2. 打开一个 c 文件, 在函数调用处输入 `CTRL + ]`, 即可跳入函数声明, 如果想跳出来, 则是 `CTRL + T`

进阶1: ctags 并非完全智能, 如果有多个地方定义宏或者函数, ctags 只会选择第一个跳入, 这时候需要我们学会一个 vim 小命令 `:ts`, 可以在 vim 中显示所有定义的列表

![vim-ts]({{site.static}}learn-c/vim-ts.png)

进阶2: 如果想要查看系统库里的函数定义, 一般为 `/usr/include` 中的库文件, 那我们需要连着库文件一起打 tag, `ctags -R /usr/include .`, 这会一下子让生成的 tags 文件, 变得很大(我这儿大了 17M), 但因此也就可以看系统库函数调用了, 比如看 strcmp 其实是什么东西

![strcmp]({{site.static}}learn-c/strcmp.png)

> 小姿势: 使用 vim 看 c 文件的时候, vim 只会载入当前执行目录的 tags 文件, 因此如果你不在 tags 所在目录执行 vim, ctags 是失效的
