---
layout: post
title: "Calling C From Lua"
date: 2025-05-02 00:00:00 -0700
tags: c lua fennel
---

I've written C extensions for Python, but I've never done it for Lua before.
The basic idea is that you write a function that takes the `lua_State *L`
pointer, and returns an `int`, referring to the number of values it's pushing
back onto Lua's stack. Then you add some code to make your C code importable
into Lua as a module. Finally, you compile your C code into a shared object
and make it available to your Lua code, either in the same working directory,
or on a path that Lua knows about.

## C Code 

```c
// myclib.c

#include <lua.h>
#include <lauxlib.h>
#include <lualib.h>

static int add_numbers(lua_State *L) {
    /* Check and get the two numbers from the stack */
    double num1 = luaL_checknumber(L, 1);
    double num2 = luaL_checknumber(L, 2);

    /* Perform the addition */
    double result = num1 + num2;

    /* Push the result onto the stack */
    lua_pushnumber(L, result);

    /* Return the number of results (1 in this case) */
    return 1;
}

/* Define a structure to map function names in Lua to your C functions */
static const struct luaL_Reg mylib_funcs[] = {
    {"add", add_numbers},
    {NULL, NULL} /* Sentinel to indicate the end of the array */
};

/* The module entry point function */
int luaopen_myclib(lua_State *L) {
    /* Create a new table and register the functions */
#if LUA_VERSION_NUM == 501
    luaL_register(L, "myclib", mylib_funcs); // For Lua 5.1
#else
    luaL_newlib(L, mylib_funcs); // For Lua 5.2 and later
#endif
    return 1; /* Return the module table */
}
```

I compiled the C code above with this command,

```console
gcc -shared -fPIC -I/opt/homebrew/Cellar/lua/5.4.7/include/lua -L/opt/homebrew/Cellar/lua/5.4.7/lib -llua5.4 -o myclib.so myclib.c
```

The `-L` flag specifies the location of an external library, and the `-l` flag
specifies the name of the shared library you want to link. These flags work
together with the `-fPIC` flag, which generates *position-independent code*,
so that the shared library can be loaded from any address, rather than a static
one. The `-I` flag tells `gcc` where to look for header files, in this case,
the Lua files we used so that the C code could interact with the Lua interpreter.

## Lua Code 

This code allows us to pull the `add` function from our C code, `myclib.c`, via the shared object, `myclib.so`, and call it from out Lua script.

```lua
-- script.lua

-- Require the module (assuming the shared library is named myclib.so/myclib.dll)
local mylib = require("myclib")

-- Call the C function
local sum = mylib.add(3, 8)
print("The sum is:", sum) -- 11
```

## Bonus: Fennel

Fennel is a LISP that compiles to Lua code, so it works almost exactly the same
as the Lua code above.

```fennel
;; script.fnl

;; Load the C module named "myclib"
(local myclib (require "myclib"))

;; Access the "add" function from the mylib module
;; and call it with arguments 10 and 20
(local sum ((. myclib :add) 3 8))

;; Print the result
(print "The sum from C is:" sum)
```

Here, `((. myclib :add) 3 8)` is like `myclib.add(3, 8)` in Lua, we're accessing
the `add` method on the `myclib` table.