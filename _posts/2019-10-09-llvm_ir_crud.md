---
title: LLVM IR-增删改查简介
keywords: compiler, llvm ir
last_updated: October 09, 2019
created: 2019-10-08
tags: [compiler, llvm, default]
summary: "本文简要介绍LLVM IR增删改查相关操作"
sidebar: mydoc_sidebar
permalink: 2019-10-09-llvm_ir_crud.html
folder: compiler/llvm
---

## 引言
本文使用两种方式来对IR进行操作，一个是生成一个单独的可执行文件来对IR进行操作，一个是写一个Pass，生成一个动态库，然后通过给clang或者opt添加参数选项来调用这个动态库，从而达到操作IR的目的。以下代码在LLVM 10上测试通过。

## 生成单独可执行文件来对IR进行操作

1.  首先给一个C语言写的例子

```c
#include <stdio.h>

int funb(int x, int y) {
  if (x > y) {
    return x - y;
  }
  return y -x;
}

int func(int x, int y, int z){
  x = y + z;
  y = x + z;
  z = x + y;
  return z;
}

int main() {
  printf("the result of funb(%d, %d) is %d\n", 2, 7, funb(2, 7));
  printf("the result of func(%d, %d, %d) is %d\n", 1, 4, 7, func(1, 4, 7));
  return 0;
}
```
在这个例子里面，写了三个函数，我们通过下面两个命令将这个例子转换成.ll文件

```c
clang hello.c -emit-llvm -S -Xclang -disable-O0-optnone
opt hello.ll  -mem2reg -S -o hello.m2r.ll
```
 我们就得到了这个例子对应的经过mem2reg优化过的IR文件，名字叫做hello.m2r.ll
 
 接下来，我们书写代码，来将这个.ll文件中包含一些信息打印出来。

```c 
#include "llvm/IR/Module.h"
#include "llvm/IRReader/IRReader.h"
#include "llvm/Support/SourceMgr.h"

using namespace llvm;

int main(int argc, char **argv) {
  if (argc < 2) {
    errs() << "Usage: " << argv[0] << " <path of IR file>\n";
    return 1;
  }

  SMDiagnostic Err;
  LLVMContext Context;
  // Parse the input LLVM IR file into a module.
  std::unique_ptr<Module> Mod(parseIRFile(argv[1], Err, Context));
  Module* M = Mod.get();

  if(M == nullptr) {
    errs() << "no module specified by argv[1] is spotted.\n";
    return 0;
  }

  errs() << "-------------------------------------------\n";
  for(auto& F : *M) {
    errs() << "Inspect details of Function " << F.getName() << "\n";
    // use itself print function to dump internals.
    F.print(errs(), nullptr);
    errs() << "-------------------------------------------\n";
  }

  return 0;
}
```
 上面这个代码是拿到了参数1指定的文件对应的Module对象，然后在这个Module对象中，依次获取其中的Function的名字，并将Function内容dump出来。这段代码放置在tutorial.cpp中，并使用如下命令编译：

 ```c
 clang++ -g tutorial.cpp `llvm-config --system-libs --cxxflags --ldflags  --libs all` -o tutorial
 ```
这样就生成了可执行文件tutorial，使用如下命令执行：
```c
./tutorial  ../hello/hello.m2r.ll
```
 因为tutorial 可执行文件从参数1中拿解析的IR文件路径，所以这里后面跟上的是hello.m2r.ll文件在笔者电脑上的路径，程序执行的输出为：

