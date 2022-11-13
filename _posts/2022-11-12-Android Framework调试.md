---
layout:     post

title:      Android Java/Native系统源码调试

subtitle:   Android调试技巧

date:       2022-11-12

author:     Anriku

header-img: img/2022_11_12_post.jpeg

catalog: true

tags:

    - Android Framework调试
    - 编译Android系统源码
    - Android Native源码调试

---

上一篇最新的Blog定格在了19年5月13，算了算3年多没有更新自己的个人Blog。这3年里从学生变成了打工人，工作日太忙下班之后的那一点点时间以及周末的时间我都想投入在自己的爱好上面，因此一直都没有更新Blog。

最近回头看了一下之前的Blog，发现很多都是学习总结；内容虽然之前也是认真写的，但在现在看来还是有很多地方可以优化。本来有删掉的打算，不过想了想作为见证自己成长的记录也不错。今天这篇Blog以及以后的Blog都不打算做学习总结了，打算分享一些Android开发的技巧和个人思考。

个人认为入门任何一个编程方向之后，掌握**全链路调试技巧**应该是最重要的事。对于Android就是能够**对Framework的Java以及C++的系统源码调试**，搭建Framework调试环境最直接的方法就是编一个自己的Debug Rom。在这里很感谢weishu大佬[这篇Blog](https://zhuanlan.zhihu.com/p/24867284)提供的一些帮助。



# Android Framework调试的作用

Android Framework调试我认为有下面几个比较重要的作用：

* **有效地分析解决疑难问题**：大多数疑难问题其实难在解决问题需要阅读底层源码的逻辑，Framework调试可以帮忙我们有效进行断点分析。比如：我在业务中进行的抖音(抖音Lite)单Activity深浅模式探索以及落地。

* **高效地解决小的业务问题**：Framework调试也可以提高解决小的业务问题的效率。比如：一个View的背景颜色被改坏了但对这块业务也不熟悉，通过对View相关代码进行断点就可以很快地定位到。

* **通过调试分析感兴趣的Android底层原理提升自己**：别人的源码分析就像授人以鱼，清晰明了的好分析很少而且有些分析没有是一方面，效率不高且不直观我觉得是最头疼的；Framework的调试能力就像授人以渔，能够帮忙我们直观且高效地去阅读底层的源码，对任何地方有疑问都可以断点去实际运行分析，让阅读Framework层的源码特别是C++层的源码时不再无从释手。

总结下来就是Framework调试教给我们的是分析底层源码逻辑以及如何解决问题的能力，个人觉得这比掌握源码本身实现逻辑更重要，也就是可以不用知道源码实现的逻辑，但在真正遇到问题的时候我们能够有效地对其进行分析。想一想Android作为一个庞大的操作系统是无数的开发者一起共建的，一个人的时间有限，我们不太可能把其中的方方面面都搞清楚，掌握分析技巧然后解放我们的时间，花更多的精力去学习更低层更基础的东西会更好一点。



# Android Framework调试环境搭建

## 准备

* **选择源码**：目前很多APP都只支持ARM的处理器，x86的Mac跑ARM的虚拟机体验也是一言难尽，所以最好的选择是选一个自己测试机的开源系统源码(如果你用的是ARM的Mac，应该是可以直接跑ARM虚拟机的，可以直接选择AOSP进行编译)。我的测试机是一加7t，选择的是[LineageOS for OnePlus 7t](https://wiki.lineageos.org/devices/hotdogb/build)。

* **编译环境**：Virtual Box + 最新的Ubutnu

* **存储设备**：剩余空间尽量比对应系统源码官网给的空间大50G左右。LineageOS官网给的剩余空间推荐是200G，但我实际编译完成后源码加编译产物一共占了226G。源码太大，空间不够重新下载或者转移至大空间的硬盘很花时间。



## 系统源码的编译以及刷入

关于系统源码的编译以及刷入，网上的Blog以及官网都有比较详细的说明了，这里我就不进行赘述。主要说一下需要注意的一些事情，下载以及编译源码是一个很花时间的事情，避免下面我遇到过的坑应该可以省不少的时间。

* AOSP已经不再维护MacOS上的源码编译了，**最好用Linux编译**，Mac上会遇到很多问题。

* M.2的固态硬盘用雷电盒子以PCI的方式连接电脑没法连Virtual Box，需要买一个**USB-C**的盒子进行连接。

* 编译debug的ROM，我这边之前没有编译debug的ROM遇到调试的时候变量没法获取的问题。

* 把Linux的性能拉到最满。



# 调试Java层系统源码

下面通过一个简单的例子来说明如何断点Java层的系统源码。假设在业务中有一个View的背景不知道在哪里被改坏了，我们通过断点View的setBackgroundDrawable方法来进行修改代码的快速定位。



## 将对应源码放在工程目录下

我们可以把**Rom对应源码**的android.view.View**按照其包名**放在**src/java**下面。

![ ](https://images-1254261164.cos.ap-chengdu.myqcloud.com/copy_java_source_code_to_as.png)



## 在源码中直接设置断点调试

![ ](https://images-1254261164.cos.ap-chengdu.myqcloud.com/debug_java_source_code.png)



# 调试Native层系统源码

## 将源代码与带debug信息的so(vmlinux)放到工程目录下

- 将我们需要的native源码放置source下面，比如：这里放了art、frameworks、system对应的源码。

- 将带有debug信息的中间编译产物放到symbols目录下面，在Mac上你可以用objdump -h来查看其是否有debug相关的段(说到so很推荐大家去看一下**程序员的自我修养**这本书，配合**AidLearning**去实践学习很不错)。大多数情况下面两块应该就够了，额外的中间产物都可以往symbols目录下加：
  - /.../LineageOS/out/target/product/hotdogb/obj/KERNEL_OBJ/vmlinux
  - /.../LineageOS/out/target/product/hotdogb/symbols下面的so

![ ](https://images-1254261164.cos.ap-chengdu.myqcloud.com/copy_native_code_and_so_to_as.png) 

## 在AS中配置Native调试

- 将debug类型改成dual
- 将带debug信息的中间编译产物的路径配置到Symbol Directories下

![ ](https://images-1254261164.cos.ap-chengdu.myqcloud.com/config_native_debug.png)



## 在Native源码中设置断点进行调试

下图是对art虚拟机的Heap::AllocObjectWithAllocator进行断点调试的效果图。

![ ](https://images-1254261164.cos.ap-chengdu.myqcloud.com/debug_native_code.png)




