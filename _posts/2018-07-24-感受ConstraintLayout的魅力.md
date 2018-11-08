---
layout:     post
title:      感受ConstraintLayout的魅力
subtitle:   ConstraintLayout的使用
date:       2018-07-24
author:     Anriku
header-img: img/2018_07_24_post.jpg
catalog: true
tags:
    - Android
    - Android布局
    - ConstraintLayout
---

ConstraintLayout也出了一段时间了。最近好好的去了解了下这个布局。感觉这个布局非常的强大。因此想在这里和大家进行一下分享。话不多说，开动！



# ConstraintLayout的基本位置控制

在这之前我们先来说明一下一些基本的概念：

![ConstraintLayout基本概念说明](https://images-1254261164.cos.ap-chengdu.myqcloud.com/2018-11-08-164112.png)



**简单的来说，ConstraintLayout基本上都是通过parent以及空间的四个边界以及内容的Baseline来进行位置、大小的控制的。**一共的布局方式有多少种我们计算一个：

```mathematica
2*2(表示Top和Bottom组合)+2*2(Left和Right组合)+2*2(Start和End组合)+1(Baseline只能和Baseline组合)
= 13种组合
```



那么，ConstaintLayout是如何进行位置的控制呢？很简单。举个例子

> **layout_constraintLeft_toRightof**就表示**当前控件的左边边界**在我们**属性值所表示的控件的右边界**右边。其它属性依次类推。**这里要注意的是我可以用parent表示直接父ContraintLayout，这样就不必给ContraintLayout指明id了。**



下面这张图就是通过位置控制得到一个布局：

![ConstraintLayout位置控制示例](https://images-1254261164.cos.ap-chengdu.myqcloud.com/2018-11-08-164113.png)

下面是xml布局：

```xml
<?xml version="1.0" encoding="utf-8"?>
<android.support.constraint.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <Button
        android:id="@+id/bt1"
        android:layout_width="100dp"
        android:layout_height="200dp"
        android:text="Button1"
        android:textAllCaps="false"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

    <Button
        android:id="@+id/bt2"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Button2"
        android:textAllCaps="false"
        app:layout_constraintBaseline_toBaselineOf="@id/bt1"
        app:layout_constraintLeft_toRightOf="@id/bt1" />


</android.support.constraint.ConstraintLayout>
```

> 这里我们还得说一下在ConstraintLayout中居中的实现。看上上面的代码**bt1**是居中。通过上面**四个方向控制**就令其在ConstraintLayout布局的中间了。当然不光是可以居ConstraintLayout的中间。**简单的来说，是上面四个方向所包围的返回的中间**

这是居中的四个方向的控制：

```xml
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintTop_toTopOf="parent"
```



> 还有就是我们通过**layout_constraintBaseline_toBaselineOf**这个属性来让两个Button的Baseline进行了对其。具体的效果图如上。



# ConstraintLayout的margin控制

![ConstraintLayout的margin控制](https://images-1254261164.cos.ap-chengdu.myqcloud.com/2018-11-08-164114.png)

![ConstraintLayout的goneMargin控制](https://images-1254261164.cos.ap-chengdu.myqcloud.com/2018-11-08-164115.png)

> 上面两张图。**第二张是第一张的Button1的可见性变为了gone的效果。Button2与父布局左边的边距是没有变的。这里都是200dp。**
>
> 
>
> 那这是怎么实现的呢？这个是ConstraintLayout的一个属性**layout_goneMarginLeft**来实现的。这属性是在**此控件(使用这个属性的控件)`左边相对`的控件的可见性为gone的时候才有效的。**其它属性依次类推。



Now！Show me the code:

```xml
<?xml version="1.0" encoding="utf-8"?>
<android.support.constraint.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <Button
        android:id="@+id/bt1"
        android:layout_width="100dp"
        android:layout_height="wrap_content"
        android:text="Button1"
        android:textAllCaps="false"
        android:visibility="gone"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

    <Button
        android:id="@+id/bt2"
        android:layout_width="100dp"
        android:layout_height="wrap_content"
        android:layout_marginLeft="100dp"
        android:text="Button2"
        android:textAllCaps="false"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintLeft_toRightOf="@id/bt1"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_goneMarginLeft="200dp" />

</android.support.constraint.ConstraintLayout>
```

> 在上面的代码中，我们可以看到button2的**layout_goneMarginLeft**的值是**layout_marginLeft的值加上Button1的width**这样实现Button2在Button1gone和visible两种情况下的位置不变的。



# ConstraintLayout更高级的位置控制

### ConstraintLayout位置比例控制

前面基本的位置控制，我们知道了如何使控件进行居中。现在我们不像让其在中间了。为了美观，我们想让这个控件的宽高都在黄金比例上。**这里普及下黄金比例，这是一个`(√5 - 1) / 2`的无理数。约为`0.618:1`**。



这时候我们就可以通过**layout_constraintHorizontal_bias**和**layout_constraintVertical_bias**两个属性来控制。**注意的是这个两个属性要在该控件的左右或者上下都有相对位置控制的时候才有效**



下面是我们的黄金比例图：

![ConstraintLayout的比例控制](https://images-1254261164.cos.ap-chengdu.myqcloud.com/2018-11-08-164116.png)



Code在这里：

```xml
<?xml version="1.0" encoding="utf-8"?>
<android.support.constraint.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">
    
    <Button
        android:id="@+id/bt_test"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Button"
        android:textAllCaps="false"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintHorizontal_bias="0.618"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintVertical_bias="0.618" />

</android.support.constraint.ConstraintLayout>
```



###ConstraintLayout圆形位置的控制

这里我直接拿官方的图来做说明：

![ConstraintLayout的圆形位置控制](https://images-1254261164.cos.ap-chengdu.myqcloud.com/2018-11-08-164120.png)

这里的两个以两个控件的中点作为圆心来进行旋转。一共有**三个属性**用来实现这样的位置控制：

* **layoutConstraintCircle**用来指定**相对位置控制的控件**
* **layoutConstraintCircleRadius**用来指定两个控件**中点间的距离**。也就是圆的半径
* **layoutConstraintCircleAngle**用来指定**当前控件**相对于**相对的控件**旋转的角度。**0度当前控件刚好在相对控件的上方**



下面是Button2相对Button旋转45度并且它们中点间的距离为80dp的效果图：

![ConstraintLayout的圆形位置控制](https://images-1254261164.cos.ap-chengdu.myqcloud.com/2018-11-08-164121.png)



Code如下：

```xml
<?xml version="1.0" encoding="utf-8"?>
<android.support.constraint.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <Button
        android:id="@+id/bt1"
        android:layout_width="100dp"
        android:layout_height="wrap_content"
        android:text="Button1"
        android:textAllCaps="false"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

    <Button
        android:id="@+id/bt2"
        android:layout_width="100dp"
        android:layout_height="wrap_content"
        android:text="Button2"
        android:textAllCaps="false"
        app:layout_constraintCircle="@id/bt1"
        app:layout_constraintCircleRadius="80dp"
        app:layout_constraintCircleAngle="45"/>

</android.support.constraint.ConstraintLayout>
```



# ConstraintLayout控件大小控制

**首先强调一点是在ConstraintLayout布局中的控件不推荐使用match_parent。在这里面通过match_constraint`(xml用0dp表示)`+控件的四个方向的控制的实现match_parent。`下面的不加说明使用0dp时都是有四个宽或高的方向控制的`**



### 当控件的宽高为wrap_content时的大小控制

此时通过

```
minWidth
maxWidth
minHeight
maxHeight
```

这四个属性来进行大小的控制。



### 当控件的宽高为match_constraint(i.e. ,0dp)时的大小控制

此时通过

```
layout_constraintWidth_min
layout_constraintWidht_max
layout_constraintHeight_min
layout_constraintHeight_max
```

这四个属性来进行控制



### 百分比大小控制

要进行比例控制控件要有下面的条件

* 控件的宽高为match_constraint(i.e. ,0dp)
* **这一项在1.1-beta1 and 1.1-beta2之后的版本可以不用设置了**。设置layout_constraintWidth_default="percent"或layout_constraintHeight_default="percent"
* 然后通过下面的属性进行宽高的比例控制**(值为0到1)**

```
//这两个属性时控件的宽高占四个方向所围面积的比例
layout_constraintWidth_percent
layout_constraintHeight_percent
```



### 宽高比例大小控制

这个控制的前面两个条件和**百分比控制**的前两个条件时一样的。

* 然后，通过下面的属性来进行宽高比例的控制

```
layout_constraintDimesionRatio
```

**其中属性值为，"width:height"。当宽高都设置为0dp的时候可以这样设置"W(H), width:height"。这样如果前面用W表示以`height为标准`宽进行变化。反之就不说了都懂的。**



前三个大小控制这里就不举例子了比较简单。这里对宽高比例控制去个例子如下：

![ConstraintLayout比例控制](https://images-1254261164.cos.ap-chengdu.myqcloud.com/2018-11-08-164122.png)



Code如下：

```xml
<?xml version="1.0" encoding="utf-8"?>
<android.support.constraint.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <Button
        android:id="@+id/bt_test"
        android:layout_width="0dp"
        android:layout_height="0dp"
        android:text="Button"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintDimensionRatio="H,1:1"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

</android.support.constraint.ConstraintLayout>
```



# ConstraintLayout的布局链(Chain)

ConstraintLayout的布局链是主要是通过下面两个属性**放到链头(链头就是布局链的第一个控件)**来进行设置

```
layout_constraintVertical_chainStyle
layout_constraintHorizontal_chainStyle
```

来进行控制的。**它们的取值有spread,spread_inside,packed。其中默认是spread**。下面将分别对三情况再加上另外的两种特殊情况进分析。



###  Spread Chain

下面是效果图：

![没有链头的布局链](https://images-1254261164.cos.ap-chengdu.myqcloud.com/2018-11-08-164128.png)



Code如下：

```xml
<?xml version="1.0" encoding="utf-8"?>
<android.support.constraint.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <Button
        android:id="@+id/bt1"
        android:layout_width="100dp"
        android:layout_height="wrap_content"
        android:text="Button1"
        android:textAllCaps="false"
        app:layout_constraintBottom_toTopOf="@id/bt2"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintVertical_chainStyle="spread" />

    <Button
        android:id="@+id/bt2"
        android:layout_width="100dp"
        android:layout_height="wrap_content"
        android:text="Button2"
        android:textAllCaps="false"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintTop_toBottomOf="@id/bt1" />

</android.support.constraint.ConstraintLayout>
```

这里我们只看构成链的方向的布局属性也就是只看垂直方向的属性。从上面的代码中可以看到**Button1Top在parent的Top下、Bottom在Button2的Top上；Button2的Top在Button1的Bottom下，Button2的Bottom在parentBottom上**这样就组成了一个布局链了。**这里我们将layout_constraintVertical_chainStyle设置为了spread，因此就形成了Spread Chain。当然我们可以在链头去掉这个属性。因为默认就是spread属性**



### Packed Chain

效果图如下：

![Packed Chain](https://images-1254261164.cos.ap-chengdu.myqcloud.com/2018-11-08-164130.png)

Code就不给了就是Speread Chain的**layout_contraintVertical_chainStyle的属性值改为packed**



### Spread Inside Chain

效果如下：

![Spread Inside Chain](https://images-1254261164.cos.ap-chengdu.myqcloud.com/2018-11-08-164131.png)





### Packed Chain with Bias

效果图如下：

![Packed Chain with Bias](https://images-1254261164.cos.ap-chengdu.myqcloud.com/2018-11-08-164132.png)

这个效果的设置也很简单，**就是前面提到的Packed Chain在链头加上`layout_constraintVertical_bias`这个属性来进行比例的设置**





### Weighted Chain

效果图如下：

![ConstraintLayout的占比链](https://images-1254261164.cos.ap-chengdu.myqcloud.com/2018-11-08-164133.png)



Code如下：

```xml
<?xml version="1.0" encoding="utf-8"?>
<android.support.constraint.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <Button
        android:id="@+id/btn1"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:text="Button1"
        android:textAllCaps="false"
        app:layout_constraintBottom_toTopOf="@id/btn2"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintVertical_weight="1" />

    <Button
        android:id="@+id/btn2"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:textAllCaps="false"
        android:text="Button2"
        app:layout_constraintBottom_toTopOf="@id/btn3"
        app:layout_constraintTop_toBottomOf="@+id/btn1"
        app:layout_constraintVertical_weight="2" />

    <Button
        android:id="@+id/btn3"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:textAllCaps="false"
        android:text="Button3"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintTop_toBottomOf="@id/btn2"
        app:layout_constraintVertical_weight="3" />

</android.support.constraint.ConstraintLayout>
```

从代码中可以看到这个的Weighted Chain**本质就是Spread Chain,只不过它们的高度都是match_constraint(i.e. ,0dp)。并且通过layout_contraintVertical_weight来进行比例的控制。水平方向的也可以此类推**



# 总结

关于ConstraintLayout大致的东西就这些了。这篇博客主要讲了三大部分：

* ConstraintLayout中的控件位置的控制
* ConstraintLayout中控件的大小控制
* ConstraintLayout中的布局链



# 参考

[官方ConstraintLayout文档](https://developer.android.com/reference/android/support/constraint/ConstraintLayout)



*转载请注明链接地址*