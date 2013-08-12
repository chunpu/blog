---
layout: post
title: Unix 文件描述符
tags:
  - linux
---

Linux通过文件来提供底层接口, 有句话说的好, Linux中一切都是文件.

不管是系统信息(`/proc`)还是目录, 甚至是socket套接字, 都是文件.

程序使用文件是通过文件描述符, 英文叫file description. 通常简写为fd.

文件描述符是一个非负整形.

程序进程运行的时候, 会默认有三个预置的文件描述符,分别是:

- 0. 标准输入, stdin

- 1. 标准输出, stdout

- 2. 标准错误输出, stderr

在shell中, 我们用小于号`<`给程序提供标准输入

```bash
nc -n 192.168.1.2 1111 < somefile

```

用大于号`>`提供标准输出

```bash
svn log > /tmp/svnlog
```

用`2>`提供错误输出

```bash
ping 192.168.1.100 2> /tmp/pingerr

```

Linux提供标准的POSIX接口, 什么是POSIX?

> POSIX 表示可移植操作系统接口（Portable Operating System Interface ，缩写为 POSIX 是为了读音更像 UNIX）

其中主要的有4个函数, read, write, open, close

他们的接口果然很标准, 很好记

```c
read(fd, buf, len);

write(fd, pos, last - pos);

open(filename, flags); // return fd

close(fd);
```

他们打开的时候如果错误, 都是返回负数, 所以习惯上把他们写在if语句中.

使用这些POSIX接口的时候, 必须要引入一个库`unistd.h`

因为标准输出是1, 所以我们可以写出一个最简单的hello world.

```c
#include <unistd.h>

int main() {
  char str[] = "hello world!";
  write(1, str, sizeof(str));
  return 1;
}
```

这里不得不提一下, 我刚开始学c的时候, hello world是用printf写的.

我一直很好奇printf是啥, 有学长跟同学跟我说是print file. 我还是很疑惑, print file是啥玩意儿, 然后就得知Linux中一切都是文件的说法, 屏幕也是文件.

不过man一下printf就知道就是`print format`的意思, 联想一下erlang的`io:format`即可.




