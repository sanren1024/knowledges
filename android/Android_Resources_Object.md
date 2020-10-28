- [Resources不同获取方式](#Resources不同获取方式)
    * [Resources.getSystem()](#Resources.getSystem())
    * [Context.getResources()](#Context.getResources())


# Resources不同获取方式

最近在测试一段资源获取代码时，发现语法等均没有错误，但无法准确获取到资源ID，结果总是返回0。

代码如下：

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContentView(R.layout.activity_proguard)

    val name = "airplane_space"
    findViewById<AppCompatButton>(R.id.button_get_identifier_res).setOnClickListener {
        val resID = Resources.getSystem().getIdentifier(name, "mipmap", packageName)
        findViewById<AppCompatImageView>(R.id.image_ret).setImageResource(resID)
    }
}
```

在运行后的结果是，<code>resID</code>返回总是0。

反复测试下，一直未怀疑到 <code>Resources</code>对象的获取方式有问题，在反复测试后才关注到这个问题。
将<code>Resources.getSystem()</code>替换为<code>resources</code>，即：

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContentView(R.layout.activity_proguard)

    val name = "airplane_space"
    findViewById<AppCompatButton>(R.id.button_get_identifier_res).setOnClickListener {
        val resID = resources.getIdentifier(name, "mipmap", packageName)
        findViewById<AppCompatImageView>(R.id.image_ret).setImageResource(resID)
    }
}
```

再运行后即可获取到正确的<code>resID</code>值。


## Resources.getSystem()

这个方式获取的<code>Resources</code>对象，其关联到的只能是系统资源，即只能访问系统资源，不能访问app资源(application resources)。

这样的获取方式，并不能在当前运行的<code>Activity</code>等组件运行时使用其**dimen**，也不会根据方向而改变。运行时不会被运行时资源影响。


## Context.getResources()

返回与Application关联的<code>Resources</code>对象，即通过这个对象，可以访问到app资源。这个<code>Resources</code>对象与通过<code>getAssets()</code>返回的<code>AssetManager</code>对象一致，即他们共享有一个<code>Configuration</code>数据。

**一般大家在开发过程中很少会使用<code>Resources.getSystem()</code>方式去获取<code>Resources</code>对象，但这里踩到了坑了，这个细节还是需要注意的。**
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTQ0MzA1MzQ0NiwtMzEyNTk2ODIxLDk2OD
YxNzk3NF19
-->