```c
-------------------------------------------
Inspect details of Function funb

; Function Attrs: noinline nounwind uwtable
define dso_local i32 @funb(i32 %0, i32 %1) #0 {
  %3 = icmp sgt i32 %0, %1
  br i1 %3, label %4, label %6

4:                                                ; preds = %2
  %5 = sub nsw i32 %0, %1
  br label %8

6:                                                ; preds = %2
  %7 = sub nsw i32 %1, %0
  br label %8

8:                                                ; preds = %6, %4
  %.0 = phi i32 [ %5, %4 ], [ %7, %6 ]
  ret i32 %.0
}
-------------------------------------------
Inspect details of Function func

; Function Attrs: noinline nounwind uwtable
define dso_local i32 @func(i32 %0, i32 %1, i32 %2) #0 {
  %4 = add nsw i32 %1, %2
  %5 = add nsw i32 %4, %2
  %6 = add nsw i32 %4, %5
  ret i32 %6
}
-------------------------------------------
Inspect details of Function main

; Function Attrs: noinline nounwind uwtable
define dso_local i32 @main() #0 {
  %1 = call i32 @funb(i32 2, i32 7)
  %2 = call i32 (i8*, ...) @printf(i8* getelementptr inbounds ([34 x i8], [34 x i8]* @.str, i64 0, i64 0), i32 2, i32 7, i32 %1)
  %3 = call i32 @func(i32 1, i32 4, i32 7)
  %4 = call i32 (i8*, ...) @printf(i8* getelementptr inbounds ([38 x i8], [38 x i8]* @.str.1, i64 0, i64 0), i32 1, i32 4, i32 7, i32 %3)
  ret i32 0
}
-------------------------------------------
Inspect details of Function printf

declare dso_local i32 @printf(i8*, ...) #1
-------------------------------------------
```
 呐，我们就打印出来了。注意一下，就是上面printf函数并不在tutorial.cpp中定义，所以该模块中，这个printf函数 只有声明（declare），没有定义（define）。
 
 接下来，我们演示另一个功能，我们希望tutorial.cpp中的代码能够将目标IR中的所有减法指令修改成加法指令。
 
 ```c
 #include "llvm/IR/Module.h"
#include "llvm/IRReader/IRReader.h"
#include "llvm/Support/SourceMgr.h"
#include "llvm/Support/raw_ostream.h"
#include "llvm/IR/IRBuilder.h"
#include "llvm/ADT/SmallVector.h"

using namespace llvm;

SMDiagnostic err;
LLVMContext context;

int main(int argc, char **argv) {
  if (argc < 2) {
    errs() << "Usage: " << argv[0] << " <path of IR file>  <path of output file>\n";
    return 1;
  }

  // Parse the input LLVM IR file into a module
  std::unique_ptr<Module> mod(parseIRFile(argv[1], err, context));
  Module* module = mod.get();

  if(module == nullptr) {
    errs() << "no module specified by argv[1] is spotted.\n";
    return 0;
  }

  //docs related to small vector -> http://llvm.org/docs/ProgrammersManual.html#llvm-adt-smallvector-h
  SmallVector<Instruction*, 10> targetInstVector;

  for (Function& fun : *module) {
    for (BasicBlock& bb : fun) {
      for (Instruction& inst : bb) {
        if (inst.getOpcode() == Instruction::Sub) {
           errs() << "\ninst detail : \n";
           inst.print(errs(), false);

           // collect the target inst
           targetInstVector.push_back(&inst);
        }
      }
    }
  }

  for (auto& instPos : targetInstVector) {
    // construct the new add instruction
    IRBuilder<> builder(instPos);
    Value *lhs = instPos->getOperand(0);
    Value *rhs = instPos->getOperand(1);
    Value *add = builder.CreateAdd(lhs, rhs);
  
    // obtain users and replace the old value with new one
    for(auto& use : instPos->uses()) {
      User* user = use.getUser();
      user->setOperand(use.getOperandNo(), add);
    }

    // erase the instruction from its containing basic block
    instPos->eraseFromParent();
  }

  // write the IR modified to file specified by parameter argv[2]
  std::error_code errCode;
  raw_fd_ostream out(argv[2], errCode);
  out << *module;
  return 0;
}
```
上面的代码应该很好懂，相关的概念比如User、Use、Value在上一篇文章中都有过解释。
编译tutorial.cpp

