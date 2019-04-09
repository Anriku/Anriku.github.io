---
layout:     post
title:      Java AOP利剑之ASM，真正程度上的AOP
subtitle:   ASM的基本使用
date:       2019-04-09
author:     Anriku
header-img: img/2019_04_09_post.jpeg
catalog: true
tags:
    - ASM
    - AOP
---

在前面有一篇博客中讲了如何通过Java的动态代理来实现AOP编程。**之前也讲过其实动态代理并不能算代码层面的AOP编程，其实质是在运行时动态生成一个类然后以静态代理的方式来实现AOP编程的。**

**今天要讲的ASM是直接通过对代码进行修改没有使用代理来进行AOP编程的。**



# ASM简介

这篇博客主要是将如何使用ASM来实现AOP的，ASM不是重点，因此ASM在这里就只做一些简单的介绍。

**ASM是一个用来`生成字节码`或者`修改字节码`的开源库。[官网地址](https://asm.ow2.io)**



**`org.objectweb.asm`包是一些基本的ASM类型，比如ClassReader、ClassWritter都在这里面。依赖于ASM的`访问者模式`可以很轻易的就在ClassReader、ClassWritter之间添加新的访问者进行字节码的修改**

**`org.objectweb.asm.tree`包下类主要的作用是将输入的字节码以`树的形式`进行保存，这样做好处是可以很清晰地在对应的节点进行字节码的修改，用着也比直接中间插入访问者方便。**

**简单来说，第一种在ClassReader、ClassWritter的方式类似于xml的sax解析；而使用树形式则类似于dom解析。**

这篇博客只会用到上面两个包下的类，其它的包有兴趣的可以自己去看一下这里就不一一介绍了。



更详细ASM的入门可以看[Instrumenting Java Bytecode with ASM](http://web.cs.ucla.edu/~msb/cs239-tutorial)，里面介绍的例子是使用的ClassReader、ClassWritter之间夹访问者的方式。

**本篇博客使用的树节点的方式进行字节码的修改实现AOP。**



# 准备

首先去[官网的这个地方](https://repository.ow2.org/nexus/content/repositories/releases/org/ow2/asm/)的**asm和asm-tree目录下下载7.1版本的jar包。**



# 代码结构

![code_structure](https://images-1254261164.cos.ap-chengdu.myqcloud.com/asm_code_structure.png)

**common包下的代码就是前面中common包下一样的代码这里就不介绍了。**

**asm包下的代码会对LinuxPC、LinuxService的字节码进行修改，进行ShadowSocks属性的添加以及代理的执行。**



## common

#### pc

```java
package common.pc;

/**
 * 代表一台个人电脑
 */
public interface PC {

    void searchByGoogle();

}
```

```java
package common.pc;

/**
 * Linux系统的个人电脑
 */
public class LinuxPC implements PC {

    @Override
    public void searchByGoogle() {
        System.out.println("Searching with Google by LinuxPC");
    }
    
}
```



#### service

```java
package common.service;

/**
 * 代表一台服务器
 */
public interface Service {
    void searchByGoogle();
}
```

```java
package common.service;

/**
 * 一台Linux服务器
 */
public class LinuxService implements Service {
    @Override
    public void searchByGoogle() {
        System.out.println("Searching with Google by LinuxService");
    }
}
```



#### ShadowSocks

```java
package common;

/**
 * 现实中ShadowSocks是要分平台的，这里为了方便就想成全平台用一个ShadowSocks。总之就想成服务端实现代理的一个软件吧。
 * 而且现实中不仅服务端需要安装ShadowSocks，客户端也是需要的。这里客户端也忽略掉。
 */
public class ShadowSocks {

    public void startProxy() {
        System.out.println("start proxy");
    }

    public void stopProxy() {
        System.out.println("stop proxy");
    }
}
```



## asm

```java
package asm;

import common.ShadowSocks;
import org.objectweb.asm.*;
import org.objectweb.asm.tree.*;
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;
import java.util.List;
import static org.objectweb.asm.Opcodes.*;

public class InstallShadowSocks {

    private static final String SHADOWSOCKS_DESCRIPTER = Type.getDescriptor(ShadowSocks.class);
    private static final String SHADOWSOCKS = Type.getInternalName(ShadowSocks.class);
    private static String FIELD_OWNER = "";

    public static void main(String[] args) {
        FIELD_OWNER = args[1].substring(0, args[1].length() - 6);
        // 其中arg[0]是源字节码文件，args[1]是目标字节码文件
        installShadowSocks(args[0], args[1]);
    }

    /**
     * 进行ShadowSocks的安装
     * @param src 源字节码
     * @param dst 目标字节码
     */
    public static void installShadowSocks(String src, String dst) {
        FileInputStream fis = null;
        FileOutputStream fos = null;
        try {
            byte[] outputByteCode;
            fis = new FileInputStream(src);
            // ClassReader读入字节码
            ClassReader cr = new ClassReader(fis);
            // ClassNode将字节码以节点树的形式表示
            ClassNode cn = new ClassNode(ASM7);
            // SKIP_FRAMES用于避免访问帧内容，因为改变字节码的过程中帧内容会被改变，比如局部变量、操作数栈都可能改变。
            cr.accept(cn, ClassReader.SKIP_FRAMES);

            // 进行ShadowSocks属性的添加
            addShadowSocksField(cn.fields);

            for (MethodNode methodNode : cn.methods) {
                if (methodNode.name.equals("<init>")) {
                    // 构造器中对ShadowSocks属性进行初始化
                    initShadowSocksField(methodNode);
                } else if (methodNode.name.equals("searchByGoogle")) {
                    // searchByGoogle方法中添加ShadowSocks的调用
                    addShadowSocksExecute(methodNode);
                }
            }
            // COMPUTE_FRAMES表示ASM会自动计算所有内容，visitFrame和visitMaxs方法都会被忽略掉
            // 还有一个COMPUTE_MAXS是会自定计算局部变量表和操作数栈的大小，visitMaxs会被忽略掉。
            ClassWriter cw = new ClassWriter(ClassWriter.COMPUTE_FRAMES);
            cn.accept(cw);
            // 生成的字节码写入目标文件中
            outputByteCode = cw.toByteArray();
            fos = new FileOutputStream(dst);
            fos.write(outputByteCode);
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if (fis != null) {
                try {
                    fis.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            if (fos != null) {
                try {
                    fos.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }

    /**
     * 给src中的字节码添加ShadowSocks的属性
     * @param fields
     */
    private static void addShadowSocksField(List<FieldNode> fields) {
        boolean isHaveShadowSocksFiled = false;
        for (FieldNode fieldNode : fields) {
            if (fieldNode.desc.equals(SHADOWSOCKS_DESCRIPTER)) {
                isHaveShadowSocksFiled = true;
                break;
            }
        }

        if (!isHaveShadowSocksFiled) {
            fields.add(new FieldNode(ACC_PRIVATE, "shadowsocks", SHADOWSOCKS_DESCRIPTER, null, null));
        }
    }

    /**
     * 在构造方法中对ShadowSocks属性进行初始化
     * @param methodNode 表示该字节码一个方法节点的值
     */
    private static void initShadowSocksField(MethodNode methodNode) {
        AbstractInsnNode[] nodes = methodNode.instructions.toArray();
        int length = nodes.length;
        // 初始化相关的字节码指令
        InsnList insnList = new InsnList();
        insnList.add(new VarInsnNode(ALOAD, 0));
        insnList.add(new TypeInsnNode(NEW, SHADOWSOCKS));
        insnList.add(new InsnNode(DUP));
        insnList.add(new MethodInsnNode(INVOKESPECIAL, SHADOWSOCKS, "<init>", "()V", false));
        insnList.add(new FieldInsnNode(PUTFIELD, FIELD_OWNER, "shadowsocks", SHADOWSOCKS_DESCRIPTER));

        methodNode.instructions.insertBefore(nodes[length - 1], insnList);
    }

    /**
     * 在searchByGoogle方法调用中进行ShadowSocks的startProxy和stopProxy调用。
     * @param methodNode 表示该字节码一个方法节点的值
     */
    private static void addShadowSocksExecute(MethodNode methodNode) {
        AbstractInsnNode[] nodes = methodNode.instructions.toArray();
        int length = nodes.length;
        // searchByGoogle方法前面添加上ShadowSocks的startProxy方法调用
        InsnList startInsnList = new InsnList();
        startInsnList.add(new VarInsnNode(ALOAD, 0));
        startInsnList.add(new FieldInsnNode(GETFIELD, FIELD_OWNER, "shadowsocks", SHADOWSOCKS_DESCRIPTER));
        startInsnList.add(new MethodInsnNode(INVOKEVIRTUAL, SHADOWSOCKS, "startProxy", "()V", false));
        methodNode.instructions.insertBefore(nodes[0], startInsnList);

        // searchByGoogle方法的后面加上ShadowSocks的stopProxy方法的调用
        InsnList endInsnList = new InsnList();
        endInsnList.add(new VarInsnNode(ALOAD, 0));
        endInsnList.add(new FieldInsnNode(GETFIELD, FIELD_OWNER, "shadowsocks", SHADOWSOCKS_DESCRIPTER));
        endInsnList.add(new MethodInsnNode(INVOKEVIRTUAL, SHADOWSOCKS, "stopProxy", "()V", false));
        methodNode.instructions.insertBefore(nodes[length - 1], endInsnList);
    }

}
```

字节码修改的过程这里就不详解了，重点注释已经写出来了。

**这里对字节码的修改主要使用了的org.objectweb.asm.tree包下的内容，比如说ClassNode、MethodNode、FieldNode都是这个包下的。**

**当然也有org.objectweb.asm包下的，主要就是ClassReader、ClassWriter用于对字节码输入以及输出。**



## 主包

```java
import common.pc.LinuxPC;
import common.service.LinuxService;

public class Main {

    public static void main(String[] args) {
        LinuxPC linuxPC = new LinuxPC();
        LinuxService linuxService = new LinuxService();
        linuxPC.searchByGoogle();
        linuxService.searchByGoogle();
    }
}
```

很简单就是基本的调用。

**然后主包下的两个jar包是在准备阶段的时候进行下载的。**



# 编译执行

然后在主包下依次输入下面的命令：

```shell
# 编译两种不同类型的国外电脑
javac common/pc/LinuxPC.java
javac common/service/LinuxService.java

# 对上面编译出来的字节码做一个备份
cp common/pc/LinuxPC.class common/pc/LinuxPC.class.bak
cp common/service/LinuxService.class common/service/LinuxService.class.bak

# 对修改字节码进行ShadowSocks添加并调用的类进行编译
javac -cp asm-tree-7.1.jar:asm-7.1.jar:. asm/InstallShadowSocks.java

# 执行InstallShadowSocks对LinuxPC、LinuxService字节码进行修改(从备份文件中读取修改的字节码方法对应的.class结尾的文件)
java -cp .:asm-7.1.jar:asm-tree-7.1.jar asm.InstallShadowSocks common/pc/LinuxPC.class.bak common/pc/LinuxPC.class
java -cp .:asm-7.1.jar:asm-tree-7.1.jar asm.InstallShadowSocks common/service/LinuxService.class.bak common/service/LinuxService.class

# 编译并调用Main
javac Main.java
java Main
```



执行之后可以看到如下执行结果：

```
start proxy
Searching with Google by LinuxPC
stop proxy
start proxy
Searching with Google by LinuxService
stop proxy
```



其实修改后的LinuxPC和LinuxService分别如下：

**修改后的LinuxPC.java:**

```java
package common.pc;

import common.ShadowSocks;

public class LinuxPC implements PC {

    private ShadowSocks shadowSocks;

    public LinuxPC() {
        shadowSocks = new ShadowSocks();
    }

    @Override
    public void searchByGoogle() {
        shadowSocks.startProxy();
        System.out.println("Searching with Google by LinuxPC");
        shadowSocks.stopProxy();
    }
}
```

**修改后的LinuxService.java:**

```java
package common.service;

import common.ShadowSocks;

public class LinuxService implements Service {
    
    private ShadowSocks shadowSocks;

    public LinuxService() {
        shadowSocks = new ShadowSocks();
    }

    @Override
    public void searchByGoogle() {
        shadowSocks.startProxy();
        System.out.println("Searching with Google by LinuxService");
        shadowSocks.stopProxy();
    }
}
```



**可以看到使用ASM这里没有用静态代理或动态代理，而是通过直接修改字节码来实现了不同类型的类相同功能的添加。**



# 总结

本篇博客就一个目标**通过ASM使用树节点的形式来实现AOP编程**

然后就是ASM修改字节码的两种方式：

* 借助于访问者模式，在ClassReader、ClassWritter之间添加自己实现的ClassVisitor来实现。可以参考[Instrumenting Java Bytecode with ASM](http://web.cs.ucla.edu/~msb/cs239-tutorial)中的例子
* 通过树节点的形式来进行字节码修改的实现。本篇博客使用的方式。



然后两个方式的优缺点：

* 添加ClassVisitor的方式比较简便，而且只用引org.objectweb.asm这一个jar包就行。但是修改起来逻辑不太清晰。
* 通过树节点的方式，类、属性、方法等都被抽象为一个个树节点。要对某个方法、某个属性进行修改十分的清新，而且更好控制。不过需要引用org.objectweb.asm和org.objectweb.asm.tree两个jar，稍微麻烦点。



*转载请注明链接*