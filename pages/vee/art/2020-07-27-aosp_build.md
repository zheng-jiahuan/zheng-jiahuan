---
title: AOSP源码下载、编译与刷机
keywords: art, aosp, build, setting up
last_updated: July 27, 2020
created: 2020-07-27
tags: [art, default]
summary: "本文简要介绍了如何下载和编译AOSP源码并刷机到Pixel 2手机的过程"
sidebar: mydoc_sidebar
permalink: 2020-07-27-aosp_build.html
folder: vee/art
---

## 环境简介
笔者的电脑是一款Win10笔记本电脑（内存16GB * 2），通过VMware® Workstation 15 Pro虚拟机虚拟出一个
[Ubuntu 20.04](https://releases.ubuntu.com/20.04/)操作系统实例。
详细的参数如下图：

![环境配置概览图](images/2020-07-27-aosp_build/vmware-config.png)

分配给这个虚拟机实例的资源配置如下：
![虚拟机实例资源配置](images/2020-07-27-aosp_build/ubuntu20-config.png)

我给ubuntu实例分配了500+GB的磁盘，并分配了22GB的内存（因为编译AOSP很耗费内存），现在编译完了，笔者又把它手动调低到了图中的16-GB。编译AOSP的时候，除了分配22GB的内存外，笔者还在ubuntu实例中分配了一个32GB的swap空间，担心内存不够，发生交换。快速创建swap空间可以参考[这篇文章](https://cloud.tencent.com/developer/article/1631696)。


## AOSP源码下载
由于墙的原因，在国内无法通过[AOSP官网](https://source.android.com/setup/build/downloading)介绍的方式来下载AOSP源码，如果你可以翻墙，那么你依据这个官网步骤来下载AOSP源码就好了。如果翻不了墙，可以通过清华大学提供的[国内镜像](https://mirrors.tuna.tsinghua.edu.cn/help/AOSP/)来下载AOSP源码。
```
wget -c https://mirrors.tuna.tsinghua.edu.cn/aosp-monthly/aosp-latest.tar # 下载初始化包
tar xf aosp-latest.tar
cd AOSP   # 解压得到的 AOSP 工程目录
# 这时 ls 的话什么也看不到，因为只有一个隐藏的 .repo 目录
```
首先通过上述这几个命令下载一个初始的包下来，然后在这个基础包的基础上，来获取你想要的特定版本的aosp源码，
因为笔者在淘宝上买了一个Pixel 2手机用来刷机的，为了一次刷机成功，必须保证要刷一个确定版本的aosp源码。
从这个[官网](https://source.android.com/setup/start/build-numbers#source-code-tags-and-builds)可以获取特定版本支持的特定手机。对于笔者来说选择的是如下所示的AOSP代码。build号是QQ3A.200705.002，tag号是android-10.0.0_r40，记住这两个号。
![本文选定的代码版本信息](images/2020-07-27-aosp_build/src-tag.png)

现在假设我们已经解压aosp-latest.tar并进入了目录AOSP下，我们执行下下面这个命令，告诉repo，我想要同步android-10.0.0_r40对应的AOSP代码。

```
repo init -u https://mirrors.tuna.tsinghua.edu.cn/git/AOSP/platform/manifest -b android-10.0.0_r40
```

上面这个命令执行完毕之后，我们就要开始从服务器上同步这个代码到本地了。同步的命令很简单，就是
```
repo sync
```
但是，在执行之前啊，我们需要设置下git，因为repo sync的过程中可能网络不好或者同步的git库太大之类的造成很多问题，
所以我们需要稍微配置下我们的git工具。说到配置软件等方面，国内的话，ubuntu 20.04的软件来源最好换成清华的源，
sudo vim /etc/apt/sources.list，将内容替换成如下内容：

```
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal main restricted universe multiverse
deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-updates main restricted universe multiverse
deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-updates main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-backports main restricted universe multiverse
deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-backports main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-security main restricted universe multiverse
deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-security main restricted universe multiverse
```
接着，我们重新从源代码来构建一款git出来，因为git默认使用的GnuTls经常造成repo sync过程产生错误，[参照这个文档](https://www.cnblogs.com/sddai/p/10209121.html)，我们构建一个采用openssl的git工具。

构建完成之后，我们接着来配置git的config文件，通过vim ~/.gitconfig这个选项来修改，替换http和https对应的属性值
，这些值使得我们在repo过程中，尽可能小出现错误。
```
[http]
  postBuffer = 1048576000
  sslVerify = false
  lowSpeedLimit = 0
  lowSpeedTime = 999999
[https]
  postBuffer = 1048576000
  sslVerify = false
  lowSpeedLimit = 0
  lowSpeedTime = 999999
[user]
  email = alanwilliams@foxmail.com
  name = zhenghanhan
```
这些都执行完毕后，我们就可以开始执行上面的repo sync命令了，坐等同步完毕，有时候一次性repo sync不成功，需要多执行几次，这个等待时间是蛮久的。



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

`<generator>`：常见的有Ninja、Unix Makefiles 等。

`[options]` ：这里介绍几个常见的

第一个就是 -DLLVM_ENABLE_PROJECTS='...' 选项， -D是define的意思，就是说，将
LLVM_ENABLE_PROJECTS整个变量定义为何值。LLVM_ENABLE_PROJECTS中列的值（如果有
多个值，以封号隔开），表示的是除了LLVM Core libraries外，还需要构建的子项目。本文需要
clang，所以本文-DLLVM_ENABLE_PROJECTS="clang"。
另一个就是 -DCMAKE_BUILD_TYPE=type ，由于默认该项值是Debug，但是由于编译Debug时间太长，
而且对内存要求比较高，笔者这里将其修改成Release，即-DCMAKE_BUILD_TYPE=release
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

笔者之前使用8G x 2的笔记本构建debug版本的llvm等，但是构建失败了，因为OOM了，于是笔者今天把内存换成了
16G x 2的了，分配给虚拟机的内存有25GB左右，然后使用下述命令再次构建。
```
cmake -G  Ninja  -DLLVM_ENABLE_PROJECTS="clang"  -DCMAKE_BUILD_TYPE=debug ../llvm
```
到最后快要构建完的时候，卡住了，卡了足足有10几个小时，整个ubuntu像freeze了一样，如下图：
{% include image.html file="2019-10-06-llvm_src_download_and_compiling/llvm_build_debug_freeze.png" url="" alt="LLVM Debug Build Freeze" caption="LLVM Debug Build Freeze the System" max-width="900" %}
使用htop命令查看了下内存占用，发现内存被吃光了，还吃光了swap分区。经过分析，猜测是ninja编译的时候启动了太多
的线程，以及默认的GNU的ld效率不高，于是我将ninja换成了make，同时linker换成gold，不再使用其默认的gnu linker，并控制开启线程的数量。首先将build目录下
的内容删除干净，然后
```
cmake -G  "Unix Makefiles"  -DLLVM_USE_LINKER=gold   -DLLVM_ENABLE_PROJECTS="clang"  -DCMAKE_BUILD_TYPE=debug ../llvm
```
这样就生成了适合make构建的文件，然后使用
```
make -j4
```
然后使用-j4限制了4线程编译，然后编译成功了。


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
