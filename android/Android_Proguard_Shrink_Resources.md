
# Android  Proguard

文章内容有些长，包含了测试内容，但是读完我相信对apk体积压缩会有一个更好的认识。

## 代码压缩(code shrinking)

<code>R8</code>工具的代码压缩功能在配置<code>minifyEnabled</code>值为<code>true</code>后就默认打开了。

代码压缩(code shrinking)是<code>R8</code>工具移除在运行时不需要使用的代码过程。这个过程中<code>R8</code>移除不需要的类，变量，方法等。


### 原理

<code>R8</code>先根据配置的proguard文件(默认，或自定义)，分析确定代码的切入点。Android会依据这些切入点打开Activity或者Service。从每个入口点开始，<code>R8</code>会分析并构建包含类，变量，方法和其他在运行时可能访问到的类的图。而未分析到的类会被认定是**不可达**的，在后续打包过程中将被移除。

![tree-shaking](https://github.com/sanren1024/knowledges/blob/main/android/images/proguard/proguard_tree-shaking.png)

在上图中显示了App运行时以来的库，<code>R8</code>在分析后将<code>MyActivity.class</code>作为入口，确定方法<code>foo()</code>，<code>faz()</code>，以及<code>AwesomeApi.class</code>的方法<code>bar()</code>是可达的。而<code>OkayApi.class</code>类是不可达的，因此在打包压缩过程中会被移除。

<code>R8</code>依据proguard文件内的<code>-keep</code>规则确认切入点。<code>-keep</code>规则指定的**class**文件是<code>R8</code>在压缩代码时不能移除的，并且将保留作为App的切入点。


### 测试

在<code>module</code>目录下的<code>build.gradle</code>文件内配置<code>minifyEnabled</code>值为<code>true</code>后，程序代码压缩功能就默认打开了，在打包<code>release</code>版本过程中，Android打包工具会源码进行压缩，移除其中不使用的类，变量，方法等，从而达到缩小最终APK体积的目的。

配置<code>minifyEnabled</code>值后体积大小对比如下图——第一张图<code>minifyEnabled=true</code>，第二张图<code>minifyEnabled=false</code>：

![minifyEnabled=true](https://github.com/sanren1024/knowledges/blob/main/android/images/proguard/Screenshot%20from%202020-10-27%2014-06-44.png) ![minifyEnabled=false](https://github.com/sanren1024/knowledges/blob/main/android/images/proguard/Screenshot%20from%202020-10-27%2014-09-05.png) 


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
xml文件中根标签是  **\<resources\>**，在 <code>**tools:keep**</code> 属性下配置需要保留的资源，在<code>**tools:discard**</code>属性下配置要丢弃的资源。

并将配置文件命名为 <code>keep.xml</code> 保存在<code>raw</code>目录下<code>res.raw/keep.xml</code>。

```xml
<?xml version="1.0" encoding="utf-8"?>  
<resources  xmlns:tools="http://schemas.android.com/tools"  tools:keep="@layout/l_used*_c,@layout/l_used_a,@layout/l_used_b*"  tools:discard="@layout/unused2"  />
```

构建工具不会将此文件打包到APK文件中。

明确要移除的资源，可能会被说还不如直接删除更加直接。但是在使用构建变量时，这是很有用的。例如，在project资源文件夹中有众多资源，且为不同的构建变量创建不同的<code>keep.xml</code>文件。此时，对于已知的构建变量，知道需要使用的资源。


### 严格引用检查

如果使用了<code>Resources.getIdentifier()</code>(或者库中使用了——AppCompat库使用了)，这意味着代码需要依据**动态生成的字符串**进行资源搜索。这样<code>R8</code>在资源压缩时会认定**动态生成的字符串资源名**开始的所有资源文件可能会被引用，因此不会进行移除。

例如：

```kotlin
val name =  String.format("img_%1d", angle +  1)  
val res = resources.getIdentifier(name,  "drawable", packageName)
```

代码中，资源名是动态生成的，因此<code>R8</code>会认定所有以<code>img_</code>开始的资源会被引用，因此一些即便不被使用，但是名字以<code>img_</code>开始的资源文件不会被移除。

同样，资源压缩器会分析代码中的字符串常量，以及<code>/res/raw/</code>目录下各种资源，类似<code>file:///android_res/drawable/ic_plus.png</code>的URL地址。如果压缩器检查到类似这些地址或资源，或者看起来可以组成类似的URL地址的资源，压缩器不会移除这些资源。

以上这些均是在**默认的safe模式**下的资源压缩。

另外一种即是**strict**模式，需要在<code>raw</code>目录下的<code>keep.xml</code>内配置<code>strict</code>值。

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources xmlns:tools="http://schemas.android.com/tools"
    tools:shrinkMode="strict" />
```
这样凡是未被<code>R8</code>认为被引用的资源将被移除。


### 资源压缩测试

在project中有资源<code>airplane_space.png</code>的图片资源，在<code>layout</code>目录下保留有不被引用的fragment xml文件。

![project resources strcuture](https://github.com/sanren1024/knowledges/blob/main/android/images/proguard/Screenshot%20from%202020-10-28%2011-46-44.png)

代码如下：

```kotlin
    val name = "airplane_space"
    findViewById<AppCompatButton>(R.id.button_get_identifier_res).setOnClickListener {
        val resID = resources.getIdentifier(name, "mipmap", packageName)
        findViewById<AppCompatImageView>(R.id.image_ret).setImageResource(resID)
    }
```

这里在运行时使用<code>getIdentifier()</code>来获取资源id。打**release**包。

在打**release**包前，还需要搞清楚一个问题，即资源压缩在默认情况下是**safe**模式下，另外一个是**strict**模式。这种模式下是资源压缩处理是不同的。

----

> 下面来看下两种模式下不同的资源表现

1. layout文件

    - **safe mode**
        ![save mode](https://github.com/sanren1024/knowledges/blob/main/android/images/proguard/proguard_shrink_resources_safe_mode.png)
    
      上图是在**safe**模式的资源压缩下，在打包过程中列出的未使用布局文件资源(unused resource)。这里可以看出，被处理的是系统文件，App下的布局文件未被处理。
    也可以通过反编译，查看到，未被使用的布局文件内容未被处理。

    - **strict mode**
        ![strict mode](https://github.com/sanren1024/knowledges/blob/main/android/images/proguard/proguard_shrink_resource_strict_mode.png)
         
      上图中显示的是**strict**模式的资源压缩下，针对App内未被引用的fragemnt  xml文件进行的处理。可以看到括弧内提示，原有文件内容被104字节内容替换掉了(***replaced with small dummy file of size 104 bytes***)。
  也就是文件没有被移除，但是文件内容被替换成了固定大小(104字节)内容。在反编译后，打开被处理过的xml文件，固定内容如下：
  
    ```xml
	<?xml version="1.0" encoding="utf-8"?>
	<x />
    ```
    
2. 图片资源

    - **safe mode**
        在**safe**模式下，使用运行时代码动态加载的图片资源未被移除。
        ![unremoved](https://github.com/sanren1024/knowledges/blob/main/android/images/proguard/proguard_shrink_resources_image_safe_mode.png)
      
    - **strict mode**
      在**strict**模式下，图片资源的会被移除，与布局资源文件一样，图片文件依然存在，但内容已经被替换。
      ![removed](https://github.com/sanren1024/knowledges/blob/main/android/images/proguard/proguard_shrink_resources_image_strict_mode.png)

如果要在**strict**资源压缩模式下，保留动态加载的图片不被处理，需要在<code>/res/raw/keep.xml</code>中使用<code>tools:keep</code>来设置需要保留的资源。

这次的测试的保留图片资源，设置带代码如下。
```xml
<?xml version="1.0" encoding="utf-8"?>
<resources xmlns:tools="http://schemas.android.com/tools"
    tools:shrinkMode="strict"
    tools:keep="@mipmap/airplane_space"/>
```


### 移除重复资源

资源压缩器只会移除不被code引用的资源，也就意味着可能因为设备配置的不同导致可选资源被移除。例如，多语言apk中会包含有多种语言的string字符串资源，但在很多情况下只需要其中一种或若干种语言翻译，此时其他的语言种类可以移除。这种情况下，可以使用gradle的<code>resConfig</code>类配置需要保留的资源包，其他未配置的语言包将被移除。

```groovy
android {
    defaultConfig {
        ...
        resConfigs "en", "fr"
    }
}
```

类似的可以配置不同的分辨率设备，以及不同的ABI配置的资源。


### 合并(merge)重复资源

Gradle在一般情况下会合并在不同资源目录下的同名资源文件，例如在不同<code>drawable</code>目录下的资源。这个合并过程不是通过<code>shrinkResources</code>配置项控制的，也不能停止，因为代码运行时在多个资源中寻找匹配的资源可以避免错误的发生。

当两个或更多资源共有相同的名字，类型，及限定名情况下，会发生资源合并。

<!--stackedit_data:
eyJoaXN0b3J5IjpbNTM5ODczMjE5LDIxMDgwODIxNDQsLTEwOD
Q1MzcwMTUsLTczNTkxNjAwNSwxNTkwNjU1MDkzLC0xNjczNjEy
NjYxLC0xNjAwNjUxMDY2LDYzODcyNjQzNCwyMDM4OTE1NjAsNz
QwOTgxMjk0LDE5MDA2Mzg3NjYsLTEwMjgwMTk4OTgsMTM5MjE0
MjU0MiwtMTI2MjEyNTc3Myw2NDcwMjI2NDIsLTIwMjIzMDY5Mz
ksLTExMDM5NDExNzhdfQ==
-->