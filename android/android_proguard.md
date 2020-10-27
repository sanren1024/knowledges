[TOC]

# Android  Proguard


## 代码压缩(code shrinking)

<code>R8</code>工具的代码压缩功能在配置<code>minifyEnabled</code>值为<code>true</code>后就默认打开了。

代码压缩(code shrinking)是<code>R8</code>工具移除在运行时不需要使用的代码过程。这个过程中<code>R8</code>移除不需要的类，变量，方法等。

### 原理

<code>R8</code>先根据配置的proguard文件(可以默认，或自定义)，确定代码中的


### 代码测试

在<code>module</code>目录下的<code>build.gradle</code>文件内配置<code>minifyEnabled</code>值为<code>true</code>后，程序代码压缩功能就默认打开了，在打包<code>release</code>版本过程中，Android打包工具会源码进行压缩，移除其中不使用的类，变量，方法等，从而达到缩小最终APK体积的目的。

配置<code>minifyEnabled</code>值后体积大小对比如下图——第一张图<code>minifyEnabled=true</code>，第二张图<code>minifyEnabled=false</code>：

![minifyEnabled=true](https://github.com/sanren1024/knowledges/blob/main/android/images/Screenshot%20from%202020-10-27%2014-06-44.png) ![minifyEnabled=false](https://github.com/sanren1024/knowledges/blob/main/android/images/Screenshot%20from%202020-10-27%2014-09-05.png) 

<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE1NDE2MDUwOSwyMTMzMzQ3NDcyLC00OT
MzMzQyMDIsMjA3MDU2MzM1NF19
-->