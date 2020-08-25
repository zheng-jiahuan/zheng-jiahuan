---
title: Dex2Oat基础
keywords: art, compiler
last_updated: Aug 20, 2020
created: 2020-08-20
tags: [art,default]
summary: 
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

























