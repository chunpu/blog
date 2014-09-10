---
layout: post
title: 代码简洁之道
tags:
- code
---

公司的项目, 我从来没有真正从开头参与过, 也就是说, 传说中的接盘侠就是我

接盘对我来说并不算太难, 难的是我必须遵守别人的代码规范, 这实在让人痛不欲生. 从开头写项目的人要远比我了解其中的代码, 同样, 他们也希望接手的人能继续小心翼翼的维护该代码, 按照之前的规则增加代码

今天一个测试妹子说的话让我陷入深深的沉思, "这改动应该很简单吧, 就是加个字段", 我竟然无言以对, 是啊, 这不就是加个类型嘛, 为什么我会改一下午?

DRY(Don't Repeat Yourself), 一个老生长谈的词, 但是做到的却很少

我的代码风格, 通常是简单, 清晰, 语义化, 这和公司中 oo(面向对象) 爱好者格格不入, 作为一个极端痛恨 oo 的人, 我可以很极端的说, 如果你让一个热爱 oo 的人去做一个逻辑很简单的事, 那你就等着招人吧, 原本100行的小脚本, 不一会儿就会变成1000行, 不久又会变成10000行, 最终, 除了那个原始开发者, 再也没人能维护了, 再招人都没用

oo 有几个明显的缺点, 第一, oo 的程序, 几乎有1/3的代码实在做变量赋值, 你会看到满屏幕的

```
_attr1 = attr1
_attr2 = attr2

// 还有
this.attr1 = attr1
this.attr2 = attr2

// 既然有赋值, 那就会有getter, setter, 还有
var obj = new Object(...)
obj.doSomething()
```

God! 这和你的业务有什么关系, 为什么不能直接用一个函数去表达呢?

oo 另一个问题就是 modular, modular 是非常能看出一个程序员水平高低的是, 模块化做得好, 文件分的清晰, 说明大局观清晰, 而 oo 的 modular 是最弱智的, 因为它已经帮你 modular 好了, 也就是一个基类, 然后各种子类, 可能在你的业务中, 只是要处理一个不同的 type 字段, 附加几个不同的值而已, 但在 oo 代码上, 你不得不定义一个子类, 继承父类, 构造, 然后做业务的事. 更甚者, 如果你的子类还有 view, active, template, dao, rpc 好吧, 加一个类型对你来说可能就是加四五个文件

因此, 不用 oo 是保持代码简洁的根本, 但我们仍需有更多需要注意的地方


### 初始化和执行不要分离

有不少程序员写代码, 喜欢先把参数传进对象, 然后再执行

```
var o = new Object(opt)
o.run()
```

为什么就不能写成 `run(opt)` 呢

可惜的是我这个例子太不明显, 但有个典型的例子可以参考, 那就是 Grunt 和 Gulp 的[对比](http://new.w3ctech.com/topic/74)

其实, 在使用中感觉就已经很明显了, GruntFile 基本没人能看懂, 同样, 才4个tasks 的 Grunt 项目, `npm install --dev` 一下子就下载了50M

### 就近声明

代码易读有一个重要原则, 就是就近声明, 如果你把变量都声明在头部, 然后到了很后面才用了一次, 这是让人非常费解的. 但是你在使用之前才声明这个变量, 这就一目了然了

### 使用 util 类库

由于语言本身的函数未必能满足业务的需求, 大多数人开发项目的时候都会加入一个 util 的 lib, 但其 util 却未必能写的简洁好用, 其实 util 的库函数大多都已经很固定了, 不妨参照 [underscore](https://github.com/jashkenas/underscore), 其简洁明了的 api 让人用起来非常舒服, 我见过别人写的 util 库, 一个判断值是否在数组中的函数叫valueInArray, 每次输入这么长的时候我都暗自吐槽为啥就不能叫 has 或者 contains 呢

为什么要按 underscore 设计 api 呢? 原因就是其 api 可以简单的组合在一起, 形成新的 api

重要的一点, util 库在代码中应该是 `_`, 而不是 util, 代码里写这么多 util 累不累啊(坑爹的还有人用 utils..)

### 少用常量

实在不能理解, c 中用来区分宏定义和正常代码的大写命名, 到其他语言变成常量了, 经常看到程序中大量的又臭又长的常量, 和旁边整洁的代码格格不入, 而且读起来也非常困难, 在查 bug 的时候, 又必须开着常量文件, 一个个对比才能找到真实的值, 为什么就不能用原值呢? 因为好维护, 以后这个值变了我改这个常量就行了, 常量爱好者一般都会这么告诉你.

可是我直接用个正常的小写变量不行吗, 更有甚者(我在说java), 会给每个数字, 字符串都套一层常量, 整个代码都已经没法看了, 全是大段大段的大写字母, 这还叫程序吗, 当然, 你都给每个变量套一层了, 当然是改起来就改一个常量, 以后是不是还要给函数都套一层常量, 这样所有 java 代码都是大写了. 对了, 这里有个字面量初始化会有性能损耗的问题, 我实在看不出这比起 oo 带来的性能损耗算什么问题

### 不要写分号

这也是非常关键的, 一个代码洁癖的人, 首先是不会容忍分号的, 当然这只是对允许不写分号的语言来说的, 如果你是 js 开发者, 你可能会反驳, 不写分号会导致问题. 分号导致的这种问题就是基本的词法解析问题, 把括号写在行首的人, 我只能替他拙计, 把符号写行首的也是, 不过我知道不写分号的人这些基本常识都是有的, 菜鸟才会问这问题

### 一行的条件语句不要换行

```
if (true) {
  return
}
==>

if (true) return
```

这么方便的语法, 为什么要 ban 掉呢? 等你真的写出 if if else 的这种代码, 你在把花括号加回去呗


写着写着态度又极端了起来, 我虽然在努力修改自己态度极端的毛病, 但这次还是忍不住, 因为维护同事极度复杂的代码已经让我内伤, 不过正是如此, 才能看到自己的优点(经常有人说看别人的代码怎么都看不懂, 但是你的一看就懂, 而且还短), 嘻嘻~





