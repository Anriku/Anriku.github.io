---
layout:     post
title:      DataBinding学习(一)
subtitle:   DataBinding的布局与表达式
date:       2018-08-02
author:     Anriku
header-img: img/2018_08_03_post.jpeg
catalog: true
tags:
    - Android
    - Android架构
    - Jetpack
    - DataBinding
---

从这篇博客起。后面我会用几篇博客来讲解Android的架构组件。包括DataBinding、LiveData、Lifecycles、Navigation、Paging、Room、ViewModel以及WorkManager。讲解的代码将会用Kotlin进行编写。那么这就开始吧！



# 启用DataBinding

一般情况下、我们直接通过下图来进行DataBinding的启用。当前还可以在gradle.properties中做一些配置。这里就不用了。

![启用DataBinding](http://oyil5gdc8.bkt.clouddn.com/%E5%90%AF%E7%94%A8DataBinding.png)



# DataBinding的简单体验

### 布局的设置

```xml
<?xml version="1.0" encoding="utf-8"?>
<layout>

    <data>
        <variable
            name="user"
            type="com.anriku.jetpackdemo.databinding.User" />
    </data>

    <androidx.constraintlayout.widget.ConstraintLayout 		  xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:app="http://schemas.android.com/apk/res-auto"
        xmlns:tools="http://schemas.android.com/tools"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        tools:context=".MainActivity">


        <TextView
            android:id="@+id/tv_first_name"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:text="@{user.firstName}"
            app:layout_constraintLeft_toLeftOf="parent"
            app:layout_constraintRight_toRightOf="parent"
            app:layout_constraintTop_toTopOf="parent" />

        <TextView
            android:id="@+id/tv_last_name"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:text="@{user.lastName}"
            app:layout_constraintLeft_toLeftOf="parent"
            app:layout_constraintRight_toRightOf="parent"
            app:layout_constraintTop_toBottomOf="@id/tv_first_name" />


    </androidx.constraintlayout.widget.ConstraintLayout>

</layout>
```

通过上面的一个简单例子。可以看到DataBinding的布局**主要有三部分组成：最外层的layout、layout中的data、layout中的布局。其中data标签中是用来定义数据的。**结构是不是很简单。



现在我们对其中的内容进行一下讲解。

首先是data中我用variable标签来定义变量。**name表示变量名。type表示变量的类(包含它所在的包)。**



然后在布局中，我们分别在两个TextView的**text属性**中通过表示式让其分别显示firstName和lastName了。

OK!具体的一些语法我将在后面进行讲解



### 数据类的定义

```kotlin
data class User(val firstName: String, val lastName: String)
```

这是用Kotlin编写的数据类。data是数据类定义的关键字。其中equals()/hashcode对、toString()、componentN()、copy()函数会在数据类中默认进行实现。有关数据类的具体内容请[戳这里](https://www.kotlincn.net/docs/reference/data-classes.html)。



注意Kotlin在主构造函数中用val、var进行修饰后。默认会实现getter、setter方法。因此我们在上面布局中才能那样进行调用。看到这里是不是觉得Kotlin特别的简洁。还没学的、快点去学啦呀！！！



### 数据绑定

在Activity中我们通过下面的代码进行绑定：

```kotlin
class MainActivity : AppCompatActivity() {
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        val binding: ActivityMainBinding = DataBindingUtil.setContentView(this,
                R.layout.activity_main)
        
        //或者通过下面的方式
        //val binding = ActivityMainBinding.inflate(layoutInflater)
        //setContentView(binding.root)

        binding.user = User("anriku", "wen")
    }
    
}
```

代码中我们通过DataBindingUtil类的setContentView来进行生成binding类或者通过binding类的静态inflatte方法。**这个类是根据我们前面的布局来进行生成的。生成的名字默认为布局的每个下划线分开的单词的首字母大写加上Binding后缀。比如说：activity_main.xml生成ActivityMainBinding类。当然可以进行修改，这个在后面进行介绍。**



### DataBinding在Fragment、ListView的Adapter、RecyclerView的Adapter中的使用

```kotlin
val listItemBinding = ListItemBinding.inflate(layoutInflater, viewGroup, false)
// or
val listItemBinding = DataBindingUtil.inflate(layoutInflater, R.layout.list_item, viewGroup, false)
```

上面是生成binding类的方法。下面是一个简单的RecyclerView的Adapter的实现。

```kotlin
class RecAdapter(private var userList: List<User>, private var context: Context) : 
        RecyclerView.Adapter<RecAdapter.RecViewHolder>() {

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): RecViewHolder {
        val layoutInflater = LayoutInflater.from(context)
        val binding:RecItemBinding = RecItemBinding.inflate(layoutInflater,parent,false)
        return RecViewHolder(binding.root)
    }

    override fun getItemCount(): Int {
        return userList.size
    }

    override fun onBindViewHolder(holder: RecViewHolder, position: Int) {
        val binding = DataBindingUtil.getBinding<RecItemBinding>(holder.itemView)
        binding!!.user = userList[position]
    }
    
    class RecViewHolder(itemView: View) : RecyclerView.ViewHolder(itemView)
}
```



# XML中的表达式

在DataBinding的xml中我们可以使用在代码中大多数的操作符。在这部分我只会将一些特殊的东西拿来进行讲解。



### 不能使用的操作

在DataBinding的xml中，我们**不能使用this、super、new关键字**



### 空合并操作

在DataBinding的xml中可以使用**??**来代替**?:**操作符。Kotlin中对三元操作符也有相应的替代，这里就不细讲了。举个例子：

```xml
android:text="@{user.displayName ?? user.lastName}"

//与上面的逻辑相同
android:text="@{user.displayName != null ? user.displayName : user.lastName}"
```



### 集合类

在DataBinding的xml中我们可以像在Kotlin中一样。通过**[]**，来代替集合类的getter方法。**当我们的index为变量的时候，还可以用.来代替。如果是其它的字符串啥的作为key，或者数字作为index的时候就不能用.代替。**举个例子：

```xml
<data>
    <import type="java.util.Map"/>
    <import type="java.util.List"/>
    <variable name="list" type="List&lt;String&gt;"/>
    <variable name="map" type="Map&lt;String, String&gt;"/>
    <variable name="index" type="int"/>
    <variable name="key" type="String"/>
</data>
…
android:text="@{list[index]}"
//或者
android:text="@{list.index}"
…
android:text="@{map[key]}"
//或者
android:text="@{map.key}"
```



### 泛型

在DataBinding中的范型我们不等用**<类型>**。而应该用**&lt**;类型**&gt**;来进行表示。细心你应该在上面的集合的例子中就发现了吧！！！



### 字符串表示

在DataBinding的xml中我们有两种方式表示字符串。形式如下：

```
//单引号中用双引号
android:text='@{map["firstName"]}'

//双引号中用`
android:text="@{map[`firstName`]}"
```





# 事件处理

在DataBinding中有两种事件处理。**一种是方法引用、另一种是监听器绑定。**



### 方法引用

方法引用的特点是我们进行时间处理的方法的参数和监听器回调的方法的参数相同。举个例子：

进行事件处理的类

```kotlin
class MyHandlers {
    fun onClickTV(view: View) {
        Toast.makeText(view.context, "Hello", Toast.LENGTH_LONG).show()
    }
}
```

xml中的实现

```xml
...
    <data>
        <variable
            name="handlers"
            type="com.anriku.jetpackdemo.databinding.MyHandlers"/>
    </data>
...
        <TextView
            ...
            android:onClick="@{handlers::onclickTV}"
            .../>
...
```

最后在Activitiy中我们向Binding类中传入一个MyHandler的实例：

```kotlin
class MainActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        ...
        binding.handlers = MyHandlers()
    }
}
```



**View的OnclickListener中的onClick方法有一个View参数。因此我们进事件处理的方法也只有一个View参数。这样我们就可以在xml的onClick中通过@{handlers::onClickTV}来设置监听器了。**



### 监听器绑定

监听器绑定通过lambda表达式来实现。与前面方法引用不同的就是我们的方法参数可以不用包含对应监听器的方法的参数。但是包含就要全部包含。而且我们可以进行自行的添加参数。下面举两个例子：



##### 不带监听器参数的

进行事件处理的类

```kotlin
class MyHandlers {
    fun onCheckCB(str: String){
        Log.e("MyHandlers", str)
    }
}
```



xml的实现

```xml
...
    <data>
        <variable
            name="hello"
            type="String" />
        <variable
            name="handlers"
            type="com.anriku.jetpackdemo.databinding.MyHandlers" />
    </data>
