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


<!--stackedit_data:
eyJoaXN0b3J5IjpbOTY4NjE3OTc0XX0=
-->