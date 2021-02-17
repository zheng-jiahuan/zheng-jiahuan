
---
title:  Dataflow Analysis中的Liveness Analysis
author: zjh
date: 2019-10-12
last_modified_at: 2021-02-17
categories: [compiler]
tags: [theory]
---
如果物品生产出来，最终却没有被使用，那么它就是没有价值的，因此这个物品也就没必要生产，原先生产这个物品过程中占用的资源也可以留作它用。

举两个例子
1. 亲戚来家里做客，然后爸妈准备下厨做饭，一口气做了18盘菜。饭毕，发现这18盘菜只有10盘菜被客人食用过，其他8盘菜原封未动，那如果早知道这8盘菜没人吃，那一开始不做就好了，从而节省人力、物力以及时间等。

2. 你告诉做饭机器人你饿了，让它随便炒个菜于是它开始做菜，首先将火腿和油分别从盘子和油桶里拿出来，放入锅里翻炒，然后再将锅里炒好的东西扔掉，接着放入牛肉和油到锅里翻炒，最后红烧牛肉出锅。你无语了，早知道是做红烧牛肉，前面炒火腿肠干嘛。

本文介绍的Liveness Analysis属于数据流分析（Data-flow analysis），数据流分析在参考文献[<sup>1</sup>](#refer-anchor-1)[<sup>2</sup>](#refer-anchor-2)中分别是这样定义的：
 
---
Data-flow analysis : a form of compile-time analysis for reasoning about the flow of values at runtime. 

Data-flow analysis derives information about the dynamic behavior of a program by only examining the static code

---

 也就是说，我们通过对静态的代码进行分析，考虑程序运行时候的所有可能，然后推断出关于程序中变量的Liveness的信息。Liveness分析是非常重要的，分析的结果可以用来
 1）发现未初始化变量 2）发现dead code 3）用于寄存器分配等。
 
 关于变量liveness的描述，我是这样理解的：
 
 给定一个变量v和一个位置p，我们说一个变量v在程序p处是活着的(**live**)，其实是说，当程序运行到p处的时候，此时变量v中存储的值是**有价值**的，即，这个值在程序运行到后面某个地方的时候，**有被用到的可能**。
 
 下面给出几个关于liveness的定义供参考，分别来自于参考文献[<sup>1</sup>](#refer-anchor-1)[<sup>3</sup>](#refer-anchor-3)[<sup>2</sup>](#refer-anchor-2)
 
---
A variable v is live at point p if and only if there exists a path in the cfg from p to a use of v along which v is not redefined.
     
1）A variable v is live at a program point p, if some path from p to program exit contains an r-value occurrence of v which is not preceded by an l-value occurrence of v. 2）A variable is live at a program point if its current value is **likely to be used later**.

A variable is live at a particular point in the program if its value at that point will be used **in the future** (dead, otherwise). To compute liveness at a given point, we need to **look into the future**.

---     

 注：上述define就是赋值(assign)的意思。
 
如果上面的定义不好懂，可以先绕过，我们看个例子[<sup>3</sup>](#refer-anchor-3)：

![](/assets/img/2019-10-12-liveness-analysis/1.png)
<center><strong>图1</strong></center>

问：上图中变量v在p点是否是live的？（假设p点到end点各个路径中所有和v相关的读写操作都已在图中显示）
 
答：变量v在p点的值是live的，因为**存在**一个分支（含a=v+2的这个分支），它使用了v在p点时候含有的值，也就是说变量v此时在p点处含有值是有价值的，将来有可能会被使用。需要注意的是，程序真正运行起来的时候，可能永远不会走a=v+2这个分支，可能一直都是走其他两个分支，但是由于我们是对程序的代码进行的静态分析，我们无法得知运行时的行为，所以，我们必须进行保守（conservative）分析，即假定三个分支都有可能走到，只要存在一处使用到v的地方，那么我们就将v在p处标记为live的。

![](/assets/img/2019-10-12-liveness-analysis/2.png)
<center><strong>图2</strong></center>

问：上图中变量v在p点是否是live的？（假设p点到end点各个路径中所有和v相关的读写操作都已在图中显示）
 
答：变量v在p点不是live的，因为变量v在p点持有的值是没有价值的，后面三个分支要么对其进行了重新赋值，那么根本就不涉及，即，v在p点持有的值在程序运行的时候没有被使用的可能，在p点持有的是无用的值，所以不是live的。

![](/assets/img/2019-10-12-liveness-analysis/3.png)
<center><strong>图3</strong></center>

问：上图中变量v在p点是否是live的？（假设p点到end点各个路径中所有和v相关的读写操作都已在图中显示）
 
答：变量v在p点是live的，因为存在一个分支（；含v=v+2的那个分支），使用了v在p点处持有的值（虽然使用这个值进行了+2，然后赋给了变量v），只要有使用的可能，那他就是有价值的，所以v在p点处是live的。
 
从上面的例子，我们可以清晰的看出，如果我们想知道程序运行到p处得时候，哪些变量所持有的值是有用的，那么我们必须看下哪些变量在此刻持有的值***在将来有可能被用到**，所以，为了算出p处live的变量集合，我们需要逆着数据流的方向倒过来算。这是一种backward的数据流分析。
 
看一个例子：

![](/assets/img/2019-10-12-liveness-analysis/4.png)
<center><strong>图4</strong></center>

以上图代码片段为例，分析的基本单位是基本块（BasicBlock），上图有三个基本块，分别为BB0,BB1和BB2，用三个黄框标示，数据流使用黄色箭头表示。一般地，需要我们通过liveness analysis来得出1,2,3,4,5,6这几个位置的live variable集合。我们来看下各个位置的值是如何计算出来的：
 
1处的live variable集合为空，因为在1处，不存在这样一个变量，该变量持有的值会在接下来的执行中可能被用到（函数已经返回了）。对于位置2处，即进入BB0基本块之前，这个位置的live variable集合含有哪些变量呢？有a，这是因为，当程序执行到2这个位置的时候，变量a含有的值在BB0基本块中会被用到，即return a指令中用到了。分析的时候，是倒过来分析的，数据流流向是从2->1,我们倒过来，在1->2穿过左下角基本块的时候，发现了a的值被使用了，所以2处的live variable集合包含了变量a。同理我们可以得出3处集合为空，4处集合含有变量b，表明程序执行到4处时，变量b含有的值是有意义的，在接下来的执行中有被使用到的可能（在这里一定会被使用到，因为BB1基本块只有一个return b指令）
 
对于位置5处，live variable集合包含了{a,b}两个变量，这个是由左边的{a} union 右边的{b}得出来的。当c>0的时候走左边分支，当c<=0的时候，走右边的分支。每个分支都有集合指明了哪些变量的值是在接下来的执行中可能有用的。所以，在位置5处，只要把左边的{a}和右边的{b}直接union就好了。也就是说，在程序执行到5处，变量a和变量b持有的值在下面的执行中是可能被用到的。
 
 ![](/assets/img/2019-10-12-liveness-analysis/5.png)
<center><strong>图5</strong></center>

现在我们接着逆流而上，5->6穿过BB2基本块，我们分析下，在被穿过的基本块中，有哪些动作。
 
第一个动作是d=d+1，它先使用了变量d中的值，+1后再赋给变量d，因为对d的使用在对d的赋值之前，所以，在位置6中，live variable集合中应该含有d，换句话说，当程序运行到6处的时候，变量d含有的值是有用处的，它在基本块中被d=d+1指令使用了。
 
第二个动作，a=c，这里c被使用了，所以在6处，live variable集合中应该含有变量c，因为6处，变量c存的值在接下来的代码中被使用了。对于a=c中的a来说，它被赋值了(赋值被翻译成define或者assign都对，一个意思），那么，程序运行到6处的时候，变量a含有的值是无用的，没有价值的，因为在经过基本块的时候，a中原先的值没有被使用就直接被赋值了。
 
第三个动作，c>0，这个操作是对c的读取操作，且基本块中，该语句的上几条语句并没有对齐有过赋值操作，所以程序运行到6处的时候，c变量含有的值是有意义的，是将来有用处的，所以6处的live variable集合中含有c。
 
综上，6处的live variable集合为 {a,b} - {a} + {d} + {c}，即{b,c,d}，也就是说，程序执行到6处的时候，b,c,d三个变量持有的值，是有价值的，有意义的，这些值在程序往后执行时是有被使用到的可能的。
 
在很多形式化定义中，1,3,5处的liveness variable集合被记做：LiveOut(BB0)、LiveOut(BB1)、LiveOut(BB2)。2,4,6处的liveness variable集合被记做：LiveIn(BB0)、LiveIn(BB1)、LiveIn(BB2)。
 
易知，**LiveOut(BB2) = LiveIn(BB0) ∪ LiveIn(BB1)**，因为BB0和BB1是BB2的successor。
 
那从LiveOut怎么得出LiveIn啊，这里需要引入两个概念[<sup>3</sup>](#refer-anchor-3)，Gen和Kill，
 
---
Gen : Use not preceded by definition //赋值之前被使用

Kill : Definition anywhere in a block   //在基本块中被赋值

---
这两个概念在上面分析的过程中已经隐含涉及了哈。

以BB2来说明，Gen(BB2) = {d,c} 而 Kill(BB2) = {a,d}，所以
 
LiveIn(BB2) = Gen(BB2) ∪ (LiveOut(BB2) - Kill(BB2)) = {d,c} + ({a,b} - {a,d}) = {b,d,c} 

在很多文献中，Gen集合中的变量也被称为Upwards Exposed Use变量。
 
例子中，BB2的LiveIn集合其实可以更加精简一下，可以看出，d变量其实是没有必要的，可以被精简掉，参考文献[<sup>3</sup>](#refer-anchor-3)中提到了strongly live variables analysis可以参考下哈。
 
另外，上面的这个例子分析是以基本块为单位的，你也可以以单个语句（single statement）为单位进行分析，流程都是一样的哈。
 
综上，进行一般性总结，我们知道计算一个节点n的LiveOut(n)，有如下公式

$$LiveOut(n) = \bigcup_{m\in succ(n)}LiveIn(m) $$

其中，m是n的successor成员。而计算一个节点m的LiveIn(n)，则有如下公式：

$$LiveIn(m) = Gen(m) \bigcup (LiveOut(m) - Kill(m))$$

我们可以将上面两个公式进行合并：

$$LiveOut(n) = \bigcup_{m\in succ(n)}(Gen(m) \bigcup (LiveOut(m) - Kill(m)))$$ 


 通过这个简单的例子，我们把live计算的大致思想过了一遍。由于这个例子中的CFG非常简单，不包含环，且我们直接从底部两个ret开始算，所以上面流程每个结点被遍历一遍就把每个结点的变量Live集合给算出来了。
 
 但是往往CFG包含有环等情况，所以我们在计算每个结点的live变量集合的时候，很多时候并不能一遍就把所有的结点的live变量计算出来。一般的做法是采用迭代策略（iterative strategy）进行计算，直至所有结点的live变量集合都不再变化为止，下面介绍下参考文献[<sup>4</sup>](#refer-anchor-4)中计算每个结点的LiveOut集合的算法，它以基本块为单位进行分析。
 
 我们知道，每个基本块的Gen集合和Kill集合都是不变的，所以一开始的时候，我们可以先把每个基本块的Gen集合和Kill集合都算出来，所以这里，这一步也叫做 **收集初始信息**（Gathering Initial Information）：
 
 ![](/assets/img/2019-10-12-liveness-analysis/6.png)
<center><strong>图6</strong></center>


图片来自下方参考文献[<sup>4</sup>](#refer-anchor-4)

算法里的UEVAR就是对应的我们上面说的Gen，而VARKILL对应的就是我们上面说的Kill哈，关于这两个集合，不同的教学材料里面使用的名字不同，但代表的意思都是一样的哈。
 
然后，依据上面我们推导的那个LiveOut等式，来求每个结点的LiveOut集合。
 
 ![](/assets/img/2019-10-12-liveness-analysis/7.png)
<center><strong>图7</strong></center>



图片来自下方参考文献[<sup>4</sup>](#refer-anchor-4)
 在计算之前，每个结点的LiveOut的初始值都是空集，然后不断的进行迭代，直至每个结点的集合都不再变化为止。
 
 这个算法肯定会终止的，不会一直循环执行下去，原因有两点，首先，程序中的变量的个数是有限的(finite)，其次，每次迭代，每个集合中变量的数量变化是单调的（monotonic）。对这个终止理论感兴趣的可以参考文献[<sup>5</sup>](#refer-anchor-5)。
 
 假设现在用上述算法来计算一个方法M中，各个基本块的live集合。方法M中含有x个变量，代号0到x-1，方法M中含有N个基本块，代号0到N-1。一般地，我们可以创建一个数组live[]，长度为N，对于任意一个数组元素live[n]来说，它含有x个bit位，每个bit位就代表一个变量对于基本块b来说，是否处于b的liveout集合中[<sup>6</sup>](#refer-anchor-6)。

## References
<div id="refer-anchor-1"></div>
[1]. [Engineering a Compiler 2nd Page 445]()
<div id="refer-anchor-2"></div>
[2]. [colostate CS553]( https://www.cs.colostate.edu/~mstrout/CS553/slides/lecture03.pdf)
<div id="refer-anchor-3"></div>
[3]. [WSSE17](https://isoft.acm.org/winterschool17/presentation-decks/WSSE17-Day1-2-Uday-talks/live-vars.pdf)
<div id="refer-anchor-4"></div>
[4]. [Engineering a Compiler 2nd Page 447]()
<div id="refer-anchor-5"></div>
[5]. [Knaster–Tarski theorem](https://en.wikipedia.org/wiki/Knaster%E2%80%93Tarski_theorem)
<div id="refer-anchor-6"></div>
[6]. [liveness analysis](https://www.cl.cam.ac.uk/teaching/2006/OptComp/slides/lecture03.pdf)
 