...
        <CheckBox
            ...
            android:onCheckedChanged="@{() -> handlers.onCheckCB(hello)}"
            ... />

...
...
```

可以看到我们直接在lambda表达式中传入了我们加的参数。



##### 带监听器参数的

进行事件处理的类

```kotlin
class MyHandlers {
    fun onCheckCB(componentButton: CompoundButton, isChecked: Boolean, str: String){
        Toast.makeText(componentButton.context, str, Toast.LENGTH_LONG).show()
    }
}
```



xml中的实现

```xml
...
    <data>
        <variable
            name="hello"
            type="String" />

        <variable
            name="handlers"
            type="com.anriku.jetpackdemo.databinding.MyHandlers" />
    </data>
...
        <CheckBox
            ...
            android:onCheckedChanged="@{(componentButton, isChecked) ->  
            handlers.onCheckCB(componentButton, isChecked,hello)}"
            ... />
...
```

**通过上面可以看到如果要带参数的话一定要把监听器方法的参数全部带上**



# imports以及includes

### imports

在DataBinding中的import和Java中的是差不多的。**只不过我们可以通过alias对引入的类进行重命名这样可以解决类名相同的问题。其中import包含在data标签中**下面直接看个例子吧！！！

```xml
<import type="android.view.View"/>
<import type="com.anriku.View"
        alias="AnrikuView"/>
```



### includes

在DataBinding中我们仍然可以用include标签来减少重复代码。**如果被包含的布局中有变量我们可以通过bind命名空间来进行变量的传递。**

```xml
<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    xmlns:bind="http://schemas.android.com/apk/res-auto">
    
    <data>
        <variable
            name="hello"
            type="String" />
    </data>

    <androidx.constraintlayout.widget.ConstraintLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        tools:context=".MainActivity">

        <include layout="@layout/include_layout"
            bind:helloInclude="@{hello}"/>

    </androidx.constraintlayout.widget.ConstraintLayout>

</layout>
```

其中include_layout是被包含的布局。**我们将这个布局中的hello变量传给了include_layout布局中的helloInclude变量。**



# 总结

这篇博客给大家讲解了关于DataBinding的基本使用。相信大家看完已经掌握得差不多了。现在看看。我们主要学到了什么。

主要就是两部分：

* 代码中对对应xml生成的Binding类的使用。
* 以及xml中的使用。



# 参考

[官方文档](https://developer.android.com/topic/libraries/data-binding/expressions#kotlin)



*转载请注明链接*