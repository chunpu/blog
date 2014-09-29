---
layout: post
title: 打印 as3 中的一切
tags:
- as3
---

as3 如果是用 trace 来调试那必然是痛苦不堪, 下面是我自己常用的打印函数, 有多实用就不谈了

用到的内置库是

```as3
import flash.utils.describeType;
import flash.utils.getQualifiedClassName;
```

打印函数

```as3
public function log(...args):void {
    args = dumpObject(args) as Array
    args.unshift('console.debug')
    try {
        ExternalInterface.call.apply(null, args)
    } catch (e:Error) {}
}
```

对象转换函数

```as3
public function dumpObject(obj:*):* {
    if ('function' == typeof obj) return '<Function>'
    var className:String = getQualifiedClassName(obj)
    var rawTypes:Array = ['Object', 'String', 'Boolean', 'int', 'void', 'null', 'Number']
    if (-1 != rawTypes.indexOf(className)) return obj
    if ('Array' == className) {
        return obj.map(function(item:*):* {
            return dumpObject(item)
        })
    }
    // xml object
    var ret:Object = {
        constructor: className
    }
    var xml:XML = describeType(obj)
    var keys:Array = ['accessor', 'method']
    for (var j:int = 0; j < keys.length; j++) {
        var xmllist:XMLList = xml[keys[j]]
        for (var i:int = 0, item:XML; item = xmllist[i++];) {
            var name:String = item.@name.toString()
            try {
                ret[name] = dumpObject(netStream[name])
            } catch (e:Error) {}
        }
    }
    return ret
}
```

