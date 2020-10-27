[TOC]

# Android  Proguard


## 代码压缩(shrink code)

### 

### 代码测试

在<code>module</code>目录下的<code>build.gradle</code>文件内配置<code>minifyEnabled</code>值为<code>true</code>后，程序代码压缩功能就默认打开了，在打包<code>release</code>版本过程中，Android打包工具会源码进行压缩，移除其中不使用的类，变量，方法等，从而达到缩小最终APK体积的目的。

配置<code>minifyEnabled</code>值后体积大小对比如下图——第一张图<code>minifyEnabled=true</code>，第二张图<code>minifyEnabled=false</code>：

![minifyEnabled=true](https://github.com/sanren1024/knowledges/blob/main/android/images/Screenshot%20from%202020-10-27%2014-06-44.png) ![minifyEnabled=false](https://github.com/sanren1024/knowledges/blob/main/android/images/Screenshot%20from%202020-10-27%2014-09-05.png) 


<!--stackedit_data:
eyJoaXN0b3J5IjpbLTgwMzYxMzMyOCwtNDkzMzM0MjAyLDIwNz
A1NjMzNTRdfQ==
-->