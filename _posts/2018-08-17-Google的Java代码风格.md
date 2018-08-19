---
layout:     post
title:      Google的Java代码风格
subtitle:   代码规范
date:       2018-08-17
author:     Anriku
header-img: img/2018_08_08_post.jpeg
catalog: true
tags:
    - Android
    - 代码规范
---

在本篇博客中，没做任何的说明的话。下面都是默认的规则：

* class: 指普通的Java类、enum类、接口以及注解
* member:包括内部类、域、方法、构造器等**所有除了初始化设置和注释的class顶层内容。**





# 源文件

### 文件名

Java的文件名有大小敏感的字母组成。并以.java为后缀



### 文件编码

以UTF-8进行编码



### 特殊字符

##### 空格

除了换行之外，在源文件中的任何位置都只能使用**ASCII horizontal space characte(0x20)，也就是ASCII的空格**



避免使用其它类型的空格、Tab键





