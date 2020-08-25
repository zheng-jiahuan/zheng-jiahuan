---
title: LLVM IR 概览 - 续 - getelementptr
keywords: compiler, llvm ir
last_updated: October 06, 2019
created: 2019-10-06
tags: [compiler, llvm, default]
summary: "本文接着介绍LLVM IR中的一个比较特殊的操作-getelementptr"
sidebar: mydoc_sidebar
permalink: 2019-10-06-llvm_ir_overview_getelementptr.html
folder: compiler/llvm
---

我们经常能够在LLVM IR中发现getelementptr，那么这条指令是何作用？通过一个例子一起来看
一下

[例子1](https://llvm.org/docs/LangRef.html#getelementptr-instruction)

```
struct RT { 
  char A; 
  int B[10][20]; 
  char C; 
}; 
struct ST { 
  int X; 
  double Y; 
  struct RT Z; 
}; 
 
int *foo(struct ST *s) { 
  return &s[1].Z.B[5][13]; 
}
```
















