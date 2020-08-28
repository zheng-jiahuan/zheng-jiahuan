---
title: Dex2Oat基础
keywords: art, compiler
last_updated: Aug 20, 2020
created: 2020-08-20
tags: [art,default]
summary: 本文简要介绍在编译AOSP的过程中，dex2oat的一些相关信息
sidebar: mydoc_sidebar
permalink: 2020-08-26-dex2oat_basics.html
folder: vee/art
---

## 简介

![build log](images/2020-08-26-dex2oat_basics/build-verbose.png)

在我们构建出来的out目录下，有一个叫做verbose.log.gz的文件，我们使用gzip -d verbose.log.gz
可以得到一个文本文件verbose.log，它保存了在构建整个系统的时候，详细的构建流程。

我们从这个log中截取构建boot.oat文件的构建命令

```
[63549/63992]
mkdir -p out/soong/walleye/dex_bootjars_unstripped/system/framework/arm && 
rm -f out/soong/walleye/dex_bootjars_unstripped/system/framework/arm/*.art out/soong/walleye/dex_bootjars_unstripped/system/framework/arm/*.oat out/soong/walleye/dex_bootjars_unstripped/system/framework/arm/*.invocation && 
rm -f out/soong/walleye/dex_bootjars/system/framework/arm/*.art out/soong/walleye/dex_bootjars/system/framework/arm/*.oat out/soong/walleye/dex_bootjars/system/framework/arm/*.invocation &&
 ANDROID_LOG_TAGS="*:e" 
 out/soong/host/linux-x86/bin/dex2oatd --avoid-storing-invocation --write-invocation-to=out/soong/walleye/dex_bootjars/system/framework/arm/boot.invocation --runtime-arg -Xms64m --runtime-arg -Xmx64m 
 --compiler-filter=speed-profile 
 --profile-file=out/soong/walleye/dex_bootjars/boot.prof 
 --dirty-image-objects=frameworks/base/config/dirty-image-objects 
 --dex-file=out/soong/walleye/dex_bootjars_input/core-oj.jar 
 --dex-file=out/soong/walleye/dex_bootjars_input/core-libart.jar 
 --dex-file=out/soong/walleye/dex_bootjars_input/okhttp.jar 
 --dex-file=out/soong/walleye/dex_bootjars_input/bouncycastle.jar 
 --dex-file=out/soong/walleye/dex_bootjars_input/apache-xml.jar 
 --dex-file=out/soong/walleye/dex_bootjars_input/framework.jar 
 --dex-file=out/soong/walleye/dex_bootjars_input/ext.jar 
 --dex-file=out/soong/walleye/dex_bootjars_input/telephony-common.jar 
 --dex-file=out/soong/walleye/dex_bootjars_input/voip-common.jar 
 --dex-file=out/soong/walleye/dex_bootjars_input/ims-common.jar 
 --dex-file=out/soong/walleye/dex_bootjars_input/android.test.base.jar 
 --dex-location=/apex/com.android.runtime/javalib/core-oj.jar 
 --dex-location=/apex/com.android.runtime/javalib/core-libart.jar 
 --dex-location=/apex/com.android.runtime/javalib/okhttp.jar 
 --dex-location=/apex/com.android.runtime/javalib/bouncycastle.jar 
 --dex-location=/apex/com.android.runtime/javalib/apache-xml.jar 
 --dex-location=/system/framework/framework.jar 
 --dex-location=/system/framework/ext.jar 
 --dex-location=/system/framework/telephony-common.jar 
 --dex-location=/system/framework/voip-common.jar 
 --dex-location=/system/framework/ims-common.jar 
 --dex-location=/system/framework/android.test.base.jar 
 --generate-debug-info 
 --generate-build-id 
 --oat-symbols=out/soong/walleye/dex_bootjars_unstripped/system/framework/arm/boot.oat 
 --strip 
 --oat-file=out/soong/walleye/dex_bootjars/system/framework/arm/boot.oat 
 --oat-location=out/soong/walleye/dex_bootjars/system/framework/boot.oat 
 --image=out/soong/walleye/dex_bootjars/system/framework/arm/boot.art 
 --base=0x70000000 
 --instruction-set=arm 
 --instruction-set-variant=cortex-a73 
 --instruction-set-features=default 
 --android-root=out/empty 
 --no-inline-from=core-oj.jar 
 --abort-on-hard-verifier-error 
 --generate-mini-debug-info || ( echo 'ERROR: Dex2oat failed to compile a boot image.It is likely that the boot classpath is inconsistent.Rebuild with ART_BOOT_IMAGE_EXTRA_ARGS="--runtime-arg -verbose:verifier" to see verification errors.' ; false )
 
 [63582/63992] 
 mkdir -p out/soong/walleye/dex_bootjars_unstripped/system/framework/arm64 && 
 rm -f out/soong/walleye/dex_bootjars_unstripped/system/framework/arm64/*.art out/soong/walleye/dex_bootjars_unstripped/system/framework/arm64/*.oat out/soong/walleye/dex_bootjars_unstripped/system/framework/arm64/*.invocation && 
 rm -f out/soong/walleye/dex_bootjars/system/framework/arm64/*.art out/soong/walleye/dex_bootjars/system/framework/arm64/*.oat out/soong/walleye/dex_bootjars/system/framework/arm64/*.invocation &&
ANDROID_LOG_TAGS="*:e" out/soong/host/linux-x86/bin/dex2oatd --avoid-storing-invocation 
--write-invocation-to=out/soong/walleye/dex_bootjars/system/framework/arm64/boot.invocation --runtime-arg -Xms64m --runtime-arg -Xmx64m 
--compiler-filter=speed-profile 
--profile-file=out/soong/walleye/dex_bootjars/boot.prof 
--dirty-image-objects=frameworks/base/config/dirty-image-objects 
--dex-file=out/soong/walleye/dex_bootjars_input/core-oj.jar 
--dex-file=out/soong/walleye/dex_bootjars_input/core-libart.jar 
--dex-file=out/soong/walleye/dex_bootjars_input/okhttp.jar 
--dex-file=out/soong/walleye/dex_bootjars_input/bouncycastle.jar 
--dex-file=out/soong/walleye/dex_bootjars_input/apache-xml.jar 
--dex-file=out/soong/walleye/dex_bootjars_input/framework.jar 
--dex-file=out/soong/walleye/dex_bootjars_input/ext.jar 
--dex-file=out/soong/walleye/dex_bootjars_input/telephony-common.jar 
--dex-file=out/soong/walleye/dex_bootjars_input/voip-common.jar 
--dex-file=out/soong/walleye/dex_bootjars_input/ims-common.jar 
--dex-file=out/soong/walleye/dex_bootjars_input/android.test.base.jar 
--dex-location=/apex/com.android.runtime/javalib/core-oj.jar 
--dex-location=/apex/com.android.runtime/javalib/core-libart.jar 
--dex-location=/apex/com.android.runtime/javalib/okhttp.jar 
--dex-location=/apex/com.android.runtime/javalib/bouncycastle.jar 
--dex-location=/apex/com.android.runtime/javalib/apache-xml.jar 
--dex-location=/system/framework/framework.jar 
--dex-location=/system/framework/ext.jar 
--dex-location=/system/framework/telephony-common.jar 
--dex-location=/system/framework/voip-common.jar 
--dex-location=/system/framework/ims-common.jar 
--dex-location=/system/framework/android.test.base.jar 
--generate-debug-info --generate-build-id 
--oat-symbols=out/soong/walleye/dex_bootjars_unstripped/system/framework/arm64/boot.oat 
--strip --oat-file=out/soong/walleye/dex_bootjars/system/framework/arm64/boot.oat 
--oat-location=out/soong/walleye/dex_bootjars/system/framework/boot.oat 
--image=out/soong/walleye/dex_bootjars/system/framework/arm64/boot.art --base=0x70000000 
--instruction-set=arm64 
--instruction-set-variant=cortex-a73 
--instruction-set-features=default 
--android-root=out/empty 
--no-inline-from=core-oj.jar 
--abort-on-hard-verifier-error 
--generate-mini-debug-info || ( echo 'ERROR: Dex2oat failed to compile a boot image.It is likely that the boot classpath is inconsistent.Rebuild with ART_BOOT_IMAGE_EXTRA_ARGS="--runtime-arg -verbose:verifier" to see verification errors.' ; false )
```
我们可以看到 out/soong/host/linux-x86/bin/dex2oatd编译boot.oat以及boot.art的命令，一个是arm 一个是arm64
这些文件刷机到手机中后，是放置在/system/framework/arm64和/system/framework/arm目录下的。

