---
layout: post
title: objective-c初学
tags:
- objc
- ios
---

首先要注意, objc的程序大多是对armv7架构的, 想像C++那样直接用vim, 然后clang编译, 运行, 这是很难办到的, 也就是说, 你必须要有xcode

`.m`表示implement
`[]` 消息表达式
`id` 表示随便, 即存放一个指针
+ 类方法
- 实例方法

### @autoreleasepool

新建一个模板, 我们知道, 文件的入口就是main.m中的main函数, 里面有个autoreleasepool. 不管用不用ARC(Automatic Reference Counting自动引用记数)都推荐的内存管理方法, [官方说明](https://developer.apple.com/library/mac/documentation/Cocoa/Reference/Foundation/Classes/NSAutoreleasePool_Class/Reference/Reference.html). 内存的事既然交给了编译器, 那我们也不用去管release啥的, 更不用去深究其原理


### xml storyboard

### outlet, action, outlet collection

<JSExport>

jsExport.h
