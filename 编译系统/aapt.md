# aapt学习笔记

这篇文章主要介绍了Android快速分析apk工具aapt的使用教程,本文讲解了什么是aapt、主要用法、使用aapt、查看apk的基本信息、查看基本信息、查看应用权限等内容

## aapt简介
aapt即Android Asset Packaging Tool，我们可以在SDK的platform-tools目录下找到该工具。aapt可以查看、 创建、 更新ZIP格式的文档附件(zip, jar, apk)。 也可将资源文件编译成二进制文件，尽管你可能没有直接使用过aapt工具，但是build scripts和IDE插件会使用这个工具打包apk文件构成一个Android 应用程序。

  ### 主要用法

  下面的这个参数列表基本向我们展示了如何使用aapt以及aapt的基本功能了。

```
 aapt l[ist]:列出资源压缩包里的内容|
 ------------------------------------
 aapt d[ump]：查看APK包内指定的内容|
 -------------------------------------
 aapt p[ackage]：打包生成资源压缩包。
 ----------------------------------
 aapt r[emove]：从压缩包中删除指定文件。
 ------------------------------------
 aapt a[dd]：向压缩包中添加指定文件。
 -----------------------------------
 aapt v[ersion]:打印aapt的版本。
 ```

  * 使用aapt

  列举出apk中的所有文件

  ```java
  aapt l test.apk
  ```

      * 详细输出
      ```
      aapt l -v test.apk
      ```

  * 查看apk的基本信息

  aapt最实用的功能，通过d(ump)参数可以查看该apk的基本信息以及权限等
      * 查看APK中的配置信息，包括包名versionCode,versionName,platformBuildVersionName(编译的时候系统添加的字段，相当targetSdkVersionCode的描述)等。同事该命令还列出了manifest.xml的部分信息，包括启动界面，manifest里配置的label，icon等信息。还有分辨率，时区，uses-feature等信息。
      ```
      aapt d badging test.apk
      ```

      * 查看应用权限
```
  aapt d permissions test.apk
  ```
