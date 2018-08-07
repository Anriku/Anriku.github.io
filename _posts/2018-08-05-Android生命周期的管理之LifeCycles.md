---
layout:     post
title:      Android生命周期的管理之LifeCycles
subtitle:   LifeCyle学习
date:       2018-08-05
author:     Anriku
header-img: img/2018_08_05_post.jpeg
catalog: true
tags:
    - Android
    - Android架构
    - Jetpack
    - LifeCycle
---

前面两篇博客，我们介绍了DataBinding的使用。这篇博客我们开始新的学习。本篇我们讲解LifeCycles相关的内容。



# 传统生命周期管理的缺陷

像Activity、Fragment中我们都很清楚它们是具有生命周期的。而生命周期对我们来说也是很重要的。但处理不好就很有可能导致内存泄漏。

传统的生命周期管理是在不同的生命周期的回调中进行相应操作的调用。比如说在进行播放器相关类的调用的时候。在onStart()生命周期中进行连接操作。但是忘了在onStop()方法中进行断连操作了。这时候就会导致某些对象得不到释放久而久之就会导致内存泄露。



那么LifeCycles是如何进行管理的呢？**LifeCycles是通过注解的方式在我们进行类的编写的时候就指定的哪些函数在哪个生命周期执行，然后使其观察拥有生命周期的类的生命周期。**作为类的调用者就可以不用管理那么多生命周期相关的东西了。不会因为粗心大意而导致内存泄露了。



# LifeCycleObserver和LifeCycleOwner

LifeCyclerObserver是我们要实现的具有生命周期感知的类的需要实现的接口，这个接口没有任何方法。在这个类中我们通过注解来表明函数在LifeCycleOwner的哪个生命周期的时候执行。

LifeCycleOwner也是一个接口，这个接口只有getLifeCycle一个方法。用于标志它的实现类是具有生命周期的类。在26.0.1版本后的support库中的Activity、Fragment都实现了LifeCycleOwner接口。说到这里又不得不说一下support Library在Android P之后将不再进行新特性的添加了。取而代之的是androidx库，Google推出的另一个Android兼容库。



下面我们来自定义一个LifeCycleObserver和LifeCycleOwner的接口类吧！



#### LifecycleOwner的实现

```kotlin
class MyLifeCycleOwner: LifecycleOwner {

    private lateinit var lifecycleRegistry: LifecycleRegistry

    fun onCreate(){
        lifecycleRegistry = LifecycleRegistry(this)
        lifecycleRegistry.markState(Lifecycle.State.CREATED)
    }

    fun onStart(){
        lifecycleRegistry.markState(Lifecycle.State.STARTED)
    }

    fun onResume(){
        lifecycleRegistry.markState(Lifecycle.State.RESUMED)
    }

    fun onPause(){
        lifecycleRegistry.markState(Lifecycle.State.RESUMED)
    }

    fun onStop(){
        lifecycleRegistry.markState(Lifecycle.State.STARTED)
    }

    fun onDestroy(){
        lifecycleRegistry.markState(Lifecycle.State.CREATED)
    }
    
    override fun getLifecycle(): Lifecycle {
        return lifecycleRegistry
    }
}
```

这里我们的类中用于管理生命周期的类是LifecycleRegistry类。这个类是LifeCycle的子类，因此我们能够在getLifeCycle方法中返回LifeCycle类。然后其它的方法很简单用于在不同Activity的不同生命周期中进行状态的标记。



#### LifecycleObserver接口的实现

```kotlin
class MyObserver(var lifeCycle: Lifecycle) : LifecycleObserver {

    @OnLifecycleEvent(Lifecycle.Event.ON_RESUME)
    fun connectListener() {
        Log.e("MyObserver", "resume")
    }

    @OnLifecycleEvent(Lifecycle.Event.ON_PAUSE)
    fun disconnectListener() {
        Log.e("MyObserver", "pause")
    }

    fun getState(){
        if (lifeCycle.currentState.isAtLeast(Lifecycle.State.RESUMED)){
            Log.e("MyObserver", "isAtLeastResumed")
        }
    }
}
```

可以看到这个类的构造函数传了一个LifeCycle类进来。这就是我们上面LifeCycleOwner实现类的getLifeCycle返回的东西。

因为我们在getState方法中需要使用LifeCycle类这里才传的。如果不需使用到LifeCycle类，我们可以不用传进来。

**然后其它的方法都使用了@OnLifecucleEvent注解，注解值为我们在哪个生命周期变化过程中调用。比如：Lifecycle.Event.ON_RESUME就是在CREATED状态到RESUME状态的过程中进行调用。**



#### MainActivity中进行调用

```kotlin
class MainActivity : AppCompatActivity() {

    private lateinit var observer: MyObserver
    private lateinit var myLifeCycleOwner: MyLifeCycleOwner

    override fun onCreate(savedInstanceState: Bundle?) {
        myLifeCycleOwner = MyLifeCycleOwner()
        myLifeCycleOwner.onCreate()
        super.onCreate(savedInstanceState)
        val binding = DataBindingUtil.setContentView<ActivityMainBinding>(this,
                R.layout.activity_main)

        observer = MyObserver(myLifeCycleOwner.lifecycle)
        myLifeCycleOwner.lifecycle.addObserver(observer)

    }
    
    override fun onStart() {
        myLifeCycleOwner.onStart()
        super.onStart()
    }

    override fun onResume() {
        myLifeCycleOwner.onResume()
        super.onResume()
        observer.getSate()
    }

    override fun onPause() {
        myLifeCycleOwner.onPause()
        super.onPause()
    }

    override fun onStop() {
        myLifeCycleOwner.onStop()
        super.onStop()
    }

    override fun onDestroy() {
        myLifeCycleOwner.onDestroy()
        super.onDestroy()
    }
}
```

这里我们其实就是让Activity的每个生命周期给了MyLifecycleOwner。



上面我们学到了如果自定义LifecycleOwner和LifecycleObserver。**但是通常情况下我们都是不需要自行实现LifecycleOwner的。因为在兼容库中的Activity和Fragment都是实现了LifecycleOwner接口的。我们只需要自己实现LifecycleObserver就行的。**就像下面一样。

```kotlin
class MainActivity : AppCompatActivity() {

    private lateinit var observer: MyObserver

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        val binding = DataBindingUtil.setContentView<ActivityMainBinding>(this,
                R.layout.activity_main)

        observer = MyObserver(lifecycle)
        lifecycle.addObserver(observer)
    }
}
```

这样的话就可以根据Activity或Fragment的不同生命周期来调用不同的方法了。



# 总结

这篇博客主要讲了下LifecycleOwner、和LifecycleObserver两个类使用。特别是LifecycleObserver的使用会让我们的生命周期管理变得很方便。



# 参考

[官方文档](https://developer.android.com/topic/libraries/architecture/lifecycle)



*转载请注明链接*







