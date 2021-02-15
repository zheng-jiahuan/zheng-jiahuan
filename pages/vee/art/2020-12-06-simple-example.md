---
title: 通过例子查看编译器编译流程
keywords: art, compiler
last_updated: Twelve 06, 2020
created: 2020-12-06
tags: [art,default]
summary: 通过例子查看编译器编译流程
sidebar: mydoc_sidebar
permalink: 2020-12-06-simple-example.html
folder: vee/art
---

## 简介

下面我们通过一个小例子来学习下编译器处理相关小知识

```java
package android.app;
public class CompilerHelper {
    /**{@hide}*/
    int zhenghanhan(int a, int b, int c){
        if (a > b) {
            return a;
        }

        switch(a){
            case 1:
                a += 1;
                break;
            case 2:
                a += 2;
                break;
            case 3:
                a += 3;
                break;
            case 4:
                a += 4;
                break;
            case 5:
                a += 5;
                break;
            case 6:
                a += 6;
                break;
            case 7:
                a += 7;
                break;
            case 8:
                a += 8;
                break;
            default:
                a = 9;
        }
        switch(b){
            case 1:
                b = 1;
                break;
            case 20:
                b = 2;
                break;
            case 300:
                b = 3;
                break;
            case 4000:
                b = 4;
                break;
            case 50000:
                b = 5;
                break;
            case 600000:
                b = 6;
                break;
            case 7000000:
                b = 7;
                break;
            case 8000000:
                b = 8;
                break;
            case 90000000:
                b = 9;
                break;
            default:
                b = 10;
        }

        int retVal = 0;

        try {
            retVal = (a + b) / c;
        } catch (Exception e) {
            System.out.println("exception occurs");
            e.printStackTrace();
            throw e;
        }
        return retVal;
    }
}

```
笔者在aosp R版本的frameworks/base/core/java目录下，新建了一个包名为android.app的CompilerHelper类，内容如上。
然后在ActivityThread.java中添加了如下这行，防止该类在编译的过程中，由于没有被使用的地方而被优化掉。
```java
public static void main(String[] args) {
         Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "ActivityThreadMain");
 
+        new CompilerHelper().zhenghanhan(1, 2, 3);
```

接着我们在R版本的根目录下执行 m -j12编译这个版本，发现有如下的报错：

```
--- frameworks/base/non-updatable-api/current.txt       2020-12-04 09:15:44.391197737 +0800
+++ out/soong/.intermediates/frameworks/base/api-stubs-docs-non-updatable/android_common/api-stubs-docs-non-updatable_api.txt   2020-12-06 12:27:29.380044847 +0800
@@ -1,4 +1,12 @@
 // Signature format: 2.0
+package  {
+
+  public class CompilerHelper {
+    ctor public CompilerHelper();
+  }
+
+}
+
 package android {
 
   public final class Manifest {
-e 
******************************
You have tried to change the API from what has been previously approved.

To make these errors go away, you have two choices:
   1. You can add '@hide' javadoc comments (and remove @SystemApi/@TestApi/etc)
      to the new methods, etc. shown in the above diff.

   2. You can update current.txt and/or removed.txt by executing the following command:
         make api-stubs-docs-non-updatable-update-current-api

      To submit the revised current.txt to the main Android repository,
      you will need approval.
******************************
```
原来，android为了限制随意的在库中添加接口，做了如上的限制，所以我们选择其中一个，比如
执行 make api-stubs-docs-non-updatable-update-current-api，然后current.txt文件会被更新，
此时我们连同current.txt文件一起，将该新增的java文件一同commit到库中，然后再次编译m -j12，此错误不再。

编译构建完成之后，我们可以在out/target/product/walleye/system/framework目录下找到framework.jar这个dex版本的jar包。
接着，我们可以执行dexdump -d命令，将这个dex中的代码输出出来看看。其中dexdump命令位于 out/host/linux-x86/bin目录下，我们将
framework.jar拷贝到该目录下，执行
```
./dexdump ./framework.jar -d > fwk.log
```
我们从fwk.log中查看CompilerHelper这个类的信息，得到的结果如下：

