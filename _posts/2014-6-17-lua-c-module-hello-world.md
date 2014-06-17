---
layout: post
title: lua 最简易C模块
tags:
- lua
---

以下是一个最简单的c扩展demo, 几乎不能更简单

`hello.c`:

```c
#include "lua.h"
#include "lauxlib.h"

static int add(lua_State *L) {
    float a1 = lua_tonumber(L, 1);
    float a2 = lua_tonumber(L, 2);
    lua_pushnumber(L, a1 + a2);
    return 1;
}

int luaopen_hello(lua_State *L) {
    lua_register(L, "add", add);
    return 1;
}
```

在lua中使用

```lua
require('hello')
add(11, 22) -- > 33
```

c的lua模块编译成`.so`文件的方法:

```bash
gcc -shared -fPIC hello.c -o hello.so
```

### C扩展中用到的lua函数

`luaopen_hello`: 这个函数相当于c中的main, 也就是说这是一个入口函数, lua代码中require了这个编译出来的hello.so, 就会立即去找`luaopen_hello`这个函数. 也就是说入口函数是`luaopen_文件名`这个格式

`lua_register`: 该函数向全局声明一个名叫name的函数, 接受三个参数, 如下所示

```c
void lua_register (lua_State *L,
                   const char *name,
                   lua_CFunction f);
```

```c
lua_register(L, "add", add);
```

如前面demo所示, 该demo向全局定义了一个名叫add的函数, 该函数可以调用多次, 定义多个函数

`lua_tonumber`: 从参数栈中取值

```c
lua_Number lua_tonumber (lua_State *L, int index); // lua_Number就是double
// 还有其他类似函数
int lua_toboolean (lua_State *L, int index);
lua_CFunction lua_tocfunction (lua_State *L, int index);
lua_Integer lua_tointeger (lua_State *L, int index);
const char *lua_tolstring (lua_State *L, int index, size_t *len);
```

需要注意的是第二个参数index可以是负值, `lua_tonumber(L, -1)`表示取倒数第一个值, 以此类推. 可以通过`lua_gettop(L)`来获取函数传参的个数

`lua_pushnumber`: 向返回值栈中加一个值, 可以看到`lua_tonumber`和`lua_pushnumber`就是一对, 前者是从参数栈中拿一个值作为number, 后者是向结果栈中压入一个number的值.
 显然lua作为一个能返回多值的语言, 我们可以连续push返回值, 但需要注意的是我们的返回值也需要随之增加

```c
static int foo(lua_State *L) {
  lua_pushnumber(L, 1);
  lua_pushnumber(L, 2);
  lua_pushnumber(L, 3);
  return 3; -- 返回几个就传几
}
```
