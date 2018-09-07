---
layout:     post
title:      Android使用动画进行Fragment、Activity的切换
subtitle:   Android使用动画进行Fragment、Activity的切换
date:       2018-09-07
author:     Anriku
header-img: img/2018_09_07_post.jpeg
catalog: true
tags:
    - Android
    - 动画
---

我们知道有些Fragment、Activity的跳转花时间比较长。这时候通过给跳转加上一些动画效果可以让用户感受不到这样等待，给用户很好的应用体验。个人觉得PayPal的动画效果就做得蛮好的。



# 给Fragment的转换添加动画

这里就难得想例子了。直接吧官方给的例子修改的一下进行讲解(官方的例子是一个用翻转动画实现Fragment切切换的例子)：

先定义好四个Animator。分别是:

* enter_animator: 这个动画将作用于将被添加的Fragment中的View
* exit_animator: 这个动画作用于将会被移除的Fragemnt中的View
* pop_enter_animator: 这个动画作用于**FragmentManager#popBackStack()方法调用或者back键按后**，将被添加的Fragment中的View
* pop_exit_animator: 这个动画作用于**FragmentManager#popBackStack()方法调用或者back键按后**，将被移除的Fragment中的View



enter_animtor.xml

```xml
<set xmlns:android="http://schemas.android.com/apk/res/android">
    <objectAnimator
        android:duration="0"
        android:propertyName="alpha"
        android:valueFrom="1.0"
        android:valueTo="0.0" />
    
    <objectAnimator
        android:duration="@integer/card_flip_full_time"
        android:interpolator="@android:interpolator/accelerate_decelerate"
        android:propertyName="rotationY"
        android:valueFrom="180"
        android:valueTo="0" />

    <objectAnimator
        android:duration="0"
        android:propertyName="alpha"
        android:startOffset="@integer/card_flip_half_time"
        android:valueFrom="0.0"
        android:valueTo="1.0" />
</set>
```

这里定义了一个属性AnimatorSet，这个set中有三个ObjectAnimator。

第一个是在开始的时候，就将将要出现的Fragment中的View的透明度设置为透明。

第二个是对将要出现的Fragment中的View进行rotationY的变换。

第三个是在动画转换到一半的时候将将要出现的Fragment中的View的透明设置为完全不透明



exit_animator.xml

```xml
<set xmlns:android="http://schemas.android.com/apk/res/android">
    <objectAnimator
        android:duration="@integer/card_flip_full_time"
        android:interpolator="@android:interpolator/accelerate_decelerate"
        android:propertyName="rotationY"
        android:valueFrom="0"
        android:valueTo="-180" />

    <objectAnimator
        android:duration="1"
        android:propertyName="alpha"
        android:startOffset="@integer/card_flip_half_time"
        android:valueFrom="1.0"
        android:valueTo="0.0" />
</set>
```

其中，第一个ObjectAnimator是对将要消失的Fragment中的View进行rotaionY的变换。

第二个是在动画执行到一半的时候对将要的消失的Fragment中的View透明度转换为透明。



后面两个动画，我就不再进行详细的介绍了。相当于前面这两个动画的逆过程。



pop_enter_animator.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android">
    <objectAnimator
        android:duration="1"
        android:propertyName="alpha"
        android:valueFrom="1.0"
        android:valueTo="0.0" />

    <objectAnimator
        android:duration="@integer/card_flip_full_time"
        android:interpolator="@android:interpolator/accelerate_decelerate"
        android:propertyName="rotationY"
        android:valueFrom="-180"
        android:valueTo="0" />

    <objectAnimator
        android:duration="1"
        android:propertyName="alpha"
        android:startOffset="@integer/card_flip_half_time"
        android:valueFrom="0.0"
        android:valueTo="1.0" />
</set>
```



pop_exit_animator.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android">
    <objectAnimator
        android:duration="@integer/card_flip_full_time"
        android:interpolator="@android:interpolator/accelerate_decelerate"
        android:propertyName="rotationY"
        android:valueFrom="0"
        android:valueTo="180" />

    <objectAnimator
        android:duration="1"
        android:propertyName="alpha"
        android:startOffset="@integer/card_flip_half_time"
        android:valueFrom="1.0"
        android:valueTo="0.0" />
</set>
```





下面是Activity的xml：

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    tools:context=".animation.CardFlipActivity">

    <Button
        android:id="@+id/btn"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="fragment flip"
        android:textAllCaps="false" />

    <FrameLayout
        android:id="@+id/container"
        android:layout_width="match_parent"
        android:layout_height="match_parent">

    </FrameLayout>

