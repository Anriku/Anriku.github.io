---
layout:     post
title:      Mac使用XAMPP搭建web服务器、FTP服务器
subtitle:   服务器搭建
date:       2018-11-25
author:     Anriku
header-img: img/2018_11_25_post.jpeg
catalog: true
tags:
    - web
    - xampp
---

搭建web服务器作为这次的一个课程作业要完成。于是借此机会就随便记录一下Mac使用XMAPP搭建服务器的过程了。



# XAMPP下载、安装

XAMPP介绍这里就不多说了。看这篇博客的应该都很清楚。

[去这里将XAMPP下载下来](https://www.apachefriends.org/zh_cn/download.html)

![xampp_download](https://images-1254261164.cos.ap-chengdu.myqcloud.com/xampp_download.png)

其中需要注意带vm和不带vm的区别。**这里我们选择不带vm的。也就是在本机上进行操作。**

下载完一路next安装好。



# XAMPP目录总览

**安装好后的XAMPP的文件放在了/Applications/XAMPP下。**

![xampp_download](https://images-1254261164.cos.ap-chengdu.myqcloud.com/xampp_directory.png)

这个目录下其实只有xamppfiles、uninstall.app两个文件。**其它的文件是软连接链接到xamppfiles中的文件。链接文件权限前面带`l`表示是软连接，不带的为硬链接。不懂软连接硬链接的就请自行查询了。**



**其中这样设置软链接的好处是。之后我们进行服务器配置的时候不用跑到xamppfiles中去配置了。直接对这个软链接进行配置就行。**xamppfiles中所有前面提到的软件都有进行装在了这个文件夹下。



# Apache的配置

### 自定义项目目录

现在我们打开`/Applications/XAMPP/etc/httpd.conf`，其实它软链接指向的是xamppfiles下的apache的httpd.conf。

![http_conf](https://images-1254261164.cos.ap-chengdu.myqcloud.com/httpd_conf.png)

**这里将DocumentRoot改成自己的项目目录。下面的Direcotry标签是对目录进行对应的设置的。这里将下面的目录配置也改成对应项目目录。**

对Direcotry中的配置这里做一下简单的说明：

* **Options：是对本目录做一些特性配置。**如：Indexs表示在本目录下如果没有DirectoryIndex(e.g.,index.html)相关的文件就返回目录列表。其中DirectoryIndex也是在httpd.conf中进行设置的。其它配置请查看官网。
* **AllowOverride：表示是否读取这个目录下的.htaccess命令。**All表示读取，None表示忽略。更多配置查看官网。
* **Require：这个用于是否需要用户认证的。**all granted表示无条件允许访问；all denied相反；还有一个常用的就是valid-user，通常与AuthName、AuthType、AuthUserFile一同使用来进行用户认证。

更多命令、[官网等着你去看](http://httpd.apache.org/docs/trunk/mod/directives.html)。

OK、到这一步，前面没搞错的话！启动了XAMPP的Apache选项**(如果启动不了看看Server Events的报错)**，输入localhost应该可以看到你设置的目录列表了。



### VirtualHost的设置

**VirtualHost顾名思义就是虚拟主机的意思。主要是让一个物理主机可以作为多个虚拟主机来使用的。**比如：我想通过anriku.com的域名访问一个目录；然后通过anriku.top访问该主机的另一个目录。



首先，现在httpd.conf中将下面一行的注释去掉。

![include_vhosts](https://images-1254261164.cos.ap-chengdu.myqcloud.com/include_vhosts.png)

**其中Include就和C语言的Include差不多。是用来引入其它文件的配置的。**



之后到Include指定的目录etc/extra/httpd-vhosts.conf中进行相关的配置。

![virtual_host](https://images-1254261164.cos.ap-chengdu.myqcloud.com/virtual_host.png)

上面就设置了两个虚拟主机。**这里的虚拟主机的设置会覆盖掉httpd.conf中的设置。**



OK，一个Web服务器基本完成了。**但是这只是一能在局域网(LAN)下进行相互设备访问的一个服务器。如果想要在外网下进行本地服务器的访问，还需要找个路由器做NAT转换、或者直接使用阿里云、腾讯云等进行Web服务器的搭建。**这并不是本篇博客的重点。这里就不进行解绍了。这个很简单、理解的计算机网络我想可以很轻易的就进行外网服务器的搭建了。



下面是我的Apache配置完成后的效果图：

![apache_finish_set](https://images-1254261164.cos.ap-chengdu.myqcloud.com/apache_finish_set.png)



# ProFTPD的配置

### 自定义FTP目录

ProFTPD是用来设置ftp服务设置的。

首先，打开`/Applications/XAMPP/etc/proftpd.conf`文件。下面是一个配置图。

![proftpd_conf](https://images-1254261164.cos.ap-chengdu.myqcloud.com/proftpd_conf.png)

**黄色方框中的Direcotry、DefaultRoot改为自定义目录。`PS:注意这里有一个坑自定义目录中不能有空格。`**

**其中下面部分是创建了User、在使用ftp连接的时候就输入这个帐号进行登录。红色标记的是真正的密码。**



ProFTPD和Apache有些类似。其它的更多的配置这里就不介绍了。可以去[官方文档进行查询](http://www.proftpd.org/docs/)。



在配置的过程中可能会在XAMPP控制台的Server Events中发现下面的问题。

![proftpd_promble](https://images-1254261164.cos.ap-chengdu.myqcloud.com/local_set.png)

按照图片中的解释在/private/hosts中添加一下对对应的域名解析就行。我想是因为proftpd用到了本机名作为域名进行处理的问题造成的。



下面是我的ProFTP配置完成的效果图：

![profftp_finish_set](https://images-1254261164.cos.ap-chengdu.myqcloud.com/ftp_finish_set.png)



# 总结

Mysql在这里我就行不说了。只不过要注意的是如果本机已经装了MySql。那么有可能XAMPP中MySql启动不了。这时候有两种解决方案：

* 停止之前的MySql服务
* 修改XAMPP中MySql的端口



这篇博客简单介绍了通过XAMPP在Mac上进行Web服务器、FTP服务器的搭建。这些都是基本的搭建。**如果想更深入的了解，请看官方文档。**

Web服务器搭建主要做了三件事：

* 下载安装。相信搞计算机都没问题。
* 自定义项目目录。**(PS：如果要配置虚拟主机这一部分可以直接放在虚拟主机的配置上)**
* 虚拟主机的设置。



FTP服务器的搭建主要就是一个修改为自定义目录。**这里再次强调一下这个坑、自定义目录中不能有空格。**



# 参考

[Apache官方文档](http://httpd.apache.org/docs/trunk/)

[ProFTPD官方文档](http://www.proftpd.org/docs/)