out/soong/host/linux-x86/bin/dex2oatd的编译选项可以参考[这个页面](./2020-08-26-dex2oat_options.html)

挑几个选项含义
```
--dex-file=<dex-file>: specifies a .dex, .jar, or .apk file to compile.
    Example: --dex-file=/system/framework/core.jar

--dex-location=<dex-location>: specifies an alternative dex location to
    encode in the oat file for the corresponding --dex-file argument.
    Example: --dex-file=/home/build/out/system/framework/core.jar
             --dex-location=/system/framework/core.jar
```
呐，对于core-oj.jar来说，

--dex-location=/apex/com.android.runtime/javalib/core-oj.jar 

--dex-file=out/soong/walleye/dex_bootjars_input/core-oj.jar 

它在服务器上被编译的时候，它的位置是out/soong/walleye/dex_bootjars_input/core-oj.jar，
但是这个jar它在手机上的位置信息是--dex-location=/apex/com.android.runtime/javalib/core-oj.jar
，从--dex-location编译选项的描述中可以得知，这个/apex/com.android.runtime/javalib/core-oj.jar
也会被编码（encode）进boot产物中。

我在dex2oat代码中加打印来着，然后编译aosp代码，想要看上面这个命令执行的过程中的一些地方的信息。
```
LOG(ERROR) << "error zheng-hanhan";
LOG(INFO) << "info zheng-hanhan";
```
发现INFO级别的Log打印不出来，ERROR可以打印出来，懒得去找调整输出日志级别的地方了，后面都使用ERROR这个级别的打印出来好了。

