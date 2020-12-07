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