</LinearLayout>
```

Activity的代码：

```kotlin
class CardFlipActivity : AppCompatActivity() {

    //是否现在背面(也就是第二个Fragment)
    private var mShowingBack = false

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_card_flip)

        if (savedInstanceState == null) {
            supportFragmentManager.beginTransaction()
                    .add(R.id.container, CardFrontFragment())
                    .commit()
        }
        
        btn.setOnClickListener {
            flip()
        }
    }

    private fun flip() {
        if (mShowingBack) {
            supportFragmentManager.popBackStack()
            mShowingBack = false
            return
        }
        mShowingBack = true
        supportFragmentManager
                .beginTransaction()
                //为Fragment转换设置动画
                .setCustomAnimations(R.animator.enter_animator,
                        R.animator.exit_animator,
                        R.animator.pop_enter_animator,
                        R.animator.pop_exit_animator)
                .replace(R.id.container, CardBackFragment())
                .addToBackStack(null)
                .commit()
    }

    override fun onBackPressed() {
        mShowingBack = false
        super.onBackPressed()
    }

    /**
     * 正面Fragment(第一个Fragment)
     */
    class CardFrontFragment : Fragment() {
        override fun onCreateView(inflater: LayoutInflater, container: ViewGroup?, 
                                  savedInstanceState: Bundle?): View? {
            return inflater.inflate(R.layout.fragment_card_front, container, false)
        }
    }

    /**
     * 背面Fragment(第二个Fragment)
     */
    class CardBackFragment : Fragment() {
        override fun onCreateView(inflater: LayoutInflater, container: ViewGroup?, 
                                  savedInstanceState: Bundle?): View? {
            return inflater.inflate(R.layout.fragment_card_back, container, false)
        }
    }
}
```

很简单这里我就不详细讲了。

**重点就是flip方法中为Fragment切换设定动画的方法。那个四个参数可以分别对应动画去看。然后不太清楚可以直接跳进行看。之前在四个动画的介绍中已经说了，这里我就不太想再说了。**



没错为Fragment设置动画就是这么easy！！！



# 给Activity的转换添加过渡动画

为Activity设置动画仍然很简单。

为Activity将借助于Transition来进行实现。前面已经讲了Transition了。这里就直接通过例子来进行讲解了。



#### 定义Activity过渡动画

定义Activity的过渡动画仍然可以在xml和代码中进行定义。

**在styles.xml定义的过渡动画会对所有的Activity使用。在代码中进行定义的过渡动画只会对被定义的Activity使用**



下面是在styles.xml中定义的例子：

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>

    <style name="AppTheme" parent="Theme.AppCompat.Light.NoActionBar">
        <!-- Customize your theme here. -->
        <item name="colorPrimary">@color/colorPrimary</item>
        <item name="colorPrimaryDark">@color/colorPrimaryDark</item>
        <item name="colorAccent">@color/colorAccent</item>

        <!--这个属性用于启用Activity过渡动画-->
        <item name="android:windowActivityTransitions">true</item>

        <!--下面两个属性设定Activity进入、离开的过渡动画-->
        <item name="android:windowEnterTransition">@android:transition/explode</item>
        <item name="android:windowExitTransition">@android:transition/explode</item>

        <!--下面两个属性定义Activity进入、离开时共享的控件的转移动画-->
        <item name="android:windowSharedElementEnterTransition">@transition/change_image_transform
        </item>
        <item name="android:windowSharedElementExitTransition">@transition/change_image_transform
        </item>
    </style>
</resources>
```

**其中给android:windowEnterTransition、android:windowExitTransition这个两个动画需要继承自Visibility才行。**比如说：Explode、Slide、Fade



下面是在代码中对应的定义：

```kotlin
with(window){
    requestFeature(Window.FEATURE_CONTENT_TRANSITIONS)

    enterTransition = Slide(Gravity.BOTTOM)
    exitTransition = Slide(Gravity.TOP)

    sharedElementEnterTransition = 
    TransitionInflater.from(this@FirstAnimationActivity)
            .inflateTransition(R.transition.change_image_transform)
    sharedElementExitTransition = 
    TransitionInflater.from(this@FirstAnimationActivity)
            .inflateTransition(R.transition.change_image_transform)
}
```

其中change_image_transform.xml定义如下：

```xml
<?xml version="1.0" encoding="utf-8"?>
<transitionSet xmlns:android="http://schemas.android.com/apk/res/android">
    <changeBounds />
    <changeImageTransform />
</transitionSet>
```

这个很简单就不介绍了。



#### 使用指定的Transition来启动不带共享View的Activity