我在static dex2oat::ReturnCode Dex2oat(int argc, char** argv) 这个函数的下面这个地方加了打印
```
// Print the complete line when any of the following is true:
//   1) Debug build
//   2) Compiling an image
//   3) Compiling with --host
//   4) Compiling on the host (not a target build)
// Otherwise, print a stripped command line.
if (kIsDebugBuild || dex2oat->IsBootImage() || dex2oat->IsHost() || !kIsTargetBuild) {
  LOG(ERROR) << CommandLine();
  LOG(ERROR) << "zheng-hanhan " << "dex2oat->IsBootImage()" << dex2oat->IsBootImage()
            << "dex2oat->IsHost()" << dex2oat->IsHost() << "\n"; 
} else {
  LOG(ERROR) << StrippedCommandLine();
  LOG(ERROR) << "zheng-hanhan " << "StrippedCommandLine " << "\n"; 
}
```
我在verbose.log中查看日志，发现所有dex2oat执行，都没有搜索到StrippedCommandLine日志的出现，也就是说，
在aosp中编译的时候，并没有dex2oat命令执行的时候走到else这个代码中。
另外，我在if分支中的打印，只发现了如下两种打印
1. dex2oat->IsBootImage()0dex2oat->IsHost()0

2. dex2oat->IsBootImage()1dex2oat->IsHost()0

也就是说，在AOSP编译的过程中，没有dex2oat->IsHost()值是true。
在所有dex2oat执行的日志中，dex2oat->IsBootImage()值为true的地方，总共有4处。

第一处：

