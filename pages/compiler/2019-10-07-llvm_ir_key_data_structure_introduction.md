---
title: LLVM IR-重要数据结构简介
keywords: compiler, llvm ir, getting_started
last_updated: October 07, 2019
tags: [getting_started, compiler, llvm]
summary: "本文简要介绍LLVM IR在内存中的表示形态与结构"
sidebar: mydoc_sidebar
permalink: 2019-10-07-llvm_ir_key_data_structure_introduction.html
folder: compiler
---

## 引言
在LLVM IR简介一文中，我们提到LLVM IR有三种存在形式，在简介一文中介绍了存在在文件中的
可读IR和二进制IR，本文将粗略介绍下IR在内存中的形态对应的数据结构。

## 常见类的层次结构（Class Hierarchy）

1. Value及常见子类

{% include image.html file="2019-10-07-llvm_ir_key_data_structure_introduction/1.png" url="" alt="Value and Its Commonly-Used SubClasses" caption="Value and Its Commonly-Used SubClasses" max-width="600" %}


我大致绘制了一些常见的Value的子类，每个Value类型的实例都是有Type的。关于[Value](http://llvm.org/docs/ProgrammersManual.html#the-value-class)
官方文档有这样一段话：
>The Value class is the most important class in the LLVM Source base. It represents a
>typed value that may be used (among other things) as an operand to an instruction.

在[Value](http://llvm.org/doxygen/Value_8h_source.html)的doxygen中也有一段描述:

>This is a very important LLVM class. It is the base class of all values computed by a
>program that may be used as operands to other values. Value is the super class of
>other important classes such as Instruction and Function. All Values have a Type. Type
>is not a subclass of Value. Some values can have a name and they belong to some
>Module. Setting the name on the Value automatically updates the module's symbol
>table.

从上图中可以看出，在LLVM中，BasicBlock、Instruction、Function、GlobalVariable等概念，
都实现为了Value的子类。除了BasicBlock外，图中列举的类还是User类的子类。不过说完还是比
较空洞，还是结合一个例子来看下比较好：

```c
int func(int x, int y, int z){ 
  x = y + z; 
  y = x + z; 
  z = x + y; 
  return z; 
}
```
将该例子放置在hello.c文件中，并通过下面两个命令将其转换成经过mem2reg优化的可读IR文
件。这两个命令在之前LLVM IR简介一文中有过解释，这里再次粘贴一下：

```
clang hello.c -emit-llvm -S -Xclang -disable-O0-optnone 
opt -mem2reg -S hello.ll > hello.m2r.ll
```

转换的IR文件内容如下：
```c
define dso_local i32 @func(i32 %0, i32 %1, i32 %2) #0 { 
  %4 = add nsw i32 %1, %2 
  %5 = add nsw i32 %4, %2 
  %6 = add nsw i32 %4, %5 
  ret i32 %6 
} 
```
为了说明问题，将其转化成下图：
{% include image.html file="2019-10-07-llvm_ir_key_data_structure_introduction/2.png" url="" alt="IR Illustration" caption="IR Illustration" max-width="600" %}


这里，着重看虚线框框出来的三条指令，每条指令在内存中都由一个Instruction子类对象来表
示，由于Instruction是Value的子类，所以上面三条Instruction可以看做是三个Value，由于这个
IR是SSA格式的，所以每个虚拟寄存器都由唯一一条Instruction赋值，所以，虚拟寄存
器%4,%5,%6也都能唯一代表对应的Instruction。


Instruction也是User的子类，Instruction会使用其他的Value来作为自己的组成部分，比如，对于
红色框中%6寄存器对应的指令来说，它有两个操作数（operand），对应两个Value，一个是%4
对应的Instruction，一个是%5对应的Instruction，如上图紫色箭头指向。

查看涉及[Value](http://llvm.org/docs/ProgrammersManual.html#the-value-class)的编程文档，有这样一段话：

> A particular Value may be used many times in the LLVM representation for a program.
To keep track of this relationship, the Value class keeps a list of all of the User s that is
using it.

也就是说，每个Value可以被使用许多次，为了跟踪哪些User使用了自己，每个Value在其对象内
部都维持了一个User链表。以上面的例子来说，对于黄色虚线框中的指令来说，其User链表中包
含了蓝色框和红色框指令，因为蓝色框指令和红色框指令都使用了黄色框指令，即，都使用了%4
寄存器。

查看[Value.h](http://llvm.org/doxygen/Value_8h_source.html)，有这样一段话：

> Every value has a "use list" that keeps track of which other Values are using this Value.

```
class Value { 
   Type *VTy; // indicates what type the current instance is 
   Use *UseList; 
   ... 
}
```

看一下[Use.h](http://llvm.org/doxygen/Use_8h_source.html#l00055)中关于Use的定义
```
// A Use represents the edge between a Value definition and its users. 
class Use { 
   ... 
   Value *Val; // refer to one User instance that uses current Value instance 
   Use *Next; 
   ... 
} 
```

所以在内存中，use list类似于这种：

{% include image.html file="2019-10-07-llvm_ir_key_data_structure_introduction/user_list.png" url="" alt="User List" 
caption="User List" max-width="600" %}

接下来关注下Function和GlobalVariable类，把上图在下面再粘贴一下
{% include image.html file="2019-10-07-llvm_ir_key_data_structure_introduction/1.png" url="" alt="Value and Its Commonly-Used SubClasses" caption="Value and Its Commonly-Used SubClasses" max-width="600" %}

为啥全局变量（GlobalVariable）和函数（Function）是用Constant的子类，可能可以这样理
解，那就是全局变量和函数它们在链接后，它们的Value（即地址address）都是固定的，从这种
角度来说，是常量。

另外，由于全局变量和函数能够在不同函数间被访问到，所以它们是GlobalValue的子类也是比较
好理解的。

2. Type及常见子类

{% include image.html file="2019-10-07-llvm_ir_key_data_structure_introduction/type_and_subclass.png" url="" alt="User List" 
caption="User List" max-width="600" %}

每个Value都有一个Type，这个很好理解哈，值得注意的是，Type不是Value的子类哈。
稍微提一下[FunctionType](http://llvm.org/docs/ProgrammersManual.html#functiontype)，它为Function指定了形式化参数和返回值信息。一个FunctionType实
例可以被多个Function实例共用，当这些Function具有相同的参数类型和返回类型。

## Module、Function、BasicBlock以及Instruction在内存中的组织关系

其次介绍一下Module、Function、BasicBlock以及Instruction实例之间在内存中的组织关系

{% include image.html file="2019-10-07-llvm_ir_key_data_structure_introduction/relation.png" url="" alt="Organization In Memory" 
caption="Organization In Memory" max-width="400" %}

上图非常清晰的将这四个结构在LLVM中的组织方式展示了出来，可以看出，这些数据结构以某种
链表的形式进行了连接，LLVM中提供了iterator以及visitor等两种方式来遍历这些结构，在遍历的
过程中，可以通过代码来对这些结构进行增删改查等一系列操作，下一篇文章将依据本章的知识，
通过一些代码来演示如何操作LLVM IR。
其他未见内容可以参考资料[<sup>1</sup>](#refer-anchor-1)[<sup>2</sup>](#refer-anchor-2)[<sup>3</sup>](#refer-anchor-3)

## References

<div id="refer-anchor-1"></div>
[1]. [LLVM编程手册]( http://llvm.org/docs/ProgrammersManual.html)
<div id="refer-anchor-2"></div>
[2]. [LLVM Pass 编写指南](http://llvm.org/docs/WritingAnLLVMPass.html)
<div id="refer-anchor-3"></div>
[3]. [LLVM API文档](https://llvm.org/doxygen)
