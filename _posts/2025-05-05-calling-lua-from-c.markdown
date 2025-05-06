---
layout: post
title: "Calling Lua from C"
date: 2025-05-05 00:00:00 -0700
tags: lua, c
---

In a previous post I covered calling C from Lua. Here, I'll call Lua from C. In
my opinion, this direction is simpler. I understand now why people use Lua as a
plugin and configuration for C/C++ applications.

## Lua Code

{% highlight lua %}
-- script.lua

function add(a, b)
  return a + b
end

greeting = "Hello from global Lua variable!"
{% endhighlight %}

## C Code

{% highlight c %}
// embed.c

#include <stdio.h>
#include <lua.h>
#include <lualib.h>
#include <lauxlib.h>

int main() {

    // Initialize Lua state
    lua_State *L = luaL_newstate(); 
    // Load standard libraries
    luaL_openlibs(L);

    // Load the Lua script
    if (luaL_dofile(L, "my_script.lua") != LUA_OK) {
        fprintf(stderr, "Error loading script: %s\n", lua_tostring(L, -1));
        lua_close(L);
        return 1;
    }

    // Push the function 'add' onto the stack
    lua_getglobal(L, "add");
    // Push first argument (10)
    lua_pushnumber(L, 10);
    // Push second argument (20)
    lua_pushnumber(L, 20);

    // Call the function: 2 args, 1 result, 0 error handler index
    if (lua_pcall(L, 2, 1, 0) != LUA_OK) {
        fprintf(stderr, "Error calling add function: %s\n", lua_tostring(L, -1));
        lua_close(L);
        return 1;
    }

    // Retrieve the result (which is now at the top of the stack, index -1)
    if (lua_isnumber(L, -1)) {
        double result = lua_tonumber(L, -1);
        printf("Lua function 'add' returned: %f\n", result);
    } else {
        printf("Lua function 'add' returned non-number.\n");
    }
    
    // Pop the result from the stack
    lua_pop(L, 1); 

    // Push the global variable 'greeting' onto the stack
    lua_getglobal(L, "greeting"); 

    if (lua_isstring(L, -1)) {
        const char *greeting_msg = lua_tostring(L, -1);
        printf("Lua global variable 'greeting': %s\n", greeting_msg);
    } else {
        printf("Lua global variable 'greeting' is not a string.\n");
    }

    // Pop the greeting string from the stack
    lua_pop(L, 1);

    // Close the Lua state
    lua_close(L); 

    return 0;
}
{% endhighlight %}

Then we can compile this as,

```console
gcc -I/opt/homebrew/Cellar/lua/5.4.7/include/lua -L/opt/homebrew/Cellar/lua/5.4.7/lib -llua5.4 -o embed embed.c
```