下面是FirstAnimationActivity的xml：

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".animation.FirstAnimationActivity">

    <Button
        android:id="@+id/btn"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:text="animate to second"
        android:textAllCaps="false"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

    <ImageView
        android:id="@+id/iv"
        android:layout_width="0dp"
        android:layout_height="0dp"
        android:scaleType="centerCrop"
        android:src="@drawable/my_animstate_drawable"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintTop_toBottomOf="@id/btn" />

</androidx.constraintlayout.widget.ConstraintLayout>
```



通过定义好的Transition来启动Activity就只需要想下面一样进行定义：

```kotlin
class FirstAnimationActivity : AppCompatActivity() {

    @RequiresApi(Build.VERSION_CODES.LOLLIPOP)
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_first_animation)

        btn.setOnClickListener{
            startActivity(Intent(this, SecondAnimationActivity::class.java),
                    ActivityOptionsCompat.makeSceneTransitionAnimation(this).toBundle())
        }
    }
}
```

是不是很简单就是加上ActivityOptionsCompat.makeSceneTransitionAnimation(this).toBundle()作为startActivity的参数就行了。



#### 使用Transition来启动带有单个共享View的Activity

这个也很简单，这里讲以共享一个ImageView的来作为例子。



**定义共享View的方式有两种：**

* 在xml中通过transitionName来进行定义。
* 在代码中通过ViewCompat.setTransitionName()来动态进行设置

**一般如果View是在xml中进行定义的就用第一种方式，如果View是在代码中进行定义的就用第二种。**



##### 在xml中定义



下面是FirstAnimationActivity中修改的代码**(其中xml仍然是上面的xml)**：

```kotlin
class FirstAnimationActivity : AppCompatActivity() {

    @RequiresApi(Build.VERSION_CODES.LOLLIPOP)
    override fun onCreate(savedInstanceState: Bundle?) {
...
        btn.setOnClickListener{
            startActivity(Intent(this, SecondAnimationActivity::class.java),
                    ActivityOptionsCompat.makeSceneTransitionAnimation(this,
                            iv, "iv").toBundle())
        }
    }
}
```

这个给ActivityOptionsCompat的makeSceneTransitionAnimation加了两个参数，**第一个是要当前layout中要共享的View，第二个是在另外的Activity中对应的View的transitionName。**



下面是SecondAnimationActivity中的xml：

```kotlin
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/cl"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".animation.SecondAnimationActivity">

    <TextView
        android:id="@+id/tv"
        style="?android:textAppearanceMedium"
        android:layout_width="0dp"
        android:layout_height="0dp"
        android:background="#a6c"
        android:gravity="center"
        android:text="Hello, this is activity animation"
        app:layout_constraintBottom_toTopOf="@id/iv"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

    <ImageView
        android:id="@+id/iv"
        android:layout_width="0dp"
        android:layout_height="0dp"
        android:scaleType="center"
        android:src="@drawable/my_animstate_drawable"
        android:transitionName="iv"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintTop_toBottomOf="@id/tv" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

**这里需要注意的有一个地方就是ImageView中的transitionName，这个就是前面makeSceneTransitionAnimation的新增的第二个参数。**



##### 在代码中定义



在代码中定义一般是如果在代码中动态生成View的情况下进行使用的。**这里就不在代码中生成了。直接使用xml中的ImageView进行举例。**



下面是SecondAnimationActivity的代码：

```kotlin
class SecondAnimationActivity : AppCompatActivity() {
    
    @RequiresApi(Build.VERSION_CODES.LOLLIPOP)
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        setContentView(R.layout.activity_second_animation)
        ViewCompat.setTransitionName(iv, "iv")
    }
}
```

**像这样定义后，你就可以将xml中ImageView中的transitionName属性去掉了！！！**





#### 使用Transition来启动带有多个共享View的Activity

当我们有多个View需要在两个View中进行共享的时候该怎么办？其实很简单基本和单个的情况是一样的。

只有下面的的一点不同：



下面通过代码举个例子，具体的例子我就写了：

```kotlin
startActivity(Intent(this, SecondAnimationActivity::class.java),
        ActivityOptionsCompat.makeSceneTransitionAnimation(this,
        Pair(view1, "view1"),
        Pair(view2, "view2")...).toBundle())
```



# 总结

这篇博客主要就是两个内容。

* 一个是给Fragment的跳转设置动画，通过`setCustomAnimations`进行设置
* 然后就是为Activity设定跳转的过渡动画。这个重要的就是ActivityOptionsCompat的makeSceneTransitionAnimation静态方法的使用



*转载请注明链接*

