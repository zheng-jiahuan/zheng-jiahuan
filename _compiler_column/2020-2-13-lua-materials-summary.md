---
layout: compiler_column
title: Lua参考资料汇总
description: 本文列举了一些学习Lua相关知识的参考资料
author: 郑憨憨 zhenghanhan
excerpt_separator: <!--more-->
---

本文列举了一些学习Lua相关知识的参考资料，和大家一起学习交流<!--more-->

## 官方资料
本节信息来自于[Lua官网](http://www.lua.org/)
1. [About](http://www.lua.org/about.html)页面总体介绍了Lua是什么、有什么特点等，比如：
> Lua is **dynamically typed**, runs by interpreting bytecode with a **register-based** virtual machine, and has **automatic memory management** with **incremental garbage collection**, making it ideal for configuration, scripting, and rapid prototyping.

2. [Doc](http://www.lua.org/docs.html)页面 汇聚了很多官方文档，如下：
   - [Reference Manual](https://www.lua.org/manual/5.3/manual.html)
   - Technical documentation
   - Papers
     - [Lua – an extensible extension language](https://www.lua.org/spe.html) by R. Ierusalimschy, L. H. de Figueiredo, W. Celes,Software: Practice & Experience 26 #6 (1996) 635–652
     - [A look at the design of Lua](http://www.lua.org/doc/cacm2018.pdf) by R. Ierusalimschy, L. H. de Figueiredo, W. Celes, Communications of the ACM 61 #11 (2018) 114–123.
     - [The Evolution of Lua](https://www.lua.org/doc/hopl.pdf)
     - [**The Implementation of Lua 5**](http://www.lua.org/doc/jucs05.pdf)
   - Books
     - Programming in Lua, fourth edition, August 2016

3. [Download](http://www.lua.org/ftp/)页面 提供了Lua的各个版本的下载链接，笔者下载了最新的5.3.5，之后进行了解压，在笔者机器上的目录为~/workspace/lua/lua-project/lua-5.3.5

4. [ReadMe](https://www.lua.org/manual/5.3/readme.html)页面 介绍了Lua的安装(笔者当前看到的最新为5.3版本)，笔者操作系统为Ubuntu16.04
   1. build Lua源码的时候对readline开发包有依赖 所以首先
    > sudo apt-get install libreadline-dev
   2. 接着在~/workspace/lua/lua-project/lua-5.3.5/目录下执行构建和安装
    > sudo make linux install

    我们发现安装到这个地方了<br>
    cd src && mkdir -p /usr/local/bin /usr/local/include /usr/local/lib /usr/local/man/man1 /usr/local/share/lua/5.3 /usr/local/lib/lua/5.3
    cd src && install -p -m 0755 lua luac /usr/local/bin<br>
    cd src && install -p -m 0644 lua.h luaconf.h lualib.h lauxlib.h lua.hpp /usr/local/include<br>
    cd src && install -p -m 0644 liblua.a /usr/local/lib<br>
    cd doc && install -p -m 0644 lua.1 luac.1 /usr/local/man/man1<br>
    其中include和lib是开发lua程序需要的，如果只是想要运行lua程序，那么只需要/usr/local/bin下面的可执行lua文件就好了,我们的操作系统可能已经内置了Lua，需要确保使用的是我们新编译的lua，如果不是的，可以在你使用的bash配置文件中设置PATH环境变量，将/usr/local/bin排在前面

5. [Lua在线源码](https://www.lua.org/source/) 可以通过浏览器浏览对应版本的Lua源码

## 其他权威资料
- [http://lua-users.org/](http://lua-users.org/)
  > lua-users.org is an internet site for and by users of the programming language Lua.<br>

  其中有一些不错的子模块，例如
  - [lua-users wiki](http://lua-users.org/wiki/)
    - [LuaDirectory](http://lua-users.org/wiki/LuaDirectory)
      - [LuaSource](http://lua-users.org/wiki/LuaSource) comments on the source code and implementation of Lua.
      - [ModifyingLua](http://lua-users.org/wiki/ModifyingLua) example of changing Lua itself


- 源码阅读顺序推荐

  Mike Pall在这个[post](https://www.reddit.com/r/programming/comments/63hth/ask_reddit_which_oss_codebases_out_there_are_so/c02pxbp/)上面给了一个阅读Lua源代码的次序表，我把他粘贴出来放在下面：

  Recommended reading order:
   - lmathlib.c, lstrlib.c: get familiar with the external C API. Don't bother with the pattern matcher though. Just the easy functions.
   - lapi.c: Check how the API is implemented internally. Only skim this to get a feeling for the code. Cross-reference to lua.h and luaconf.h as needed.
   - lobject.h: tagged values and object representation. skim through this first. you'll want to keep a window with this file open all the time.
   - lstate.h: state objects. ditto.
   - lopcodes.h: bytecode instruction format and opcode definitions. easy.
   - lvm.c: scroll down to luaV_execute, the main interpreter loop. see how all of the instructions are implemented. skip the details for now. reread later.
   - ldo.c: calls, stacks, exceptions, coroutines. tough read.
   - lstring.c: string interning. cute, huh?
   - ltable.c: hash tables and arrays. tricky code.
   - ltm.c: metamethod handling, reread all of lvm.c now.
   - You may want to reread lapi.c now.
   - ldebug.c: surprise waiting for you. abstract interpretation is used to find object names for tracebacks. does bytecode verification, too.
   - lparser.c, lcode.c: recursive descent parser, targetting a register-based VM. start from chunk() and work your way through. read the expression parser and the code generator parts last.
   - lgc.c: incremental garbage collector. take your time.
   - Read all the other files as you see references to them. Don't let your stack get too deep though.


- 博客资料
  - lua stack and register
      - [https://the-ravi-programming-language.readthedocs.io/en/latest/lua-parser.html](https://the-ravi-programming-language.readthedocs.io/en/latest/lua-parser.html)
  - Lua Bytecode Reference
      - [https://the-ravi-programming-language.readthedocs.io/en/latest/lua_bytecode_reference.html](https://the-ravi-programming-language.readthedocs.io/en/latest/lua_bytecode_reference.html)
      - [A No-Frills Introduction to Lua 5.1 VM Instructions](http://luaforge.net/docman/83/98/ANoFrillsIntroToLua51VMInstructions.pdf)
      - [Lua 5.2 Bytecode and Virtual Machine](http://files.catwell.info/misc/mirror/lua-5.2-bytecode-vm-dirk-laurie/lua52vm.html)
  - lua 基础数据结构（TValue、TString、lua_State）
      - [http://geekluo.com/](http://geekluo.com/)
  - lua精选阅读-[tufts大学](https://www.google.com/search?rlz=1C1SQJL_zh-CNUS821US821&sxsrf=ALeKk03birZEhE4lNcGbXYcXKcn0u_9AOA%3A1582076966466&ei=JpRMXuX_G82rtQaQroawCg&q=tufts+&oq=tufts+&gs_l=psy-ab.3..35i39.376261.376261..376549...0.0..0.875.1391.5-1j1......0....1..gws-wiz.olapw8zcCcw&ved=0ahUKEwjlqt_8v9znAhXNVc0KHRCXAaYQ4dUDCAs&uact=5)
      - [http://www.cs.tufts.edu/comp/250RTS/#lua](http://www.cs.tufts.edu/comp/250RTS/#lua)
      - [Notes on Lua code](http://www.cs.tufts.edu/comp/250RTS/lua.html)
