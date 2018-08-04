---
layout:     post
title:      DataBinding学习(二)
subtitle:   Binding Adapter
date:       2018-08-03
author:     Anriku
header-img: img/2018_08_04_post.jpeg
catalog: true
tags:
    - Android
    - Android架构
    - Jetpack
    - DataBinding
---

前一篇博客我们介绍了DataBinding的使用。这一篇我们来看点更高级的东西---Binding Adapter。话不多说，开始吧！！！



# 配置

由于BindingAdapter是通过注解处理器来实现的。因此我们在之前的BindingAdapter的启用基础上还有在依赖中添加如下内容：

```groovy
...

apply plugin: 'kotlin-kapt'

android {
...
    dataBinding{
        enabled = true
    }
}

dependencies {
    ...
    kapt "com.android.databinding:compiler:3.1.3"
}
```

**这里用的是Kotlin因此用kapt来进行添加，而且还要用到kotlin-kapt插件。如果是Java的话用annotationProcessor来代替kapt。**



# 自定义方法对应的属性名

在有些View中的某些setter方法也许没有对应的xml属性。但是我们有想在xml中进行定义怎么办！这是我们就需要为方法自定义属性名了。当然我可以为已存在的setter设置另外的属性名。下面举个例子：

```kotlin
@BindingMethods(value = [
    (BindingMethod(type = android.widget.TextView::class,
            attribute = "myBackgroundColor",
            method = "setBackgroundColor"))
])
class MyTextView: TextView {

    constructor(context: Context?) : super(context)
    constructor(context: Context?, attrs: AttributeSet?) : super(context, attrs)
    constructor(context: Context?, attrs: AttributeSet?, defStyleAttr: Int) : super(context, attrs, defStyleAttr)
}
```

这里我们为TextView的setBackgroundColor方法设置了另外的一个属性名叫做myBackgroundColor，**它的命名空间是app(也可以不添加命名空间)。如果想设置为android就需要在在前面显示的设置android命名空间。如："android:myBackgroundColor"**



下面是我们在xml中的使用：

```xml
<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools">

    <data>
    </data>
    
    <androidx.constraintlayout.widget.ConstraintLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        tools:context=".MainActivity">

        <com.anriku.jetpackdemo.databinding.MyTextView
            ...
            app:myBackgroundColor="@{@color/colorPrimary}"
            ... />

    </androidx.constraintlayout.widget.ConstraintLayout>
</layout>
```

可以看到在MyTextView的属性设置中我们使用了myBackgroundColor属性。效果图就不给了。自己实践下吧！**这里需要注意的我们不能直接使用@color/colorPrimary而应该使用@{@color/colorPrimary}**



# 自定义属性名及其逻辑

上面讲的@BindMethods都是建立在已有的方法之上的。我们通过@BindAdapter还可以自定义属性名以及它所做的逻辑。

举个例子：

```kotlin
object BindingAdapter {
    @BindingAdapter("name")
    @JvmStatic fun setName(view: View, name: String) {
        Toast.makeText(view.context,name, Toast.LENGTH_SHORT).show()
    }
}
```

可以看到我们自定义了一个name属性。对应的方法是Toast这个名字出来。**这里要注意的是Kotlin中使用object并且将方法用@JvmStatic进行修饰。也就是Java中的静态方法**



下面是xml的实现

```xml
<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools">

    <androidx.constraintlayout.widget.ConstraintLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        tools:context=".MainActivity">

        <TextView
            ...
            name="@{`anriku`}"
            ... />

    </androidx.constraintlayout.widget.ConstraintLayout>

</layout>
```



我们再具体的看下@BindingAdapter这个注解。**下面的注解代码是Java代码**

```java
@Target(ElementType.METHOD)
public @interface BindingAdapter {
    String[] value();
    boolean requireAll() default true;
}
```

可以看到有两个值一个是value，另一个是requireAll并且它的默认值为true。**value我们可以发现是String数组，因此我们可以设置多个属性来激化同一个方法。requireAll的话就是当我们的属性设置为多个的时候，是否需要一个控件设置了全部属性才能激活这个方法。默认的话就是要全部属性都设置了才行**



下面我们来看一个value有两个值、requireAll设置为false的例子吧！

```kotlin
object BindingAdapter {
    @BindingAdapter(value = ["firstName", "lastName"], requireAll = false)
    @JvmStatic
    fun setName(view: View, firstName: String?, lastName: String?) {
        Toast.makeText(view.context, "${firstName?:""}${lastName?:""}", 
        Toast.LENGTH_SHORT).show()
    }
}
```

这样我们就可以在xml中设置对这个两个属性任意的组合设置了。**如果requireAll为true(默认就是)，那么设置了要么全部设置，要么全不设置。**



# 自定义属性值转换

情况如下：

```xml
<layout>
    <data>
        <variable
            name="handlers"
            type="com.anriku.jetpackdemo.MyHandlers"/>
    </data>

...
        <TextView
            ...
            android:text="@{handlers}"
            .../>
...

</layout>
```

当遇到如上情况，我们给text属性传递了一个MyHandlers对象(就是一个简单自定义对象)。它不能被接受这时候就会报错。那么我们如果偏要让text属性能接受MyHandlers对象呢？那么我们就要用自定义转换来进行实现了。



代码如下：

```kotlin
object BindingAdapter {

    @BindingConversion
    @JvmStatic fun convertToString(obj: MyHandlers): String{
        Log.e("MyHandlers", "convertToString")
        return obj.toString()
    }
}
```

**自定义转换用@BindingConversion注解来实现。**这里我们在属性接受到了MyHandlers对象的时候，调用其toString方法。



现在在xml中你就可以直接想text属性传入MyHandlers对象了。



# 总结

这篇博客给大家讲了下BindingAdapter相关的东西。

主要有三点：

* 给已有的方法设置属性名或起别名。**用@BindingMethods**
* 自定义属性以及其实现逻辑。**用@BindingAdapter**
* 对属性参数值进行转换。**用@BindingConversoin**



# 参考

[官方文档](https://developer.android.com/topic/libraries/data-binding/binding-adapters#kotlin)



*转载请注明链接*