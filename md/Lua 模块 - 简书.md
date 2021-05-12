> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.jianshu.com](https://www.jianshu.com/p/aef15eb52927)

Lua 包库为 Lua 提供简易的加载和创建模块的方法，由`require`方法、`module`方法和`package`表组成...

从 Lua5.1 开始，对模块和包添加了新的支持，可使用`module`和`require`来定义和使用模块 (module) 和包（package）。在 Lua 中，模块是“第一类值”，一个模块就是一个程序库，可通过`require(module)`来加载后获得一个全局的`table`变量，这个`table`类似命名空间，其内容就是模块中导出的所有东西，如函数和变量。

module
------

模块定义的原始行为为

```
local module = ...
local M = {}
_G[module] = M
package.loaded[module] = M
-- setup for external access
setfenv(1, M)
```

Lua5.1 提供新函数`module()`，调用时会创建表并将其赋予给全局变量和`loaded table`，最后还会将这个表设置为主程序块的环境。

默认情况下，`module`不提供外部访问，必须在调用前为需要访问的外部函数或模块声明适当的局部变量。也可以通过继承来实现外部访问，只需要在调用`module`时添加`package.seeall`选项。

```
module(..., package.seeall)
-- 等价于
setmetatable(M, {--index = _G}
```

模块的处理流程

```
# 建立一个模块
module(module_name, [, ...])

module(module_name, callback1, callback2, ...)
```

1.  如果`package.loaded[module_name]`是一个`table`，则将这个`table`作为一个`module`。
2.  如果全局变量`module_name`是一个`table`，则将全局变量作为一个`module`。
3.  当前两种情况都不存在表`module_name`时，将新建一个`table`，并使其作为全局名为`module_name`的值，并执行`package.loaded[module_name]`...
4.  依次调用回调函数
5.  将当前模块的环境设置为`module`，同时将`package.loaded[module_name] = module`

模块的使用

```
-- 在模块文件中使用module函数
module "module_name"

--[[
等同语法
--]]

-- 定义模块名
local moduleName = "module_name"
-- 定义用于返回的模块表
local M = {}
-- 将模块表加入到全局变量
_G[moduleName] = M
-- 将模块表加入到package.loaded中防止多次加载
package.loaded[moduleName] = M
-- 将模块表设置为函数的环境表，使得模块中的所有操作是在模块表中，这样定义函数就直接定义在模块表中。
setfenv(1, M)
```

通过`module()`可以方便的编写模块中的内容，`module`指令运行完后，整个环境都被压栈，将导致前面全局的数据再也看不到了。

```
local _G = _G
module("module_name")
```

另一种巧妙的方式是 Lua5.1 提供了`package.seeall`作为`module`的`option`传入`module("module_name", package.seeall)`。

通过`module("module_name", package.seeall)`来显式声明一个包，但官方不推荐使用这种方式，因为：

*   `package.seeall`的方式破坏了模块的高内聚，原本引入`old_module`指向调用它的`foo()`函数，但是它却可以读写全局属性，如`old_module.os`。
*   `module`函数的`side-effect`，会污染全局环境变量。

所以，还是通过`return table`来实现模块更为优雅。

模块定义

有时候需要将一个模块重命名，以避免命名冲突。例如在测试中需加载同一模块的不同版本，而获得版本之间的性能区别。如何加载同一模块的不同版本呢？对于一个 Lua 文件而言，可以很轻易的重命名。但对于一个 C 程序库，是没有办法编辑其中的`luaopen_*`函数的名称的。

为了重命名的需求，`require`用到一个技巧：若一个模块名称中包含了连字符，`require`就会用连字符后的内容来创建`luaopen_*`函数名。如此一来，对于不同版本进行测试的需求即可迎刃而解了。

在 Lua 中创建一个模块最简单的方式是创建一个`table`，并将所有需要导出的函数放入其中，最后返回这个`table`。

```
-- 定义全局变量的模块名称
modname = {} 

function modname.new(i,j)
  retun {i=i, j=j}
end

-- 定义常量
modname.i = modname.nex(0,1)

function modname.add(c1,c2)
  return modname.new(c1.i+c2.i, c1.j+c2j)
end

-- 返回模块的table
return module
```

缺陷：必须显式地将模块名放到每个函数定义中，函数调用时必须限定被调用函数名称。  
思路：在模块中定义一个局部的`table`类型的变量，通过这个局部变量来定义和调用模块内的函数，然后将这个局部名称赋予模块的最终名称。

```
-- 定义局部变量
local M = {}
-- 将局部变量最终赋值给全局模块名
modname = M

function M.new(i,j)
  return {i=i, j=j}
end

-- 定义常量
M.i = M.new(0,1)

function M.add(c1,c2)
  return M.new(c1.i+c2.i, c1.j+c2.j)
end

-- 返回模块的table
return modname
```

缺陷：模块内部其实使用的是一个局部变量，简单粗暴。模块内的函数仍需要一个前缀，如何完全避免写模块名称呢？  
思路：消除前缀，将局部变量最终赋值给模块名。

```
$ vim mod.lua

-- 定义局部模块名称
local modname = ...

-- 打印参数
for i=1, select('#', ...) do
  print(select(i, ...))
end

-- 定义局部变量
local M = {} 
-- 将局部变量最终赋值给模块名
_G[modname] = M 
complex = M

function M.new(i,j)
  return {i=i, j=j}
end

-- 定义常量
M.i = M.new(0,1)

function M.add(c1,c2)
  return M.new(c1.i+c2.i, c1.j+c2.j)
end

-- 返回模块的table
return complex

$ vim test.lua
require "mod"
c1 = mod.new(0,1)
c2 = mod.new(1,2)
ret  = mod.add(c1,c2)
print(ret.i, ret.j)
```

注意：

*   `...`的作用是可以完全不用在模块中定义模块名称，若需重命名模块仅需重命名定义它的文件即可
*   `return` 定义模块时`return`是非常漏写的，是否可以将所有与模块相关的设置任务都集中在模块开头呢？

思路：消除`return`

*   消除`return`方法是将模块`table`直接赋值给`package.loaded`即可
*   `require`会将模块名作为参数传递给模块，即无需`return`模块名称，因为若一个模块没有返回值的话，
*   `require`就会返回`package.loaded[module]`的当前值。

```
$ vim mod.lua

-- 三个点以避免模块重命名问题
local modname = ...
-- 局部变量
local M = {}
-- 将局部变量赋值给模块名
_G[modname] = M
-- 消除结尾return直接将模块赋值给package.loaded
package.loaded[modname] = M
```

`package.loaded`是什么呢？`require`会将返回值存储到`package.loaded`的`table`中，若加载器`loader`没有返回值，`require`会返回`package.loaded`中`table`的值。

缺陷：访问同一模块中其他函数时都需要添加限定名称，当模块内部的一个`local`函数由私有转换为公有后，相应的调用`local`函数的地方都需要修改。

思路：通过 “函数环境” 可解决这个问题，可以让模块的主程序块有一个独占的环境，这样不仅它的所有函数都可共享这个`table`，而且它的所有全局变量也都记录在这个`table`中，还可将所有函数声明为全局变量。这样他们就都自动地记录在一个独立的`table`中。而模块要做的就是将这个`table`赋予模块名和`package.loaded`。

```
$ vim mod.lua

local modname =  ...

local M = {}
_G[modname ] = M

package.loaded[modname ] = M

-- 使用函数环境，无需return，因为模块无返回值，require会返回 package.loaded[modname]的当前值。
-- 当调用setfenv之后，将一个空table的M作为环境后，就无法访问前一个环境中全局变量了。
setfenv(1,M) -- 设置函数环境后就再也不能使用_G中table的内容了

function new(i,j)
  return {i=i, j=j}
end
function add(c1,c2)
  return new(c1.i+c2.i, c1.j+c2.j)
end
```

缺陷：当调用`setfenv`之后，将一个空`table`的`M`作为环境后，就无法访问前一个环境中全局变量。  
思路 1：最简单的方式是使用元表，通过设置`__index`，模拟继承来实现。

```
$ vim mod.lua

local modname = ...

local M = {}
_G[modname] = M

package.loaded[modname] = M

setmetatable(M, {__index = _G})
setfenv(1, M)
```

缺陷：设置元表会有一点的开销  
思路 2：使用局部变量保存全局的环境变量，当访问前一个环境中的变量时，需添加前缀`_G`，由于没有涉及到元方法，此方式比第一种略快。

```
$ vim mod.lua

local modname = ...

local M = {}
_G[modname] = M

package.loaded[modname] = M

local _G = _G -- 保存全局的环境变量
setfenv(1, M)
```

思路 3：最正规的方法是将那些需要用到的函数或模块声明为局部变量，此方式所需做的工作是最多的，但是性能是最好的。

```
$ vim mod.lua

local modname = ...

local M = {}
_G[modname] = M

package.loaded[modname] = M

-- 将所需使用的模块声明为局部变量先保存下来
local sqrt = math.sqrt
local io = io

setfenv(1,M)
```

综上所得，定义一个模块时的步骤如下：

1.  从`require`传入的参数中获取模块名

```
local modname = ...
```

2.  建立一个空的`table`

```
local M = {}
```

3.  在全局环境`_G`中添加模块名对应的字段，将空`table`赋值给此字段。

```
_G[modname] = M
```

4.  在已经加载的`table`中设置该模块

```
package.loaded[modname] = M
```

5.  设置环境变量

```
setfenv(1, M)
```

为简化操作，Lua5.1 + 提供了`module()`函数，它包含了以上这些步骤完成的功能，在编写模块时，直接替代上述的操作。

```
module(...)
```

默认情况下，`module`不提供对外访问，也就是说你是无法访问前一个环境的，必须在调用它之前为所需访问的外部函数或模块声明强档的局部变量。也可以通过继承来实现外部访问。只需在`module`上添加`package.seeall`选项。

```
-- 功能相当于在之前基础上添加了 setmetatable(M, {__index=_G})
module(..., package.seeall)
```

require
-------

Lua 提供高级的`require`函数来加载运行库，简单来说`require`和`dofile`完成同样功能但有 2 点不同：

1.  `require`会搜索目录加载文件
2.  `require`会判断是否文件已经加载，避免重复加载同一个文件。

由于上述特征，`require`在 Lua 中是加载库的更好的函数。

加载指定的模块

```
# 加载指定的模块
require(module_name)
```

`require()`函数先检测`package.loaded`表中是否存在`module_name`，若存在则直接返回当中的值，若不存在则通过定义的加载器加载`module_name`。

从 Lua5.1 + 以后，Lua 使用标准的模块管理库，所有模块加载都是通过`require()`完成。`require()`设计的颇具扩展性，它会从若干个已定义的`loader`中逐个尝试加载新的模块。系统库中提供 4 个`loader`，分别实现已加载模块、Lua 模块、C 扩展模块。这些`loader`以 `CFunction`的形式存放在`require`的环境中的一个`table`中。若想更换 Lua 模块的加载方式，只需替换或增加一个新的`loader`即可。

```
local module = require('module_name')
```

`require`执行流程

1.  在`package.loaded`中查找`module_name`

```
for k,v in pairs(package.loaded) do
    print(k, v)
end
print(math.pi);
```

2.  在`package.preload`中查找`module_name`，若`preload`中存在则将其作为`loader`并调用`loader(L)`
3.  根据`package.path`查找 Lua 文件

`package.path`保存加载外部模块的搜索路径，这种路径是 “模板式的路径”，路径中会包含可替换符号`?`，这个符号会被替换然后 Lua 查找这个文件是否存在，若存在就会调用其中特定的接口。

`package.path`在虚拟机启动的是时候设置，若存在环境变量`LUA_PATH`则使用环境变量作为其值，并将环境变量中的`;;`替换为`luaconf.h`中定义的默认值，若不存在该变量就直接使用`luaconf.h`定义的默认值。

```
print(package.path)
```

4.  根据`packkage.cpath`查找 C 库，并调用相应名称的接口。

`package.cpath`的作用和`package.path`一样，但它是用于加载第三方 C 库，其初始值可通过环境变量`LUA_CPATH`来设置。

`package.loadlib(libname, func)`相当于手工打开 C 库`libname`，并导出函数`func`后返回，`loadlib`其实是`ll_loadlib`。

```
print(package.cpath)
C:\lua\?.dll;C:\lua\..\lib\lua\5.3\?.dll;C:\lua\loadall.dll;.\?.dll
```

```
function require(module)
    -- 判断模块是否已经被加载
    if not package.loaded[module] then
        -- 获取模块的加载器
        local loader = findloader(module)
        if loader==nil then
            error("unable to load module "..module)
        end
        -- 将模块标记为已加载
        package.loaded[module] = true
        -- 初始化模块
        local result = loader(module)
        if result~=nil then
            package.loaded[module] = result
        end
    end
    return package.loaded[module]
end
```

搜索目录加载文件

```
?;?.lua;c:\windows\?;/usr/local/lua/?/?.lua
```

`require`使用的路径和普通路径是有些区别的，普通路径是一个目录列表，而`require`路径是一个模式列表，每个模式指明一种由虚文件名（`require`的参数）转成实文件名的方法。更加明确的说，每个模式都是一个包含可选的问号`?`的文件名。匹配时 Lua 会首先将问号`?`用虚文件名替换，然后查看文件是否存在。如若不存在则继续使用同样的方法用第二个模式匹配。

`require`关注的问题只有分号`;`（模式之间的分隔符）和问号`?`，其他的信息（目录分隔符，文件扩展名）在路径中定义。为了确定路径，Lua 首先检查全局变量`LUA_PATH`是否为一个字符串，若是则认为此字符串就是路径。否则`require`会检查环境变量`LUA_PATH`的值。如果两者都是失败，`require`则使用固定的路径。

要加载一个模块，就必须知道模块在哪里。Windows 平台中会根据环境变量 Path 来搜索，`require`使用的路径与传统路径不同，采用的路径是一连串的模式，其中每项都是将模块名转换为文件名的方式。`requie`会使用模块名来替换`?`。然后根据替换的结果来检查是否存在文件，若不存在则尝试下一项。路径中每项都是以分号分割。`require`只处理`;`和`?`，其它的都由路径自己定义。

```
?;?.lua;c:\windows\?;/usr/local/lua/?/?.lua
```

`require`的路径是一个模式列表，使用提供给`require`的虚文件名去替换模式中的问号，并判断文件是否存在，若不存在则使用第二个模式尝试匹配。为了让`require`能找到自己编写的 Lua 模块，需要把该模块的路径加入到`LUA_PATH`中，在 LuaStudio 中是`package.path`。

Lua 中有一个`table`用来保存所有加载过的文件列表，在 LuaStudio 中是`package.loaded`。可通过查看`package.loaded`表中是否存在所要加载的文件名来判断是否已经加载过。

实际编程中,`require`用于搜索的 Lua 文件的路径存放在变量`package.path`中。当 Lua 启动时，便以环境变量`LUA_PATH`的值来初始化变量`package.path`。若无`LUA_PATH`则使用一个编译时定义的默认路径来初始化。

```
$ lua
Lua 5.3.4  Copyright (C) 1994-2017 Lua.org, PUC-Rio

> print(package.path)
C:\lua\lua\?.lua;C:\lua\lua\?\init.lua;C:\lua\?.lua;C:\lua\?\init.lua;C:\lua\..\share\lua\5.3\?.lua;C:\lua\..\share\lua\5.3\?\init.lua;.\?.lua;.\?\init.lua

> print(package.cpath)
C:\lua\?.dll;C:\lua\..\lib\lua\5.3\?.dll;C:\lua\loadall.dll;.\?.dll
```

`require` 函数是如何加载模块的呢？

*   首先检查`package.loaded`是否已经加载。
*   若`require`为指定模块找到了一个 Lua 文件，它就会通过`loadfile`来加载该文件。
*   若`require`无法找到与模式名相符的 Lua 文件，就会寻找 C 程序库，其搜索地址为`package.cpath`对应的路径。
*   若找到的是一个 C 程序库，则通过`loadlib`来加载。

注意的是`loadfile`和`loadlib`仅仅只是加载代码，并未运行他们。为了运行代码，`require`会以模块名作为参数来调用代码。

require & dofile & loadfile

Lua 提供`require()`函数用来加载运行库，`require()`和`dofile()`完成相同的功能，不同点是

*   `require`会搜索目录以加载文件
*   `require`不会重复加载同一模块
*   若要让每次加载文件都执行，可使用`dofile`。
*   若加载后不执行，等需要时执行，可使用`loadfile`。