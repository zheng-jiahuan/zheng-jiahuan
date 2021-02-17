---
title:  Dataflow Analysis中的Available Expressions Analysis
author: zjh
date: 2019-10-13
last_modified_at: 2021-02-17
categories: [compiler]
tags: [theory]
---

先看下Available Expressions提出的动机和定义

---
**available expressions** is an analysis algorithm that determines for each point in the program 
the set of expressions that **need not be recomputed**. Those expressions are said to be **available** at
such a point. To be available on a program point, the operands of the expression should not be
modified on any path from the occurrence of that expression to the program point.

Programs may contain code whose result is needed, but in which some computation is **simply a redundant
repetition of earlier computation** within the same program
     
To identify **redundant** expressions, the compiler can compute information about the availability of 
expressions. An expression e is available at point p in a procedure if and only if on every path from
the procedure's entry to p, e is evaluated and none of its constituent subexpressions is 
redefined between that evaluation and p. 

---

可以看出，Available Expressions是为了找出冗余计算的，给定你一个程序中的位置，你要告诉我，在这个位置时，计算哪些表达式是不必要的，
因为这个表达式之前已经计算过，其产生的值和在这个位置上再次计算的值是一样的，那么这里就有优化的可能。
Available Expressions常常用来消除公共子表达式（eliminate common sub expressions）。
 
看例子
 ![](/assets/img/2019-10-13-available-expression-analysis/1.png)

<center><strong>图1</strong></center>

在图1中，y * z在第一个蓝色框中被计算了一次，在第二个蓝色框中，y * z又被计算了一次，很容易看出，第二次计算y  *  z是冗余的，程序执行到第一个y * z的时候，其所得的值和程序执行到第二个y * z的时候所得的值是相同的。识别到了这个特性，那么我们就有机会对代码进行优化，进而节省计算。

![](/assets/img/2019-10-13-available-expression-analysis/2.png)

<center><strong>图2</strong></center>

在图2中，第一个y * z计算出来的值，对于return语句来说其值是无效的，不可用的，因为中间经过了y = a+b;这条语句时，y被重新赋值了，当程序执行到第二个y * z时，这个表达式算出来的值和之前y * z算出来的值不能保证一样，那对于return语句来说，在进入这条语句之前，可用表达式集合不包含y * z。

![](/assets/img/2019-10-13-available-expression-analysis/3.png)

<center><strong>图3</strong></center>

先说明下，BB是basic block的缩写。图3中，在进入BB1之前，可用表达式集合是空的，接着，BB1中产生了（generate）一个表达式x * y，离开1号基本块的时候，这个表达式的值依然是可用的，也就是说，如果此时，紧接着BB1出口有一个x * y的表达式，那么这个表达式的计算属于冗余计算。
 
由于BB2只有BB1这一个predecessor，所以基本块1出口处可用表达式集合和基本块2入口处可用表达式集合是一样的，都是{x * y}，另外，BB2中并没有产生新的表达式（Generate），也没有使得某些表达式不可用（KILL），所以，随着程序顺着数据流流出BB2，其可用表达式集合依然为{x * y}，接着，程序分两个方向走，一个是BB2 -> BB3，一个是BB2 -> BB4。但是不论走了哪个方向，可用表达式集合都为{x * y}没有变。现在问题是，图中，在进入BB5之前，9号问号处，可用表达式是什么？显然，不论是走红色虚线分支还是棕色虚线分支，在9号位置时，都有可用的表达式x * y，也就是说，如果9号处出现了表达式x * y，那么该处表达式的计算是冗余的，因为前方（不论哪个分支上）有表达式x * y的结果和现在你将要计算的x * y的值是一样的，所以在9处，可用表达式集合是包含x * y的，BB5中一开始就遇到了x * y表达式，我们可知，此处return语句中的x * y计算是冗余的。
 
![](/assets/img/2019-10-13-available-expression-analysis/4.png)

<center><strong>图4</strong></center>

我们看下图4，图4相对于图3，修改了BB3，在进入BB3之前，可用表达式集合包含了一个表达式x * y，在通过BB3的时候发生了这样的变化，首先是x=5，由于修改了x的值，所以可用表达式集合中凡是含有x的表达式，都会被这条指令去掉（KILL），所以紧接着x=5这条指令，可用表达式集合是空集，接着看BB3中的第二条语句 t=x * y，这条语句产生了表达式x * y，所以在离开t=x * y的时候，或者说，在离开BB3的时候，即在位置6处，可用表达式的集合为{x * y}，也就是说，如果在t=x * y后面紧接着出现了x * y，那么这个紧接着出现的x * y的计算是**冗余的**，因为在t=x * y中x * y已经算了一遍了，其值和你现在重算一遍x * y的值是一样的。8处仍为{x * y}。同理，9处在进入BB5之前，可用表达式集合依然为{x * y}，因为1）如果程序按照红色虚线执行过来，在9处如果出现x * y，那么和t=x * y中的x * y值是一样的，2）如果从棕色虚线执行过来，在位置9处如果出现x * y，那么和BB1中x * y计算的值是一样的，不论哪条执行路径，此处出现x * y的计算都是**冗余**的。所以9处的可用表达式集合包含x * y。

