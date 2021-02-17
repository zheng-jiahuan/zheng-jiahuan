---
title:  Dataflow Analysis中的Anticipable Expressions Analysis
author: zjh
date: 2019-10-14
last_modified_at: 2021-02-16
categories: [compiler]
tags: [theory]
---

本文介绍Dataflow Analysis中的Anticipable Expressions Analysis
 
## 定义
 
首先，先给出两个关于Anticipable Expressions的定义[<sup>1</sup>](#refer-anchor-1)[<sup>2</sup>](#refer-anchor-2)

---
An expression e is considered anticipable, or very busy, on exit from block b if and only if 
(1) every path that leaves b evaluates and subsequently uses e, and (2) evaluating e at the end
of b would produce the same result as the first evaluation of e along each of those paths.

An expression A op B is very busy at a point p if along every path from p there is an expression A op B 
before a redefinition of either A or B.

---

Anticipable Expressions 也叫 Very Busy Expressions，该分析可用于code movement optimization 
 
## 分析
 
同样地，我们通过例子来学习这个概念 
 
![图1](/assets/img/2019-10-14-anticipable-expressions-analysis/1.png)

<center><strong>图1</strong></center>

对于图1中展示的例子来说，我们可能想要对其进行如下的改造： 
 
![图2](/assets/img/2019-10-14-anticipable-expressions-analysis/2.png)

<center><strong>图2</strong></center>

图2中的几个转换不会改变程序的原有行为。
 
上面这个改造（transformation）的过程就是将某些表达式的计算提前。上图中，v = x + y再往前到y = 10就不可以了，因为y = 10会去掉(KILL)
所有含有y的表达式。如果v = x + y移动到了y = 10上面，那么程序执行结果就和原先不一样了（除非程序执行时y的值在y = 10之前就是10）。
 
这个过程也叫做code hoisting（代码上提？），一般地，结合其他的优化分析比如copy propagation，可以将code hoisting产生的赋值语句
u = v 以及 t = v优化掉，进而可以缩减code size（如下图）。另外，在有循环的场景下，如果循环里面的表达式可以通过code hoisting提出循环外，
则可以提升程序性能。
 
![图3](/assets/img/2019-10-14-anticipable-expressions-analysis/3.png)

<center><strong>图3</strong></center>

Anticipable Expressions Analysis是一种向后（backwards）的数据流分析，也就是逆着数据流的方向，如下图的红色和绿色箭头。

![图4](/assets/img/2019-10-14-anticipable-expressions-analysis/4.png)

<center><strong>图4</strong></center>

同样地，我们选择基本块的前后位置，来求Anticipable Expressions集合。
 
图4中，1处是{}，在经过return语句的时候产生（GEN）了表达式w + z，所以在2处，集合变成了{w+z}，3和5两处和2一样，然后经过BB2和BB3的时候，w和z分别在两个基本块中各自kill了表达式w+z，但是产生了x + y表达式，所以4,6两处的Anticipable Expressions都是{x + y}。
 
加入到集合中的表达式都有这样一个特点，就像前面说的，这个表达式可以被提前到前面进行计算，其计算结果和此处表达式计算是一样的。对于7处来说，我们取4,6的交集。所以7处仍然是{x + y},经过BB1的时候，由于y被redefine了，那么7处中所有含有y的表达式都会被去掉(KILL)，所以8处的Anticipable Expressions集合为{}。
 

![图5](/assets/img/2019-10-14-anticipable-expressions-analysis/5.png)

<center><strong>图5</strong></center>

对于图5来说，我们改了BB2的代码，加了一个x = 3,这条语句使得我们不能将表达式x+y加入到4处的Anticipable Expressions集合中，因为在4处假设我插入了一条语句来计算x + y，计算出的值后面是没人用的，会被x = 3这条语句invalidate掉，没啥意义了。对于7处，4和6处Anticipable Expressions集合取交集，所以就变成了{} 。因为我如果在7处加一条x+y的计算，并不能将计算的值用于将来最近的一条x+y的值（当程序走BB1->BB2分支的时候）。
 
## 形式化公式
 
  和前面的文章中的分析一样，我们先引入两个概念

---
 Gen :  a set of upward exposed expressions Kill :  all those expressions that are "killed" by a definition

---


Gen(n)代表的是这样一个表达式集合，这个集合中的表达式都是由基本块n中产生的，且从这个表达式产生处到这个基本块n的入口处，表达式的各个组成部分没有被redefine过，所以这个也叫做Upward-Exposed Expressions集合。像图5中的BB2中，x+y就不属于Gen(BB2)集合，但是属于Kill集合，因为x = 3这条指令。 总结下，得出如下公式： 
 
$$ AntOut(n) = \bigcap_{m\subseteq succ(n)}(AntIn(m)) $$

$$ AntIn(m) = Gen(m) \bigcap (AntOut(m) - Kill(m)) $$
 
合并可以得出： 
 
$$ AntOut(n) = \bigcap_{m\in succ(n)}(Gen(m) \bigcap (AntOut(m) - Kill(m)))$$

注：一开始时，各个集合的初始值为：
 
$$ AntOut(n_{f}) = \oslash  ( n_{f} 是exit node)$$
 
$$AntOut(n) = \left\{ all\ expressions \right\} , \forall n \ne n_{f} $$

上述步骤采用的也是类似于我们在liveness analysis中的迭代的算法来进行计算。 参考文献[<sup>1</sup>](#refer-anchor-1)[<sup>2</sup>](#refer-anchor-2)[<sup>3</sup>](#refer-anchor-3)


## References
<div id="refer-anchor-1"></div>
[1]. [Engineering a Compiler P491: Anticipable Expressions]()
<div id="refer-anchor-2"></div>
[2]. [Very Busy Expressions](https://web.cs.wpi.edu/~kal/PLT/PLT9.6.html)
<div id="refer-anchor-3"></div>
[3]. [Dataflow Analysis](https://www.geeksforgeeks.org/data-flow-analysis-compiler/) 