---
layout:     post
title:      Android中的Gradle分析
subtitle:   Gradle
date:       2018-07-19
author:     Anriku
header-img: img/2018_07_19_post.jpeg
catalog: true
tags:
    - Android
    - Gradle
---

Gradle是我们在Android开发中用来构建应用的工具。相信做Android开发的朋友们都不陌生。虽然大家从开始写Android的时候就和它一直相处的，其中的大部分人应该对Gradle如何构建Android的应用不太清楚吧！今天我想简单的通过一个Android应用来对Android中的Gradle相关的内容做一下说明。



# 创建一个Android应用

这个步骤大家都懂，所以略...



# 各个文件分析

首先，我们创建了一个叫做GradleTest的项目。下面的是文件架构以及标出了我们即将分析的文件,在Gradle脚本中没有被标注的不是那么重要最后一笔带过：

![gradle(1)](https://images-1254261164.cos.ap-chengdu.myqcloud.com/2018-11-08-164153.png)

#### 1. build.gradle(Project: GradleTest)

此文件是项目的构建文件。具体的分析留在后面。

#### 2. build.gradle(Module:app)

此文件是模块的构建文件。不同的模块都有自己的构建文件。

#### 3. gradle-wrapper.properties

这个文件是对gradle的下载、存储以及版本做配置的。

#### 4.settings.gradle

这个文件是用于对多个项目(模块)进行配置的。

#### 5.其它文件

* Proguard-rules.pro做混淆的时候用的
* gradle.properties是对项目进行配置的时候用的。比如对JVM内存进行控制等
* local.properties是对sdk以及ndk进行配置的

**下面给出一张各个文件在Project视图下的具体位置:**

![gradle(7)](https://images-1254261164.cos.ap-chengdu.myqcloud.com/2018-11-08-164154.png)







# gradle-wrapper.properties

上面四个重要文件中先对这个简单的文件做一下分析。下面是新建项目中该文件的截图：

![gradle(2)](https://images-1254261164.cos.ap-chengdu.myqcloud.com/2018-11-08-164155.png)



* 前面的四行表面的是第一运行的时候gradle下载并且存储的位置。默认是存储在用户目录下的.gradle/wrapper/dists目录中的。

![gradle(3)](https://images-1254261164.cos.ap-chengdu.myqcloud.com/2018-11-08-164156.png)



* 最后一行表面的是我们下载gradle的地址



# build.gradle(Project: GradleTest)

这个文件是对整个项目的构造做配置的。下面是新创建的项目的该文件的截图：

![gradle(4)](https://images-1254261164.cos.ap-chengdu.myqcloud.com/2018-11-08-164157.png)



* buildscript定义插件相关的内容

  * 在这个块的第一行有一个**ext.kotlin_version = '1.2.51'**gradle的属性设置。简单的说就是配置变量方便后面的使用。属性很多的时候可以一起放在**ext块**中进行管理

  * repositories表明插件在goole或jcenter两个地方下载。
  * dependencies表明下载的的插件。**注意上面的gradle是指的gradle插件，和之前提的graddle-wrapper.properites是不一样的。其中插件以及后面在子模块中添加的依赖要满足`'group:name:version'`的格式**

* allprojects对所有的项目和模块进行配置。这里表明所有的依赖要么从goole、要么从jcenter中下载。

* 最后一个是自定义的task用于删除项目目录中build目录生成的文件



# build.gradle(Module: app)

该文件是对app这个模块进行配置的。该文件的截图如下：

![gradle(6)](https://images-1254261164.cos.ap-chengdu.myqcloud.com/2018-11-08-164159.png)

* apply plugin是在该模块中应用的插件。这个插件与前面的根项目下的gradle中buildscript声明的插件相关
* 后面android块、dependences块都是应用的插件添加的配置、依赖。当然还可以添加自定义task
* android块
  * compileSdkVersion:与Android SDK相关。通常是最新的SDK
  * defaultConfig
    * applicationId:由域名和项目名组成。**每一个应用在应用商店中是独一无二的。**
    * minSdkVersion:应用兼容到最小的Android版本
    * targetSdkVersion:当设备的api级别在mSdkVersion和targetSdkVersion(包括在内)之间时。系统不启用任何兼容性行为。当高于targetSdkVersion的时候，会通过启动兼容性行为来确保应用以期望的方式工作。
    * versionCode:应用的版本号
    * versionName:版本名
    * testInstrumentationRunner:表明通过JUnit 4 test runner来进行应用测试
  * buildTypes:用来指定构建类型有debug和release两个类型
* dependencies块
  * testImplementation和androidTestImplementation都是和应用测试相关的依赖
  * 第一行**implementation fileTree(dir: 'libs', include: ['*.jar'])**添加模块中**libs**目录下的任何**jar包**作为依赖
  * **implementation+'group:name:version'**表示从jcenter或者google中获取依赖



# 总结

通过本篇Blog我们了解到gradle在构建Android应用中的整体架构。以及各个文件相应时干啥的。想要了解更多相关的内容。请到[官网](https://docs.gradle.org/current/userguide/userguide.html)



# 参考

[Building Android apps](https://guides.gradle.org/building-android-apps/?_ga=2.92453717.951381537.1532131873-369079375.1527669523)
