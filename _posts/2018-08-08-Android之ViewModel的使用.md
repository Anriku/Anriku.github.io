---
layout:     post
title:      Android之ViewModel的使用
subtitle:   ViewModel的使用
date:       2018-08-08
author:     Anriku
header-img: img/2018_08_08_post.jpeg
catalog: true
tags:
    - Android
    - Android架构
    - Jetpack
    - ViewModel
---

Android中的ViewModel是一个可以用来存储UI相关的数据的类。ViewModel的生命周期会比创建它的Activity、Fragment的生命周期长。

这里拿官方的一张图：

![ViewModel-Lifecycle](http://oyil5gdc8.bkt.clouddn.com/viewmodel-lifecycle.png)

**这张图是在在没任何设置屏幕发生转换Activity的生命周期变化和ViewModel的生命周期。可以看重建的时候，ViewModel中的数据是不会被清理的。**



借助于上面这一特点，ViewModel有下面的两个优点：

* Activity进行重建的时候，ViewModel的数据不会被回收调用。这时候我们就可以不用通过onSaveInstanceState()方法来进行数据的存储了。而且用onSaveInstanceState()方法为了使Activity能够尽快的重建还只能存储少量的数据进行恢复。
* Activity中通常会有有那种在其创建的时候获取数据，然后在其销毁的时候释放数据的方法。如果这些放在Activity中的话，在Activity进行重建的时候，会很浪费资源。但是如果方法在ViewModel中的话，Activity的重建将不会导致数据的重复获取。



**当然，屏幕的旋转，你也可以通过configChanges的设置来阻止它的重建。但是其它的有些意外情况Activity也是有可能重建的**



# ViewModel的使用

其实，ViewModel的使用在前面的LiveData例子中就有相应的体现。这篇博客就写一个通过ViewModel来实现Fragment之间通信的例子吧！



##### 先进行ViewModel的创建

```
class SharedViewModel : ViewModel() {
    var sharedName:MutableLiveData<String> = MutableLiveData()
    
    init {
        sharedName.value = "anriku"
    }
}
```

这个ViewModel创建得很简单，就是自定义一个SharedViewModel继承自ViewModel。这里ViewModel中包含一个LiveData对象。关于ViewModel更难一些内容将在后面进行介绍。



##### 再进行OneFragment的创建

xml如下：

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical" android:layout_width="match_parent"
    android:layout_height="match_parent">

    <Button
        android:id="@+id/bt_fragment_one"
        android:text="改变SharedViewModel的值"
        android:layout_width="match_parent"
        android:layout_height="wrap_content" />

</LinearLayout>
```

这个Fragment的xml很简单，就只有一个Button控件。这个Button控件是用来改变ViewModel的值的。然后在TwoFragment中体现改变的值。



OneFragment的代码如下：

```kotlin
class OneFragment: Fragment() {

    override fun onCreateView(inflater: LayoutInflater, container: ViewGroup?, 
                              savedInstanceState: Bundle?): View? {
        return inflater.inflate(R.layout.fragment_one, container, false)
    }

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)

        val model = ViewModelProviders.of(activity!!).get(SharedViewModel::class.java)

        bt_fragment_one.setOnClickListener{
            model.sharedName.value = "anrikuwen"
        }
    }

    override fun onDestroy() {
        Toast.makeText(activity,"OneFragment is destroyed", Toast.LENGTH_SHORT).show()
        super.onDestroy()
    }
}
```

这里OneFragment中我们通过**ViewModelProviders.of(activity!!).get(SharedViewModel::class.java)**来进行SharedViewModel的获取，这里需要注意的就是of的参数给的是Fragment所绑定的Activity，因此它的生命周期就以Fragment绑定的Activity作为参照对象了。Fragment的创建销毁对SharedViewModel没有任何影响。



然后Button控件设置监听器，使其按一下后改变SharedViewModel中的sharedName的值。



##### 再进行TwoFragment的创建

xml如下：

```kotlin
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical" android:layout_width="match_parent"
    android:layout_height="match_parent">

    <TextView
        android:id="@+id/tv"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />

</LinearLayout>
```

没看错，依然如此的清新。就是一个TextView控件在里面。用于显示ShanredViewModel中的属性的值。



下面是TwoFragment的代码：

```kotlin
class TwoFragment: Fragment() {