```
out/soong/host/linux-x86/bin/dex2oatd
--avoid-storing-invocation
--write-invocation-to=out/soong/walleye/dex_apexjars/system/framework/arm/apex.invocation
--runtime-arg
-Xms64m
--runtime-arg
-Xmx64m
--compiler-filter=speed-profile
--profile-file=out/soong/walleye/dex_bootjars/boot.prof
--dirty-image-objects=frameworks/base/config/dirty-image-objects
--dex-file=out/soong/walleye/dex_apexjars_input/core-oj.jar
--dex-file=out/soong/walleye/dex_apexjars_input/core-libart.jar
--dex-file=out/soong/walleye/dex_apexjars_input/okhttp.jar
--dex-file=out/soong/walleye/dex_apexjars_input/bouncycastle.jar
--dex-file=out/soong/walleye/dex_apexjars_input/apache-xml.jar
--dex-file=out/soong/walleye/dex_apexjars_input/framework.jar
--dex-file=out/soong/walleye/dex_apexjars_input/ext.jar
--dex-file=out/soong/walleye/dex_apexjars_input/telephony-common.jar
--dex-file=out/soong/walleye/dex_apexjars_input/voip-common.jar
--dex-file=out/soong/walleye/dex_apexjars_input/ims-common.jar
--dex-file=out/soong/walleye/dex_apexjars_input/android.test.base.jar
--dex-location=/apex/com.android.runtime/javalib/core-oj.jar
--dex-location=/apex/com.android.runtime/javalib/core-libart.jar
--dex-location=/apex/com.android.runtime/javalib/okhttp.jar
--dex-location=/apex/com.android.runtime/javalib/bouncycastle.jar
--dex-location=/apex/com.android.runtime/javalib/apache-xml.jar
--dex-location=/system/framework/framework.jar
--dex-location=/system/framework/ext.jar
--dex-location=/system/framework/telephony-common.jar
--dex-location=/system/framework/voip-common.jar
--dex-location=/system/framework/ims-common.jar
--dex-location=/system/framework/android.test.base.jar
--generate-debug-info
--generate-build-id
--oat-symbols=out/soong/walleye/dex_apexjars_unstripped/system/framework/arm/apex.oat
--strip
--oat-file=out/soong/walleye/dex_apexjars/system/framework/arm/apex.oat
--oat-location=out/soong/walleye/dex_apexjars/system/framework/apex.oat
--image=out/soong/walleye/dex_apexjars/system/framework/arm/apex.art
--base=0x70000000
--instruction-set=arm
--instruction-set-variant=cortex-a73
--instruction-set-features=default
--android-root=out/empty
--no-inline-from=core-oj.jar
--abort-on-hard-verifier-error
--generate-mini-debug-info
```

第二处：

