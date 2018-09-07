---
layout:     post
title:      Android使用Transition来更改布局
subtitle:   Android Transition的使用
date:       2018-09-06
author:     Anriku
header-img: img/2018_09_06_post.jpeg
catalog: true
tags:
    - Android
    - Transition
---

前面讲了属性动画。这篇博客将会给大家分享一下如何使用Transition来进行布局的更改。当然Transition也是基于属性动画的。



# Transition的特点

当布局更改后通过Transition可以实现布局的过渡动画。

它有如下的特点：

* **Group-level animations：**可以对一个View层级下的所有View应用一个或多个动画效果
* **Built-in animations**: 可以应用提前定义好的动画
* **Resource file support**: 可以从xml中加载内置的动画
* **Lifecycle callbacks**: 可以接收回调实现动画生命周期的控制



# 使用Transition步骤

这里直接拿官方的来进行说明：

![transition](http://oyil5gdc8.bkt.clouddn.com/transitions_diagram.png)

通过这张图可以看到，整个过程由Transition Manager通过Transition将Starting Scene转换为Ending Scene。



因此，这里有三部进行Transition的使用：

* 为Starting layout、Ending layout创建[Scene](https://developer.android.google.cn/reference/android/transition/Scene.html)
* 创建定义好你想要的动画效果的[Transition](https://developer.android.google.cn/reference/android/transition/Transition.html)
* 调用[TransitionManager.go()](https://developer.android.google.cn/reference/android/transition/TransitionManager.html#go(android.transition.Scene))进行layout的切换



# 代码准备

创建TransitionActivity：

```kotlin
class TransitionActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_transition)
        if (savedInstanceState == null) {
            val translation = supportFragmentManager.beginTransaction()
            translation.replace(R.id.container, TransitionFragment())
            translation.commit()
        }
    }
}
```

这个Activity很简单，xml就是一个无子View的名为container的FramLayout。这里就不列出来了。



然后是TransitionFragment的xml：

```kotlin
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <RadioGroup
        android:id="@+id/rg"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal">

        <TextView
            style="?android:textAppearanceMedium"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:text="scene" />

        <RadioButton
            android:id="@+id/rb_1"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:checked="true"
            android:text="1" />

        <RadioButton
            android:id="@+id/rb_2"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:text="2" />

        <RadioButton
            android:id="@+id/rb_3"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:text="3" />

        <RadioButton
            android:id="@+id/rb_4"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:text="4" />

    </RadioGroup>

    <FrameLayout
        android:id="@+id/scene_root"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1">

        <include layout="@layout/scene1" />

    </FrameLayout>

</LinearLayout>
```

这里上面RaidoGroup是用来进行各个Scene的转换的。

**下面的FramLayout是各个变换的布局的父布局，它的子View将会做Transition中定义的动画变换。这里在FrameLayout通过inclue来引入了scene1布局。**

TransitionFragment的代码在Scene再进行说明。



# Scene的创建

首先，先创建三个scene的xml如下：

scene1.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:id="@+id/container"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <View
        android:id="@+id/accent_view"
        android:layout_width="50dp"
        android:layout_height="50dp"
        android:background="@color/colorAccent"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

    <View
        android:id="@+id/primary_view"
        android:layout_width="50dp"
        android:layout_height="50dp"
        android:background="@color/colorPrimary"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintTop_toBottomOf="@id/accent_view" />

    <View
        android:id="@+id/dark_primary_view"
        android:layout_width="50dp"
        android:layout_height="50dp"
        android:background="@color/colorPrimaryDark"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintTop_toBottomOf="@id/primary_view" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

scene2.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:id="@+id/container"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <View
        android:id="@+id/accent_view"
        android:layout_width="50dp"
        android:layout_height="50dp"
        android:background="@color/colorAccent"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

    <View
        android:id="@+id/primary_view"
        android:layout_width="50dp"
        android:layout_height="50dp"
        android:background="@color/colorPrimary"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintLeft_toLeftOf="parent" />

    <View
        android:id="@+id/dark_primary_view"
        android:layout_width="50dp"
        android:layout_height="50dp"
        android:background="@color/colorPrimaryDark"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintRight_toRightOf="parent" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

scene3.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:id="@+id/container"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <View
        android:id="@+id/accent_view"
        android:layout_width="50dp"
        android:layout_height="50dp"
        android:background="@color/colorAccent"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

    <View
        android:id="@+id/primary_view"
        android:layout_width="50dp"
        android:layout_height="50dp"
        android:background="@color/colorPrimary"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintLeft_toLeftOf="parent" />

    <View
        android:id="@+id/dark_primary_view"
        android:layout_width="50dp"
        android:layout_height="50dp"
        android:background="@color/colorPrimaryDark"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintRight_toRightOf="parent" />

    <TextView
        android:id="@+id/transition_title"
        style="?android:textAppearanceMedium"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Hello This Transition"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

</androidx.constraintlayout.widget.ConstraintLayout>
```



前面说明了TransitionFragment的xml，这里接着说明对应的代码：

```kotlin
class TransitionFragment : Fragment(), RadioGroup.OnCheckedChangeListener {

    private lateinit var mScene1: Scene
    private lateinit var mScene2: Scene
    private lateinit var mScene3: Scene
    private lateinit var mTransitionManagerForScene3: TransitionManager


    override fun onCreateView(inflater: LayoutInflater, container: ViewGroup?, savedInstanceState: Bundle?): View? {
        return inflater.inflate(R.layout.fragment_transition, container, false)
    }

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        //从ViewGroup对象中获取Scene
        mScene1 = Scene(scene_root, container).apply {
            setEnterAction {
                Toast.makeText(activity, "enter action", Toast.LENGTH_LONG).show()
            }
            setExitAction {
                Toast.makeText(activity, "exit action", Toast.LENGTH_LONG).show()
            }
        }
        //从layout中获取Scene
        activity?.let {
            mScene2 = Scene.getSceneForLayout(scene_root, R.layout.scene2, it)
            mScene3 = Scene.getSceneForLayout(scene_root, R.layout.scene3, it)
            //获取xml中的Transition
            mTransitionManagerForScene3 = TransitionInflater.from(it).inflateTransitionManager(R.transition.scene3_transition_manager, scene_root)
        }
        //给RaidoGroup设置监听器
        view.findViewById<RadioGroup>(R.id.rg).setOnCheckedChangeListener(this)
    }


    override fun onCheckedChanged(group: RadioGroup?, checkedId: Int) {
        when (checkedId) {
            R.id.rb_1 -> {
                TransitionManager.go(mScene1)
            }
            R.id.rb_2 -> {
                TransitionManager.go(mScene2)
            }
            R.id.rb_3 -> {
                mTransitionManagerForScene3.transitionTo(mScene3)
            }
            R.id.rb_4 -> {
                TransitionManager.beginDelayedTransition(scene_root)
                val accentView = accent_view
                val params = accentView.layoutParams
                params.width = resources.getDimensionPixelSize(R.dimen.red_view_new_size)
                params.height = params.width
                accentView.layoutParams = params
            }
        }
    }
}
```

从上面的代码可以看到这里有两种方式获取Scene：

* 当前sceneRoot下就是一个Scene的View层级的时候，可以通过[Scene(sceneRoot, viewHierarchy)](https://developer.android.google.cn/reference/android/transition/Scene.html#Scene(android.view.ViewGroup,%20android.view.View))获取
* 其它的Scene通过[Scene.getSceneForLayout(sceneRoot, layoutId, context)](https://developer.android.google.cn/reference/android/transition/Scene.html#getSceneForLayout(android.view.ViewGroup,%20int,%20android.content.Context))从其layout中获取



然后从Transition和TransitionManager相关的内容在稍后进行讲解。

最后，就是给RadioGroup设置监听器了。





# Transition的使用

在上面的TransitionFragment中看到关于Transition和TransitionManager将在这里给大家详细的介绍了。

* **Transition就是在不同Scene或者是同一个Scene的布局改变实现动画过渡效果的类。**

* **你可以使用内置的一些Transition类，比如：[AutoTransition](https://developer.android.google.cn/reference/android/transition/AutoTransition.html)、[Fade](https://developer.android.google.cn/reference/android/transition/Fade.html)、[ChangeBounds](https://developer.android.google.cn/reference/android/transition/ChangeBounds.html)等，更多的就请查看官方文档了。**下面对一些常用的内置Transition类进行一下介绍：
  * [ChangeBounds](https://developer.android.google.cn/reference/android/transition/ChangeBounds.html): 当不同的Scene的View的位置或者大小发生变化后，ChangeBounds可以对它们进行动画。
  * [Fade](https://developer.android.google.cn/reference/android/transition/Fade.html): 这个Transition继承自Visibility类，Fade对那些出现、消失的View进行动画操作。
  * [AutoTransition](https://developer.android.google.cn/reference/android/transition/AutoTransition.html): 这是一个默认使用的Transition。它是一个TransitionSet。**它以Fade out、ChangeBounds、Fade in为顺序进行Transition动画操作**

* **你还可以进行自定义Transition。**



Transition的定义和Scene的获取是一样的，可以通过xml或者代码来进行定义

#### 在xml中定义Transition

首先，创建res/transition/目录

然后，新建transition xml文件到这个目录下

最后，在xml文件添加transition



changbounds_fadein_together.xml

```kotlin
<?xml version="1.0" encoding="utf-8"?>
<transitionSet xmlns:android="http://schemas.android.com/apk/res/android">
    <changeBounds />
    <fade android:fadingMode="fade_in">
        <targets>
            <target android:targetId="@id/transition_title" />
        </targets>
    </fade>
</transitionSet>
```

**transitionSet和前面讲的animatorSet是一样的可以包含多个Transition。**

**在Fade Transition中可以看到设置了target，target是用来指定Transition在某个对象上执行的**



在代码中获取Transition：

```kotlin
val transition = TransitionInflater.from(this).
inflateTransition(R.transition.changbounds_fadein_together)
```



#### 在代码中进行定义

下面是上面xml中定义的Transition在代码中对应的例子：

```kotlin
val changeBounds = ChangeBounds()
val fade = Fade(Fade.IN).apply {
    addTarget(R.id.transition_title)
}
val transitionSet = TransitionSet().apply {
    addTransition(changeBounds)
    addTransition(fade)
}
```





# TransitionManager的使用

在上面定义好了Transition，下面就该通过TransitionManager来使用Transition了。TransitionManager仍然定义在xml和代码中。



#### xml中进行定义

scene3_transition_manager.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<transitionManager xmlns:android="http://schemas.android.com/apk/res/android">
    <transition
        android:toScene="@layout/scene3"
        android:transition="@transition/changebounds_fadein_together" />
</transitionManager>
```

**其中toScene表示所有Scene转换到当前toScene时使用changebounds_fadein_together Transition。**

**如果加上fromScene，就表示从fromScene到toScene是使用changebounds_fadein_together Transition。从其它的Scene跳到toScene的时候使用默认的AutoTransition**



然后通过下面的代码获取xml中定义的TransitionManager：

```kotlin
mTransitionManagerForScene3 = TransitionInflater.from(activity).
inflateTransitionManager(R.transition.scene3_transition_manager, scene_root)
```



#### 代码中定义TransitionManager

```kotlin
val transitionManager = TransitionManager()
transitionManager.setTransition(mScene3, changebounds_fadein_together)
```



#### 使用TransitionManager进行Scene的转换

现在又回到TransitionFragment的代码中去：

看到为RaidoGroup设置监听器的代码中去。



**如果你不是在xml中进行TransitionManager的定义，那么就没有必要实例化一个TransitionManager出来。直接使用TransitionManager的go静态方法就行。你可以不传Transition进去然后使用默认AutoTransition。或者传入自己定义Transition。**其中，rb_1、rb_2就是使用的go静态方法。



**如果你在xml中定义了TransitionManager，那么你可以想rb_3获取TransitionManager进行Transition动画过渡**



rb_4中，这是不通过Scene使用Transition的情况。

**使用TransitionManager的beginDelayedTransition静态方法。可以不使用Scene。然后在后面通过代码对当前布局进行修改。**对于两个相差无几的布局就可以不同花大力气去定义两个布局来通过Scene来实现了。直接通过这样的方式来进行Transition的使用。



# 自定义Transition

通过继承自Transition可以自定义Transition。

自定义Transition主要有三个方法要进行重写：captureStartValues、captureEndValues、createAnimator。



#### captureStartValues

这个方法用来捕获starting Scene中所有View中我们想要捕获的属性。这些属性值将以键值对存储在TransitionValues中



#### captureEndValues

这个方法是用来捕获ending Scene中所有View中我们想要捕获的属性。这些属性值将以键值对存储在TransitionValues中



#### createAnimator

当starting Scene中的TransitionValues中的属性值与ending Scene中TransitionValues的属性值发生变化的时候，才会调用这个方法。在这个方法中就是定义自己的过渡动画了。



#### TransitionValues

这是一个用来为一个View保存其捕获的值、作用于它上面的Transition的数据结构。

它里面有三个主要的属性：

* view：用来保存这是哪个View的TransitionValues
* values：一个HashMap用来存储捕获的属性值。其中键通常是以**package_name:transition_name:property_name**这样的方式进行命名的。比如：com.anriku.jetpackdemo.animation:custom_transition:scale_x。
* mTargetedTransitions：这个属性是用来保存作用在当前View上的Transition的。



现在通过一个例子再在进行另外的一些内容的介绍：

```kotlin
@RequiresApi(Build.VERSION_CODES.KITKAT)
class CustomTransition : Transition {

    //在代码中实例化需要的构造器
    constructor() : super()
    //在xml中定义需要的构造器
    constructor(context: Context?, attrs: AttributeSet?) : super(context, attrs)

    companion object {
        private const val PROPNAME_SCALE_X = 
        "com.anriku.jetpackdemo.animation:custom_transition:scale_x"
        private const val PROPNAME_SCALE_Y = 
        "com.anriku.jetpackdemo.animation:custom_transition:scale_y"
    }


    override fun captureStartValues(transitionValues: 
                                    androidx.transition.TransitionValues) {
        captureValues(transitionValues)
    }

    override fun captureEndValues(transitionValues: androidx.transition.TransitionValues) 
    {
        captureValues(transitionValues)
    }

    private fun captureValues(transitionValues: TransitionValues) {
        val view = transitionValues.view
        transitionValues.values[PROPNAME_SCALE_X] = view.width
        transitionValues.values[PROPNAME_SCALE_Y] = view.height
    }

    override fun createAnimator(sceneRoot: ViewGroup, startValues: TransitionValues?, 
                                endValues: TransitionValues?): Animator? {
        startValues ?: return null
        endValues ?: return null

        val x = (startValues.values[PROPNAME_SCALE_X] as Int).toFloat() / 
        endValues.values[PROPNAME_SCALE_X] as Int
        val y = (startValues.values[PROPNAME_SCALE_Y] as Int).toFloat() / 
        endValues.values[PROPNAME_SCALE_Y] as Int

        val animSet = AnimatorSet()
        //Object为endValues中的view，因为这是将要显示的View，就对它进行动画操作
        val scaleXAnim = ObjectAnimator.ofFloat(endValues.view, "scaleX", x, 1f)
        val scaleYAnim = ObjectAnimator.ofFloat(endValues.view, "scaleY", y, 1f)

        return animSet.apply {
            duration = 1000
            play(scaleXAnim).with(scaleYAnim)
        }
    }
}
```

先看到构造器：

**在代码中进行实例化需要无参的构造器。在xml中定义的时候需要另外一个带有context、attrs的构造器。这个其实和自定义View有点像。**



然后在两个captureXXX方法中进行了View的width、height属性的捕获。



最后，进行动画操作的代码就不做详细解释了。这就是讲过的属性动画来实现View的压缩动画操作。



#### 自定义Transition的使用

在代码中自定义Transition的使用和内置的Transition是一样的。

**在xml中的使用稍有不同：**

```kotlin
<transition class="com.anriku.jetpackdemo.animation.CustomTransition" />
```

这里使用transition标签，然后在class属性中设置自定义的Transition的类。





# 总结

在这篇博客中讲解了的东西蛮清晰的。

主要是三个东西的使用：

* Scene的创建使用
* Transition的创建使用
* TransitionManager的使用



# 参考

[官方文档](https://developer.android.google.cn/training/transitions/)



*转载请注明链接*