    override fun onCreateView(inflater: LayoutInflater, container: ViewGroup?, 
    savedInstanceState: Bundle?): View? {
        return inflater.inflate(R.layout.fragment_two,container,false)
    }

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)

        val model = ViewModelProviders.of(activity!!).get(SharedViewModel::class.java)

        model.sharedName.observe(this, Observer {
            tv_fragment_two.text = it
        })
    }

    override fun onDestroy() {
        Toast.makeText(activity, "TwoFragment is destroyed", Toast.LENGTH_SHORT).show()
        super.onDestroy()
    }
}
```

这里获取SharedViewModel的方式和OneFragment是一样的。

但是在TwoFragment中，我们给SharedViewModel中的LiveData属性设置了一个Observer对象，在其数据发生了变化之后，如果TwoFragment处于活跃状态的话就进行改变，不活跃就会等其活跃之后再进行对应的UI变化。



##### 最后看看用来管理这两个Fragment的Activity

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".viewmodel.ViewModelActivity">

    <Button
        android:id="@+id/bt_one"
        android:layout_width="0dp"
        android:text="切换OneFragment"
        android:layout_height="wrap_content"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintBottom_toTopOf="@id/bt_two"
        app:layout_constraintRight_toRightOf="parent"/>

    <Button
        android:id="@+id/bt_two"
        android:layout_width="0dp"
        android:text="切换TwoFragment"
        android:layout_height="wrap_content"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintBottom_toTopOf="@id/fl"
        app:layout_constraintTop_toBottomOf="@id/bt_one"/>

    <FrameLayout
        android:id="@+id/fl"
        android:layout_width="0dp"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintTop_toBottomOf="@id/bt_two"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        android:layout_height="0dp">

    </FrameLayout>

</androidx.constraintlayout.widget.ConstraintLayout>
```

上面xml没啥难点。这里的两个Button分别用来显示对应的Fragment的。FramLayout用来显示Fragment的。



再来看看Activity中的代码：

```kotlin
class ViewModelActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_view_model)

        replaceFragment(TwoFragment())
        
        bt_one.setOnClickListener{
            replaceFragment(OneFragment())
        }
        bt_two.setOnClickListener{
            replaceFragment(TwoFragment())
        }
    }

    private fun replaceFragment(fragment: Fragment){
        val fragmentManager = supportFragmentManager
        val transaction = fragmentManager.beginTransaction()
        transaction.replace(R.id.fl,fragment)
        transaction.commit()
    }

    override fun onDestroy() {
        Log.e("ViewModelActivity", "onDestroy")
        super.onDestroy()
    }
}
```

对现在在学习ViewModel的你上面代码没啥问题吧，就是Fragment之间的转换逻辑。



OK，万事俱备了。现在自己来试试吧！在OneFragment对其SharedViewModel的值进行改变。然后再切换到TwoFragment中。当当当！！！轻松的就实现了Fragment之间的通信了吧！



#构造器有参数的ViewModel

 当我们的ViewModel需要进行构造器需要穿参数的时候，就不能像上面一样进行实例化了。而需要借助于ViewModelProvider的Fatory来进行构造。



下面是对前面的SharedViewModel进行修改的代码：

```kotlin
class SharedViewModel(sharedName: String) : ViewModel() {
    var sharedName: MutableLiveData<String> = MutableLiveData()

    init {
        this.sharedName.value = sharedName
    }

    class SharedViewModelFactory(private val sharedName: String) : 
    ViewModelProvider.Factory {
        override fun <T : ViewModel?> create(modelClass: Class<T>): T {
            return SharedViewModel(sharedName) as T
        }
    }
}
```

可以看到在SharedViewModel有一个内部类基础自ViewModelProvider.Factory接口。并重写了create方法。这个方法返回一个ViewModel这里我们就是要实例化SharedViewModel对象，因此这里返回一个SharedViewModel对象，可以看到参数这时候就通过SharedViewModelFactory传给了SharedViewModel了。



这时候不光是SharedViewModel要进行修改，在进行SharedViewModel的获取的时候也是需要进行相应的修改的。

前面两个Fragment获取SharedViewModel部分的代码都要改成如下代码：

```kotlin
val model = ViewModelProviders.of(activity!!, SharedViewModel.SharedViewModelFactory("anriku")).get(SharedViewModel::class.java)
```

其实就是在获取ViewModelProvider对象的of方法有修改。这里用的是带有Factory的of方法。传入刚才在SharedViewmodel中写的Factory的实例。





# AndroidViewModel

使用ViewModel的时候，需要注意的是ViewModel不能够持有View、Lifecycle、Acitivity引用，而且不能够包含任何包含前面内容的类。因为这样很有可能会造成内存泄漏。



那如果需要使用Context对象改怎么办。这时候我们可以给ViewModel一个Application。Application是一个Context，而且一个应用也只会有Application。



我们自己添加Application？其实没必要Google还有一个AndroidViewModel。这是一个包含Application的ViewModel。



下面是一个AndroidViewModel的源码：

```kotlin
public class AndroidViewModel extends ViewModel {
    @SuppressLint("StaticFieldLeak")
    private Application mApplication;

    public AndroidViewModel(@NonNull Application application) {
        mApplication = application;
    }

    /**
     * Return the application.
     */
    @SuppressWarnings("TypeParameterUnusedInFormals")
    @NonNull
    public <T extends Application> T getApplication() {
        //noinspection unchecked
        return (T) mApplication;
    }
}
```

可以看到只是在ViewModel的基础上添加了Application。



# 总结

这篇博客主要讲解了ViewModel构造参数有参数以及无参数情况下的创建。并且通过一个例子，看到了ViewModel如何轻易的实现Fragment之间的通信的。



# 参考

[官方文档](https://developer.android.google.cn/topic/libraries/architecture/viewmodel)