```
out/soong/host/linux-x86/bin/dex2oatd
--avoid-storing-invocation
--write-invocation-to=out/soong/walleye/dex_apexjars/system/framework/arm64/apex.invocation
--runtime-arg
-Xms64m
--runtime-arg
-Xmx64m
--compiler-filter=speed-profile 
--profile-file=out/soong/walleye/dex_bootjars/boot.prof
--dirty-image-objects=frameworks/base/config/dirty-image-objects
--dex-file=out/soong/walleye/dex_apexjars_input/core-oj.jar
--dex-file=out/soong/walleye/dex_apexjars_input/core-libart.jar
--dex-file=out/soong/walleye/dex_apexjars_input/okhttp.jar
--dex-file=out/soong/walleye/dex_apexjars_input/bouncycastle.jar
--dex-file=out/soong/walleye/dex_apexjars_input/apache-xml.jar
--dex-file=out/soong/walleye/dex_apexjars_input/framework.jar
--dex-file=out/soong/walleye/dex_apexjars_input/ext.jar
--dex-file=out/soong/walleye/dex_apexjars_input/telephony-common.jar
--dex-file=out/soong/walleye/dex_apexjars_input/voip-common.jar
--dex-file=out/soong/walleye/dex_apexjars_input/ims-common.jar
--dex-file=out/soong/walleye/dex_apexjars_input/android.test.base.jar
--dex-location=/apex/com.android.runtime/javalib/core-oj.jar
--dex-location=/apex/com.android.runtime/javalib/core-libart.jar
--dex-location=/apex/com.android.runtime/javalib/okhttp.jar
--dex-location=/apex/com.android.runtime/javalib/bouncycastle.jar
--dex-location=/apex/com.android.runtime/javalib/apache-xml.jar
--dex-location=/system/framework/framework.jar
--dex-location=/system/framework/ext.jar
--dex-location=/system/framework/telephony-common.jar
--dex-location=/system/framework/voip-common.jar
--dex-location=/system/framework/ims-common.jar
--dex-location=/system/framework/android.test.base.jar
--generate-debug-info
--generate-build-id
--oat-symbols=out/soong/walleye/dex_apexjars_unstripped/system/framework/arm64/apex.oat
--strip
--oat-file=out/soong/walleye/dex_apexjars/system/framework/arm64/apex.oat
--oat-location=out/soong/walleye/dex_apexjars/system/framework/apex.oat
--image=out/soong/walleye/dex_apexjars/system/framework/arm64/apex.art
--base=0x70000000
--instruction-set=arm64
--instruction-set-variant=cortex-a73
--instruction-set-features=default
--android-root=out/empty
--no-inline-from=core-oj.jar
--abort-on-hard-verifier-error
--generate-mini-debug-info
```

第三处：
```
out/soong/host/linux-x86/bin/dex2oatd
--avoid-storing-invocation
--write-invocation-to=out/soong/walleye/dex_bootjars/system/framework/arm/boot.invocation
--runtime-arg
-Xms64m
--runtime-arg
-Xmx64m
--compiler-filter=speed-profile
--profile-file=out/soong/walleye/dex_bootjars/boot.prof
--dirty-image-objects=frameworks/base/config/dirty-image-objects
--dex-file=out/soong/walleye/dex_bootjars_input/core-oj.jar
--dex-file=out/soong/walleye/dex_bootjars_input/core-libart.jar
--dex-file=out/soong/walleye/dex_bootjars_input/okhttp.jar
--dex-file=out/soong/walleye/dex_bootjars_input/bouncycastle.jar
--dex-file=out/soong/walleye/dex_bootjars_input/apache-xml.jar
--dex-file=out/soong/walleye/dex_bootjars_input/framework.jar
--dex-file=out/soong/walleye/dex_bootjars_input/ext.jar
--dex-file=out/soong/walleye/dex_bootjars_input/telephony-common.jar
--dex-file=out/soong/walleye/dex_bootjars_input/voip-common.jar
--dex-file=out/soong/walleye/dex_bootjars_input/ims-common.jar
--dex-file=out/soong/walleye/dex_bootjars_input/android.test.base.jar
--dex-location=/apex/com.android.runtime/javalib/core-oj.jar
--dex-location=/apex/com.android.runtime/javalib/core-libart.jar
--dex-location=/apex/com.android.runtime/javalib/okhttp.jar
--dex-location=/apex/com.android.runtime/javalib/bouncycastle.jar
--dex-location=/apex/com.android.runtime/javalib/apache-xml.jar
--dex-location=/system/framework/framework.jar
--dex-location=/system/framework/ext.jar
--dex-location=/system/framework/telephony-common.jar
--dex-location=/system/framework/voip-common.jar
--dex-location=/system/framework/ims-common.jar
--dex-location=/system/framework/android.test.base.jar
--generate-debug-info
--generate-build-id
--oat-symbols=out/soong/walleye/dex_bootjars_unstripped/system/framework/arm/boot.oat
--strip
--oat-file=out/soong/walleye/dex_bootjars/system/framework/arm/boot.oat
--oat-location=out/soong/walleye/dex_bootjars/system/framework/boot.oat
--image=out/soong/walleye/dex_bootjars/system/framework/arm/boot.art
--base=0x70000000
--instruction-set=arm
--instruction-set-variant=cortex-a73
--instruction-set-features=default
--android-root=out/empty
--no-inline-from=core-oj.jar
--abort-on-hard-verifier-error
--generate-mini-debug-info
```

