---
title: LLVM源码下载与编译
keywords: compiler, llvm, getting_started, download and compile
last_updated: October 06, 2019
tags: [getting_started, compiler, llvm]
summary: "本文简要介绍了如何下载和编译LLVM代码"
sidebar: mydoc_sidebar
permalink: 2019-10-06-llvm_src_download_and_compiling.html
folder: compiler
---

## 环境简介
笔者的环境是搭在VMware Workstation虚拟出的Ubuntu 16.04操作系统实例上的

## LLVM源码下载
LLVM的下载来源主要来自于这两个官方链接，一个是LLVM 稳定版本的下载[<sup>1</sup>](#refer-anchor-1)，即稳定的发布版
本。一个是LLVM在github上的下载[<sup>2</sup>](#refer-anchor-2)，这个可以理解成最新的不断更新的开发版本。本文采用后
者。将整个LLVM大项目下载下来（包含clang，llvm core，compiler-rt等等），整个过程比较耗时。

笔者在计算机上手动新建了一个目录，~/workspace/llvm。 然后在该目录下，执行了如下命令： 
```
git clone https://github.com/llvm/llvm-project.git
```


## LLVM源码编译
代码下载完毕后，在~/workspace/llvm目录下，会出现一个目录llvm-project，进入该目录下，
可以看到如下文件陈列：


{% include image.html file="2019-10-06-llvm_src_download_and_compiling/1.png" url="" alt="LLVM Src Download and Compile" caption="LLVM Src Download and Compile" max-width="600" %}

LLVM不支持直接在源码树中直接构建 （in-tree build is not supported），这也比较容易理解，
这样可以使得项目保持整洁，不至于混乱不堪，搅在一起。现在新建一个目录，用来存放待会编译
过程中产生的各种文件。我们在llvm-project这个文件夹下，新建一个目录，目录名可以随意取，
这里取名build，然后我们进入build目录，准备开始构建LLVM的编译。在正式开始之前，我们需
要简单了解一下CMake[<sup>3</sup>](#refer-anchor-3)

> CMake is a cross-platform build-generator tool. CMake does not build the project, it
> generates the files needed by your build tool (GNU make, Visual Studio, etc.) for
> building LLVM.

CMake是跨平台的构建生成器，本身并不构建项目，而是生成各种makefile。真正构建项目的软
件（例如GNU make、Ninja等）将依据这些生成的make file文件来构建项目。

在build目录下，待会要执行如下的命令，从而产生必要的构建文件。

```
cmake -G <generator> [options] ../llvm
```

`<generator>`：常见的有Ninja、Unix Makefiles 等，由于Ninja备受官网推荐，所以本文使用
Ninja。

`[options]` ：这里介绍几个常见的

第一个就是 -DLLVM_ENABLE_PROJECTS='...' 选项， -D是define的意思，就是说，将
LLVM_ENABLE_PROJECTS整个变量定义为何值。LLVM_ENABLE_PROJECTS中列的值（如果有
多个值，以封号隔开），表示的是除了LLVM Core libraries外，还需要构建的子项目。本文需要
clang，所以本文-DLLVM_ENABLE_PROJECTS="clang"。
另一个就是 -DCMAKE_BUILD_TYPE=type ，由于默认该项值是Debug，但是由于编译Debug时间太
长，而且对内存要求比较高，笔者这里将其修改成Release，即-DCMAKE_BUILD_TYPE=release
综上，笔者在build目录下，执行了如下命令：

```
cmake -G  Ninja  -DLLVM_ENABLE_PROJECTS="clang"  -DCMAKE_BUILD_TYPE=release ../llvm
```

在上述命令执行的过程中，cmake会检查当前的环境是否符合后续构建的要求，比如必要的软件是
否安装上、已经安装的软件，其版本是否符合要求等。比如，笔者遇到了Ninja版本不符合要求的
情况:

```
CMake Error: the detected version of Ninja (log: ninja version 0.1.3 initializing) is less than
the version of Ninja required by CMake (1.3).
```
也就是说，检测出Ninja的版本过低，这个时候可以去ninja的官网下载一个最新版本，然后替换掉
现有的低版本的Ninja。替换完毕之后，重新执行上面的cmake命令，仍然会提示上面ninja版本过
低的错误，这是由于，cmake基于上次执行的结果，直接给出的，它并没有刷新，所以，可以粗暴
的将build目录下的文件都删掉，然后重新执行一下上面的cmake命令，它就可以往下走了。

待cmake命令将构建文件生成完毕，此时在build目录下，直接执行
```
ninja
```
就开始构建clang和llvm了。构建完毕之后，build目录结构如下：

{% include image.html file="2019-10-06-llvm_src_download_and_compiling/2.png" url="" alt="LLVM Build DIR" caption="Detail of LLVM Build DIR" max-width="600" %}

我们构建出的各个可执行文件在build/bin目录下，如下图：
{% include image.html file="2019-10-06-llvm_src_download_and_compiling/3.png" url="" alt="LLVM Build BIN DIR" caption="Detail of LLVM Build BIN DIR" max-width="600" %}

将这个bin目录放置到系统的PATH变量中，就可以随时随地使用这些刚刚编译出炉的executable了。

以上部分主要参考这两个官方链接，一个是Getting Started with the LLVM System[<sup>4</sup>](#refer-anchor-4)，
一个是Getting Started: Building and Running Clang[<sup>5</sup>](#refer-anchor-5)，如果想进一步了解细节的，可以参考这两个链接。


## References

<div id="refer-anchor-1"></div>
[1]. [llvm release版本下载](http://releases.llvm.org/download.html)
<div id="refer-anchor-2"></div>
[2]. [llvm github 源码位置](https://github.com/llvm/llvm-project)
<div id="refer-anchor-3"></div>
[3]. [llvm之cmake](http://llvm.org/docs/CMake.html)
<div id="refer-anchor-4"></div>
[4]. [Getting Started with the LLVM System](https://llvm.org/docs/GettingStarted.html)
<div id="refer-anchor-5"></div>
[5]. [Getting Started: Building and Running Clang](http://clang.llvm.org/get_started.html)