```c
 clang++ -g tutorial.cpp `llvm-config --system-libs --cxxflags --ldflags  --libs all` -o tutorial
```
运行tutorial可执行文件
```c
 ./tutorial  ../hello/hello.m2r.ll    ../hello/hello.m2r.sub2add.ll 
```

我们得到了修改后的hello.m2r.sub2add.ll，为节省篇幅，我只粘贴了有减法的funb在hello.m2r.sub2add.ll中的样子
 
```c
define dso_local i32 @funb(i32 %0, i32 %1) #0 {
  %3 = icmp sgt i32 %0, %1
  br i1 %3, label %4, label %6

4:                                                ; preds = %2
  %5 = add i32 %0, %1
  br label %8

6:                                                ; preds = %2
  %7 = add i32 %1, %0
  br label %8

8:                                                ; preds = %6, %4
  %.0 = phi i32 [ %5, %4 ], [ %7, %6 ]
  ret i32 %.0
}
```
 我们看到，原先的sub指令已经被新的add指令取代了。
 
 我们现在运行下hello.m2r.ll 和 hello.m2r.sub2add.ll 看看tutorial在修改IR前和修改IR后程序的执行结果
```c
lli hello.m2r.ll
``` 
```
the result of funb(2, 7) is 5
the result of func(1, 4, 7) is 29
```

```c
lli hello.m2r.sub2add.ll
```
```
the result of funb(2, 7) is 9
the result of func(1, 4, 7) is 29
```
可见，我们使用tutorial程序成功的修改了程序。


##  通过Pass来操作LLVM IR

上面我们说的，是通过编译出一个单独的可执行文件来对LLVM IR进行修改，现在要介绍的是如何将我们的IR修改动作内嵌到LLVM的Pass机制中，从而达到我们的目的。
 
在LLVM IR简介一文中，第一幅图中就绘制了Pass在整个编译过程的位置和角色。本文介绍的Pass依然是官方文档 中这种Pass，对于新近出现的New Pass机制（2012年开始的）这里暂不涉及，因为在当前这个阶段，PassManager框架的新旧并不是关键。
 
我们常见的Pass有 ModulePass、CallGraphSCCPass、FunctionPass、LoopPass、RegionPass、BasicBlockPass等。每种Pass都表明了其处理的单位是什么。比如FunctionPass，它作用于代码中的每个函数，是基于函数这个单位进行处理的。BasicBlockPass的处理单位是基本块，即，作用于代码中的每个基本块。
 
那处理的顺序是什么样的呢？举个例子，假设输入的程序有三个函数，分别叫做F、G、H，然后有两个FunctionPass，分别叫做X、Y，那么处理的流程是：X(F)Y(F) X(G)Y(G) X(H)Y(H)，不是 X(F)X(G)X(H) Y(F)Y(G)Y(H)，也就是说，一次处理一个函数，而不是一次处理一个Pass。
 