第4处：
```
out/soong/host/linux-x86/bin/dex2oatd
--avoid-storing-invocation
--write-invocation-to=out/soong/walleye/dex_bootjars/system/framework/arm64/boot.invocation
--runtime-arg
-Xms64m
--runtime-arg
-Xmx64m
--compiler-filter=speed-profile 
--profile-file=out/soong/walleye/dex_bootjars/boot.prof
--dirty-image-objects=frameworks/base/config/dirty-image-objects
--dex-file=out/soong/walleye/dex_bootjars_input/core-oj.jar
--dex-file=out/soong/walleye/dex_bootjars_input/core-libart.jar
--dex-file=out/soong/walleye/dex_bootjars_input/okhttp.jar
--dex-file=out/soong/walleye/dex_bootjars_input/bouncycastle.jar
--dex-file=out/soong/walleye/dex_bootjars_input/apache-xml.jar
--dex-file=out/soong/walleye/dex_bootjars_input/framework.jar
--dex-file=out/soong/walleye/dex_bootjars_input/ext.jar
--dex-file=out/soong/walleye/dex_bootjars_input/telephony-common.jar
--dex-file=out/soong/walleye/dex_bootjars_input/voip-common.jar
--dex-file=out/soong/walleye/dex_bootjars_input/ims-common.jar
--dex-file=out/soong/walleye/dex_bootjars_input/android.test.base.jar
--dex-location=/apex/com.android.runtime/javalib/core-oj.jar
--dex-location=/apex/com.android.runtime/javalib/core-libart.jar
--dex-location=/apex/com.android.runtime/javalib/okhttp.jar
--dex-location=/apex/com.android.runtime/javalib/bouncycastle.jar
--dex-location=/apex/com.android.runtime/javalib/apache-xml.jar
--dex-location=/system/framework/framework.jar
--dex-location=/system/framework/ext.jar
--dex-location=/system/framework/telephony-common.jar
--dex-location=/system/framework/voip-common.jar
--dex-location=/system/framework/ims-common.jar
--dex-location=/system/framework/android.test.base.jar
--generate-debug-info
--generate-build-id
--oat-symbols=out/soong/walleye/dex_bootjars_unstripped/system/framework/arm64/boot.oat
--strip
--oat-file=out/soong/walleye/dex_bootjars/system/framework/arm64/boot.oat
--oat-location=out/soong/walleye/dex_bootjars/system/framework/boot.oat
--image=out/soong/walleye/dex_bootjars/system/framework/arm64/boot.art
--base=0x70000000
--instruction-set=arm64
--instruction-set-variant=cortex-a73
--instruction-set-features=default
--android-root=out/empty
--no-inline-from=core-oj.jar
--abort-on-hard-verifier-error
--generate-mini-debug-info
```

这4处分别针对arm和arm64，这四处中--profile-file=out/soong/walleye/dex_bootjars/boot.prof是一样的。
我们对比下第一处和第三处这个arm的来分析下：
--profile-file的值是一样的，--dex-file看似不一样，比如一个
是out/soong/walleye/dex_bootjars_input/core-oj.jar，另一个中是 --dex-file=out/soong/walleye/dex_apexjars_input/core-oj.jar
，不同目录下的同名jar包。
我使用md5sum比较了下，发现他们的md5值是一样的，也就是说，这两个jar包是一样的。
那生成的oat文件一样吗？

--oat-file=out/soong/walleye/dex_apexjars/system/framework/arm/apex.oat

--oat-file=out/soong/walleye/dex_bootjars/system/framework/arm/boot.oat