```
Class #341            -
  Class descriptor  : 'Landroid/app/CompilerHelper;'
  Access flags      : 0x0001 (PUBLIC)
  Superclass        : 'Ljava/lang/Object;'
  Interfaces        -
  Static fields     -
  Instance fields   -
    #0              : (in Landroid/app/CompilerHelper;)
      name          : 'cnt'
      type          : 'I'
      access        : 0x0000 ()
      hiddenapi     : 0x0002 (BLACKLIST)
  Direct methods    -
    #0              : (in Landroid/app/CompilerHelper;)
      name          : '<init>'
      type          : '()V'
      access        : 0x10001 (PUBLIC CONSTRUCTOR)
      hiddenapi     : 0x0010 (WHITELIST,TEST-API)
      code          -
      registers     : 2
      ins           : 1
      outs          : 1
      insns size    : 7 16-bit code units
1c1008:                                        |[1c1008] android.app.CompilerHelper.<init>:()V
1c1018: 7010 40fc 0100                         |0000: invoke-direct {v1}, Ljava/lang/Object;.<init>:()V // method@fc40
1c101e: 1200                                   |0003: const/4 v0, #int 0 // #0
1c1020: 5910 8221                              |0004: iput v0, v1, Landroid/app/CompilerHelper;.cnt:I // field@2182
1c1024: 0e00                                   |0006: return-void
      catches       : (none)
      positions     :
        0x0000 line=2
        0x0003 line=4
      locals        :
        0x0000 - 0x0007 reg=1 this Landroid/app/CompilerHelper;

  Virtual methods   -
    #0              : (in Landroid/app/CompilerHelper;)
      name          : 'zhenghanhan'
      type          : '(III)I'
      access        : 0x0000 ()
      hiddenapi     : 0x0002 (BLACKLIST)
      code          -
      registers     : 8
      ins           : 4
      outs          : 2
      insns size    : 160 16-bit code units
1c0e94:                                        |[1c0e94] android.app.CompilerHelper.zhenghanhan:(III)I
1c0ea4: 3765 0300                              |0000: if-le v5, v6, 0003 // +0003
1c0ea8: 0f05                                   |0002: return v5
1c0eaa: 2b05 5f00 0000                         |0003: packed-switch v5, 00000062 // +0000005f
1c0eb0: 1305 0b00                              |0006: const/16 v5, #int 11 // #b
1c0eb4: 2819                                   |0008: goto 0021 // +0019
1c0eb6: d805 050a                              |0009: add-int/lit8 v5, v5, #int 10 // #0a
1c0eba: 2816                                   |000b: goto 0021 // +0016
1c0ebc: d805 0509                              |000c: add-int/lit8 v5, v5, #int 9 // #09
1c0ec0: 2813                                   |000e: goto 0021 // +0013
1c0ec2: d805 0507                              |000f: add-int/lit8 v5, v5, #int 7 // #07
1c0ec6: 2810                                   |0011: goto 0021 // +0010
1c0ec8: d805 0506                              |0012: add-int/lit8 v5, v5, #int 6 // #06
1c0ecc: 280d                                   |0014: goto 0021 // +000d
1c0ece: d805 0505                              |0015: add-int/lit8 v5, v5, #int 5 // #05
1c0ed2: 280a                                   |0017: goto 0021 // +000a
1c0ed4: d805 0504                              |0018: add-int/lit8 v5, v5, #int 4 // #04
1c0ed8: 2807                                   |001a: goto 0021 // +0007
1c0eda: d805 0503                              |001b: add-int/lit8 v5, v5, #int 3 // #03
1c0ede: 2804                                   |001d: goto 0021 // +0004
1c0ee0: d805 0501                              |001e: add-int/lit8 v5, v5, #int 1 // #01
1c0ee4: 0000                                   |0020: nop // spacer
1c0ee6: 2c06 5900 0000                         |0021: sparse-switch v6, 0000007a // +00000059
1c0eec: 1306 0a00                              |0024: const/16 v6, #int 10 // #a
1c0ef0: 2815                                   |0026: goto 003b // +0015
1c0ef2: 1306 0900                              |0027: const/16 v6, #int 9 // #9
1c0ef6: 2812                                   |0029: goto 003b // +0012
1c0ef8: 1306 0800                              |002a: const/16 v6, #int 8 // #8
1c0efc: 280f                                   |002c: goto 003b // +000f
1c0efe: 1276                                   |002d: const/4 v6, #int 7 // #7
1c0f00: 280d                                   |002e: goto 003b // +000d
1c0f02: 1266                                   |002f: const/4 v6, #int 6 // #6
1c0f04: 280b                                   |0030: goto 003b // +000b
1c0f06: 1256                                   |0031: const/4 v6, #int 5 // #5
1c0f08: 2809                                   |0032: goto 003b // +0009
1c0f0a: 1246                                   |0033: const/4 v6, #int 4 // #4
1c0f0c: 2807                                   |0034: goto 003b // +0007
1c0f0e: 1236                                   |0035: const/4 v6, #int 3 // #3
1c0f10: 2805                                   |0036: goto 003b // +0005
1c0f12: 1226                                   |0037: const/4 v6, #int 2 // #2
1c0f14: 2803                                   |0038: goto 003b // +0003
1c0f16: 1216                                   |0039: const/4 v6, #int 1 // #1
1c0f18: 0000                                   |003a: nop // spacer
1c0f1a: 1200                                   |003b: const/4 v0, #int 0 // #0
1c0f1c: 9001 0506                              |003c: add-int v1, v5, v6
1c0f20: b371                                   |003e: div-int/2addr v1, v7
1c0f22: 6200 f984                              |003f: sget-object v0, Ljava/lang/System;.out:Ljava/io/PrintStream; // field@84f9
1c0f26: 1a02 f5b5                              |0041: const-string v2, "exception occurs" // string@b5f5
1c0f2a: 6e20 74fb 2000                         |0043: invoke-virtual {v0, v2}, Ljava/io/PrintStream;.println:(Ljava/lang/String;)V // method@fb74
1c0f30: 0000                                   |0046: nop // spacer
1c0f32: 1d04                                   |0047: monitor-enter v4
1c0f34: 5240 8221                              |0048: iget v0, v4, Landroid/app/CompilerHelper;.cnt:I // field@2182
1c0f38: b010                                   |004a: add-int/2addr v0, v1
1c0f3a: 5940 8221                              |004b: iput v0, v4, Landroid/app/CompilerHelper;.cnt:I // field@2182
1c0f3e: 1e04                                   |004d: monitor-exit v4
1c0f40: 0f01                                   |004e: return v1
1c0f42: 0d00                                   |004f: move-exception v0
1c0f44: 1e04                                   |0050: monitor-exit v4
1c0f46: 2700                                   |0051: throw v0
1c0f48: 0d01                                   |0052: move-exception v1
1c0f4a: 2807                                   |0053: goto 005a // +0007
1c0f4c: 0d01                                   |0054: move-exception v1
1c0f4e: 6e10 e4fb 0100                         |0055: invoke-virtual {v1}, Ljava/lang/Exception;.printStackTrace:()V // method@fbe4
1c0f54: 0000                                   |0058: nop // spacer
1c0f56: 2701                                   |0059: throw v1
1c0f58: 6202 f984                              |005a: sget-object v2, Ljava/lang/System;.out:Ljava/io/PrintStream; // field@84f9
1c0f5c: 1a03 f5b5                              |005c: const-string v3, "exception occurs" // string@b5f5
1c0f60: 6e20 74fb 3200                         |005e: invoke-virtual {v2, v3}, Ljava/io/PrintStream;.println:(Ljava/lang/String;)V // method@fb74
1c0f66: 2701                                   |0061: throw v1
1c0f68: 0001 0a00 0100 0000 1b00 0000 0300 ... |0062: packed-switch-data (24 units)
1c0f98: 0002 0900 0100 0000 1400 0000 2c01 ... |007a: sparse-switch-data (38 units)
      catches       : 3
        0x003e - 0x003f
          Ljava/lang/Exception; -> 0x0054
          <any> -> 0x0052
        0x0048 - 0x0051
          <any> -> 0x004f
        0x0055 - 0x005a
          <any> -> 0x0052
      positions     :
        0x0000 line=8
        0x0002 line=9
        0x0003 line=12
        0x0006 line=38
        0x0009 line=35
        0x000b line=36
        0x000c line=32
        0x000e line=33
        0x000f line=29
        0x0011 line=30
        0x0012 line=26
        0x0014 line=27
        0x0015 line=23
        0x0017 line=24
        0x0018 line=20
        0x001a line=21
        0x001b line=17
        0x001d line=18
        0x001e line=14
        0x0020 line=15
        0x0021 line=40
        0x0024 line=69
        0x0027 line=66
        0x0029 line=67
        0x002a line=63
        0x002c line=64
        0x002d line=60
        0x002e line=61
        0x002f line=57
        0x0030 line=58
        0x0031 line=54
        0x0032 line=55
        0x0033 line=51
        0x0034 line=52
        0x0035 line=48
        0x0036 line=49
        0x0037 line=45
        0x0038 line=46
        0x0039 line=42
        0x003a line=43
        0x003b line=72
        0x003c line=75
        0x003f line=80
        0x0046 line=81
        0x0047 line=83
        0x0048 line=84
        0x004d line=85
        0x004e line=87
        0x004f line=85
        0x0052 line=80
        0x0054 line=76
        0x0055 line=77
        0x0058 line=78
        0x005a line=80
        0x0061 line=81
      locals        :
        0x003c - 0x003f reg=0 retVal I
        0x003f - 0x0052 reg=1 retVal I
        0x0052 - 0x0059 reg=0 retVal I
        0x0000 - 0x0059 reg=4 this Landroid/app/CompilerHelper;
        0x0000 - 0x0059 reg=5 a I
        0x0000 - 0x0059 reg=6 b I
        0x0000 - 0x0059 reg=7 c I
        0x0055 - 0x005a reg=1 e Ljava/lang/Exception;
        0x005a - 0x00a0 reg=0 retVal I
        0x005a - 0x00a0 reg=4 this Landroid/app/CompilerHelper;
        0x005a - 0x00a0 reg=5 a I
        0x005a - 0x00a0 reg=6 b I
        0x005a - 0x00a0 reg=7 c I

  source_file_idx   : 8001 (CompilerHelper.java)
```
针对这两行，我们发现有省略号，dexdump工具没有完全把它显现出来