![](/assets/img/2019-10-13-available-expression-analysis/5.png)

<center><strong>图5</strong></center>

 图5中，BB4也做了修改，BB4和BB3的情况是一样的，也很容易得知，位置9处可用表达式集合也是{x * y}。

![](/assets/img/2019-10-13-available-expression-analysis/6.png)

<center><strong>图6</strong></center>
我们看下图6，BB4中x=3将位置7处可用表达式集合中含有x的表达式都去掉了(KILL)，所以到离开BB4的时候，可用表达式为{}。那么此时位置9处，进入BB5之前，可用表达式集合中的可用表达式有哪些？答案是{}，因为，如果程序走红色虚线过来，那么再次计算x * y是冗余的，但是如果程序走棕色虚线过来，再次计算x * y并不是冗余的。我们现在是在进行静态分析，不能预知程序在运行的时候到底走哪条路线，所以9处的表达式集合是{}，换句话说，如果此时在9处出现x * y表达式，我们不能说它是冗余计算，因为程序可能从棕色虚线执行过来。

我们前面在介绍Liveness Analysis的时候，知道Liveness Analysis属于backwards data-flow analysis，它使用future指令所包含的信息来指导当前。而这里的Available Expressions Analysis是forwards data-flow analysis，这是因为，它使用past指令所包含的信息来指导当前。
 
如一开始所说，我们给定一个位置p，你要告诉我，在这个p位置上计算哪些表达式是冗余的，我拿到这些信息之后，好做代码优化。一般地，给定一个CFG，我们需要计算出每个基本块的入口和出口处的可用表达式集合。
 
我们先约定AvailIn(n)代表基本块n的入口位置的可用表达式集合，AvailOut(n)代表基本块n的出口位置的可用表达式集合。
 
和之前进行liveness分析一样，我们也引入两个概念：

---
Gen :  a set of downward exposed expressions Kill :  all those expressions that are "killed" by a definition

---

 Gen(n)代表的是这样一个表达式集合，这个集合中的表达式都是由基本块n中产生的，且从这个表达式产生处到这个基本块n的出口处，表达式的各个组成部分没有被redefine过。比如图6中BB3的t=x * y这个语句产生了x * y这个表达式，从该语句位置到BB3出口处，x和y没有被重新赋值过。所以Gen(BB3) = {x * y}。
 
 Kill(n)代表的是这样一个表达式集合，这个集合中的表达式的构成元素在基本块n中被redefine过。以图6中BB3为例，x = 3语句将x进行了redefine，所以凡是包含x的表达式都归类在Kill(n)中，这里Kill(BB3) = {x * y}。
 
 AvailOut(BB3) = (AvailIn(BB3) - Kill(BB3)) U Gen(BB3)
 
 所以AvailOut(BB3) = {x * y}。
 
 图6中AvailIn(BB5)怎么求？根据上面几个例子的详细描述，我们其实知道，
 
 $$AvailIn(BB5) = AvailOut(BB3)  \cap\cap  AvailOut(BB4)$$
 
 接下来我们队上面的公式进行泛化
 
$$AvailOut(m) = Gen(m) \bigcup (AvailIn(m) - Kill(m))$$

$$AvailIn(n) = \bigcap_{m\in pred(n)}AvailOut(m) $$

也可以合并下，得到如下公式：

$$AvailIn(n) = \bigcap_{m\in pred(n)}(Gen(m) \bigcup (AvailIn(m) - Kill(m)))$$

然后，公式得到了，在进行迭代计算之前，需要给每个节点的AvailIn附上初始值。
 
对于ENTRY来说，其AvailIn(ENTRY) = {}，即空集，对于其他的节点n，其AvailIn(ENTRY) = {all expressions}，注意，这个all expressions一般是指一个待优化函数中出现的所有表达式。
 
上述都完成了，就可以进行类似于前文liveness analysis的迭代计算算法就可以了。
 
针对上图5，不论BB3->BB5还是BB4->BB5，x * y都会被计算两次，我们可能希望进行这样的改造：

 ![](/assets/img/2019-10-13-available-expression-analysis/7.png)

<center><strong>图7</strong></center>
这样改造后，不论BB3->BB5还是BB4->BB5，x * y只会被计算一次。不过却多了两条拷贝指令，并且还引入了一个新的寄存器v，如果目标寄存器数量不够用了，那么还会造成寄存器spill，一旦发生spill，那么对内存的store和load很可能比重复计算表达式更加昂贵。所以是否改造后变快了还要结合目标体系结构来看。


## References
<div id="refer-anchor-1"></div>
[1]. [wiki available expression ](https://en.wikipedia.org/wiki/Available_expression)
<div id="refer-anchor-2"></div>
[2]. [rochester lecture05 available ](https://www.cs.rochester.edu/u/sree/courses/csc-255-455/spring-2019/static/05-avail.pdf)
<div id="refer-anchor-3"></div>
[3]. [Engineering a Compiler 2nd Page 490]() 