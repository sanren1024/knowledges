
# Android  Proguard


## 代码压缩(code shrinking)

<code>R8</code>工具的代码压缩功能在配置<code>minifyEnabled</code>值为<code>true</code>后就默认打开了。

代码压缩(code shrinking)是<code>R8</code>工具移除在运行时不需要使用的代码过程。这个过程中<code>R8</code>移除不需要的类，变量，方法等。


### 原理

<code>R8</code>先根据配置的proguard文件(默认，或自定义)，分析确定代码的切入点。Android会依据这些切入点打开Activity或者Service。从每个入口点开始，<code>R8</code>会分析并构建包含类，变量，方法和其他在运行时可能访问到的类的图。而未分析到的类会被认定是**不可达**的，在后续打包过程中将被移除。

![tree-shaking](https://github.com/sanren1024/knowledges/blob/74b5e998da1b6dc241e83b54b222bb3c2595170f/android/images/proguard_tree-shaking.png)

在上图中显示了App运行时以来的库，<code>R8</code>在分析后将<code>MyActivity.class</code>作为入口，确定方法<code>foo()</code>，<code>faz()</code>，以及<code>AwesomeApi.class</code>的方法<code>bar()</code>是可达的。而<code>OkayApi.class</code>类是不可达的，因此在打包压缩过程中会被移除。

<code>R8</code>依据proguard文件内的<code>-keep</code>规则确认切入点。<code>-keep</code>规则指定的**class**文件是<code>R8</code>在压缩代码时不能移除的，并且将保留作为App的切入点。


### 测试

在<code>module</code>目录下的<code>build.gradle</code>文件内配置<code>minifyEnabled</code>值为<code>true</code>后，程序代码压缩功能就默认打开了，在打包<code>release</code>版本过程中，Android打包工具会源码进行压缩，移除其中不使用的类，变量，方法等，从而达到缩小最终APK体积的目的。

配置<code>minifyEnabled</code>值后体积大小对比如下图——第一张图<code>minifyEnabled=true</code>，第二张图<code>minifyEnabled=false</code>：

![minifyEnabled=true](https://github.com/sanren1024/knowledges/blob/main/android/images/Screenshot%20from%202020-10-27%2014-06-44.png) ![minifyEnabled=false](https://github.com/sanren1024/knowledges/blob/main/android/images/Screenshot%20from%202020-10-27%2014-09-05.png) 


### 自定义保留类

默认的ProGuard规则(<code>proguard-android-optimize.txt</code>)对<code>R8</code>在压缩代码过程中移除不需要的代码已经足够。

但也有<code>R8</code>会错误移除的个别情况：

- 调用JNI(Java Native Interface)接口； 
- 调用反射接口；

测试过程中可以揭露由于错误移除导致的错误，但是也可以通过配置产生一个report文件查看移除与保留的类。

怎么解决错误移除问题呢？ **使用<code>-keep</code>规则**，例如

```script
-keep public class MyClass
```

也可以使用<code>@Keep</code>标注解决上述问题。<code>@Keep</code>标注在类声明上面，该类会保持原有类名及内部结构，不会被压缩处理。

**注意**：使用<code>@Keep</code>，前提是使用*AndroidX Annotation Library*标注库。


## 资源压缩(Resource Shrinking)

资源压缩(Resource Shrinking)与代码压缩(Code Shrinking)一同进行。在代码压缩执行完成，移除无用代码后，资源压缩就也可以确定哪些资源是不再被使用的(反之，明确哪些资源是继续被使用的)。

通过在<code>module</code>目录下的<code>build.gradle</code>文件中设置<code>shrinkResources</code>为<code>true</code>，打开该功能，配置代码如下：

```groovy
android {
    ...
    buildTypes {
        release {
            shrinkResources true
            minifyEnabled true
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
}
```

再设置此项值前，确认是否设置了<code>minifyEnabled</code>，如还未设置，可以先设置这项来打开代码压缩功能。


### 自定义保留资源

若希望保留/丢弃某些特殊资源，可以在一个xml文件中进行配置。
xml文件中跟
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTg2MTgyMjE0OSwtODUxNDY2MzY0LDcyOD
YzMzM5NSwxMjc3NzQzMDY2LDE2NjYxMDk5MTIsLTg2OTc0MjEy
MywxMDgzNDY5OTkyLDg4NTQ2NDI1OCwtMTM0NDUzMjc4MywxND
E1MTI0OTA3LDIxMzMzNDc0NzIsLTQ5MzMzNDIwMiwyMDcwNTYz
MzU0XX0=
-->