1c0f68: 0001 0a00 0100 0000 1b00 0000 0300 ... |0062: packed-switch-data (24 units)
1c0f98: 0002 0900 0100 0000 1400 0000 2c01 ... |007a: sparse-switch-data (38 units)

我们使用vim看相应的二进制（在本例子中，该二进制文件是framework.jar中的classes.dex），看看对应的数据：
其中，1c0f68: 0001 0a00 0100 0000 1b00 0000 0300 ... |0062: packed-switch-data (24 units)
对应的数据如下：
```
001c0f60:                     0001 0a00 0100 0000  n t.2.'.........
001c0f70: 1b00 0000 0300 0000 1800 0000 1500 0000  ................
001c0f80: 1200 0000 0f00 0000 0c00 0000 0300 0000  ................
001c0f90: 0900 0000 0600 0000                      ................
```

1c0f98: 0002 0900 0100 0000 1400 0000 2c01 ... |007a: sparse-switch-data (38 units)
展开后的数据如下：
```
001c0f90:                     0002 0900 0100 0000  ................
001c0fa0: 1400 0000 2c01 0000 a00f 0000 50c3 0000  ....,.......P...
001c0fb0: c027 0900 c0cf 6a00 0012 7a00 804a 5d05  .'....j...z..J].
001c0fc0: 1800 0000 1600 0000 1400 0000 1200 0000  ................
001c0fd0: 1000 0000 0e00 0000 0c00 0000 0900 0000  ................
001c0fe0: 0600 0000 
```