上面这两个一样吗？我md5sum了一下，发现他们不一样，这就很奇怪了，怀疑是文件头写入了时间或者是文件名导致md5sum不一样，很可能内容是一样的。
所以我对他们分别作了

./out/host/linux-x86/bin/oatdump --oat-file=out/soong/walleye/dex_bootjars/system/framework/arm/boot.oat --header-only

./out/host/linux-x86/bin/oatdump --oat-file=out/soong/walleye/dex_apexjars/system/framework/arm/apex.oat --header-only

输出他们的文件头

```
AGIC:
oat
170

LOCATION:
out/soong/walleye/dex_bootjars/system/framework/arm/boot.oat

CHECKSUM:
0x57acb3eb

INSTRUCTION SET:
Thumb2

INSTRUCTION SET FEATURES:
div,atomic_ldrd_strd,armv8a

DEX FILE COUNT:
1

EXECUTABLE OFFSET:
0x000b5000

JNI DLSYM LOOKUP OFFSET:
0x000b5001

QUICK GENERIC JNI TRAMPOLINE OFFSET:
0x000b5009

QUICK IMT CONFLICT TRAMPOLINE OFFSET:
0x000b5011

QUICK RESOLUTION TRAMPOLINE OFFSET:
0x000b5019

QUICK TO INTERPRETER BRIDGE OFFSET:
0x000b5021

KEY VALUE STORE:
bootclasspath = /apex/com.android.runtime/javalib/core-oj.jar:/apex/com.android.runtime/javalib/core-libart.jar:/apex/com.android.runtime/javalib/okhttp.jar:/apex/com.android.runtime/javalib/bouncycastle.jar:/apex/com.android.runtime/javalib/apache-xml.jar:/system/framework/framework.jar:/system/framework/ext.jar:/system/framework/telephony-common.jar:/system/framework/voip-common.jar:/system/framework/ims-common.jar:/system/framework/android.test.base.jar
compiler-filter = speed-profile
concurrent-copying = true
debuggable = false
native-debuggable = false

SIZE:
2958834

.data.bimg.rel.ro: empty.

.bss: 18 methods, 242 GC roots.
```


另一个文件头


```
AGIC:
oat
170

LOCATION:
out/soong/walleye/dex_apexjars/system/framework/arm/apex.oat

CHECKSUM:
0x57acb3eb

INSTRUCTION SET:
Thumb2

INSTRUCTION SET FEATURES:
div,atomic_ldrd_strd,armv8a

DEX FILE COUNT:
1

EXECUTABLE OFFSET:
0x000b5000

JNI DLSYM LOOKUP OFFSET:
0x000b5001

QUICK GENERIC JNI TRAMPOLINE OFFSET:
0x000b5009

QUICK IMT CONFLICT TRAMPOLINE OFFSET:
0x000b5011

QUICK RESOLUTION TRAMPOLINE OFFSET:
0x000b5019

QUICK TO INTERPRETER BRIDGE OFFSET:
0x000b5021

KEY VALUE STORE:
bootclasspath = /apex/com.android.runtime/javalib/core-oj.jar:/apex/com.android.runtime/javalib/core-libart.jar:/apex/com.android.runtime/javalib/okhttp.jar:/apex/com.android.runtime/javalib/bouncycastle.jar:/apex/com.android.runtime/javalib/apache-xml.jar:/system/framework/framework.jar:/system/framework/ext.jar:/system/framework/telephony-common.jar:/system/framework/voip-common.jar:/system/framework/ims-common.jar:/system/framework/android.test.base.jar
compiler-filter = speed-profile
concurrent-copying = true
debuggable = false
native-debuggable = false

SIZE:
2958834

.data.bimg.rel.ro: empty.

.bss: 18 methods, 242 GC roots.
```

我们发现 LOCATION:这个内容不一致，其他的内容是一样的，文件头中记录的checksum是一样的。

所以，这两个oat文件除了文件头中记录的LOCATION内容不一致，其余内容都是一样的。

apex是谷歌用来更好管控android，具体的参阅官方文档。