下面我将介绍下如何搭建书写Pass的环境，毕竟搭建环境是浪费时间而又没有太大意义的。参考Sampson教授搭建的Pass骨架库[<sup>2</sup>](#refer-anchor-2) ，我在上面做了点修改，使得在我的机器上能够正常编译。
 
首先在你的机器上将他的git库克隆下来（随便一个目录，只要你当时是全局安装LLVM的）
```c
 $ git clone https://github.com/sampsyo/llvm-pass-skeleton.git
```
 
 然后在follow他给的操作步骤之前，我们对拉下来的库中这个几个文件做下修改：
 
 对于文件 llvm-pass-skeleton/skeleton/CMakeLists.txt 修改成下面这个样子：
 
```c
add_library(SkeletonPass MODULE
    # List your source files here.
    Skeleton.cpp
)

set(CMAKE_CXX_FLAGS "-std=gnu++1y")

# LLVM is (typically) built with no C++ RTTI. We need to match that;
# otherwise, we'll get linker errors about missing RTTI data.
set_target_properties(SkeletonPass PROPERTIES
    COMPILE_FLAGS "-fno-rtti"
)
```

对于文件 llvm-pass-skeleton/skeleton/Skeleton.cpp 修改成了下面这个样子：
 
```c
#include "llvm/Pass.h"
#include "llvm/IR/Function.h"
#include "llvm/Support/raw_ostream.h"
#include "llvm/IR/LegacyPassManager.h"
#include "llvm/Transforms/IPO/PassManagerBuilder.h"
using namespace llvm;

namespace {
  struct SkeletonPass : public FunctionPass {
    static char ID;
    SkeletonPass() : FunctionPass(ID) {}

    virtual bool runOnFunction(Function &F) {
      errs() << "I saw a function called " << F.getName() << "!\n";
      return false;
    }
    virtual void getAnalysisUsage(AnalysisUsage &Info) {
        Info.setPreservesAll();
    }
  };
}

char SkeletonPass::ID = 0;

// Automatically enable the pass.
// http://adriansampson.net/blog/clangpass.html
static void registerSkeletonPass(const PassManagerBuilder &, legacy::PassManagerBase &PM) {
  PM.add(new SkeletonPass());
}
static RegisterStandardPasses
  RegisterMyPass(PassManagerBuilder::EP_EarlyAsPossible,
                 registerSkeletonPass);

// adding this line enable invocation from opt via option 
static RegisterPass<SkeletonPass> X("skeleton", "Skeleton Pass",
                             false /* Only looks at CFG */,
                             false /* Analysis Pass */);

```
好了，现在就可以回到llvm-pass-skeleton目录，执行如下命令了。 

```c
$ mkdir build 
$ cd build 
$ cmake .. 
$ make 
$ cd ..
```

 我们从上面的SkeletonPass可以看到，每个函数都会以Function &F的形式传入runOnFunction函数中，你可以在这里，从而对IR中的每个Function都进行操作，比如前面做的替换减法指令为加法指令。这我就不赘述了，代码是一样的。依据上面的这个骨架，你可以很快的将自己写的Pass编译出so出来，然后通过clang或者opt命令来对IR对象进行操作，我这里粘贴下两种方式执行的命令：
 
```c
clang -Xclang -load -Xclang llvm-pass-skeleton/build/skeleton/libSkeletonPass.so hello.c opt -disable-output  -load llvm-pass-skeleton/build/skeleton/libSkeletonPass.so -skeleton < hello.ll
```

在我这里，上面两个执行的结果都是：
 
```c
I saw a function called funb!
I saw a function called func!
I saw a function called main!
```


 clang的那个命令有一个好处就是，它的输入就是hello.c文件，而opt命令在处理的时候，你需要先用clang将hello.c转换成ll文件，然后再把自己的Pass作用在ll文件上，所以比较麻烦。
 
 如果您有什么建议和意见，欢迎留言，批评指正。
 
 参考资料[<sup>1</sup>](#refer-anchor-1)[<sup>2</sup>](#refer-anchor-2)[<sup>3</sup>](#refer-anchor-3)[<sup>4</sup>](#refer-anchor-4)[<sup>5</sup>](#refer-anchor-5)

## References
<div id="refer-anchor-1"></div>
[1]. [WritingAnLLVMPass](http://llvm.org/docs/WritingAnLLVMPass.html)
<div id="refer-anchor-2"></div>
[2]. [llvm-pass-skeleton](https://github.com/sampsyo/llvm-pass-skeleton)
<div id="refer-anchor-3"></div>
[3]. [CMU课件](http://www.cs.cmu.edu/afs/cs/academic/class/15745-s13/public/lectures/L3-LLVM-Overview.pdf)\
<div id="refer-anchor-4"></div>
[4]. [asampson blog](https://www.cs.cornell.edu/~asampson/blog/llvm.html)
<div id="refer-anchor-5"></div>
[5]. [clang pass](http://www.cs.cornell.edu/~asampson/blog/clangpass.html)