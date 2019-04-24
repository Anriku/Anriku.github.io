---
layout:     post
title:      ThreadLocal完全解析
subtitle:   ThreadLocal
date:       2019-04-24
author:     Anriku
header-img: img/2019_04_24_post.jpeg
catalog: true
tags:
    - ThreadLocal
    - 源码
---

熟悉Android消息机制的话，对ThreadLocal这个类应该都不陌生。**Android消息机制中的Looper就是通过ThrealLocal来实现为每个线程建立一个独立Looper的。**



# 简单的Demo

```java
public class ThreadLocalTest {

    public static void main(String[] args) {
        ThreadLocal<String> threadLocal = new ThreadLocal<>();
        // 给主线程设置ThreadLocal值
        threadLocal.set("I am in main thread");
        Thread thread = new Thread(new Runnable() {
            @Override
            public void run() {
                // 给子线程设置ThreadLocal值
                threadLocal.set("I am in sub thread");
                // 在子线程获取ThreadLocal值
                System.out.println(Thread.currentThread() + ":" + threadLocal.get());
            }
        });
        thread.start();
        // 等待子线程执行完
        try {
            thread.join();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        // 在主线程获取ThreadLocal值
        System.out.println(Thread.currentThread() + ":" + threadLocal.get());
    }
    
}
```

通过上面程序可以发现**ThreadLocal在不同的线程设置和获取值是不会相互影响的。**



# ThreadLocal是如何实现不同线程独立存储的

其实每个线程都一个叫做`threadLocals`的`ThreaLocal.ThreadLocalMap类型`成员属性。

深入源码可以发现ThreadLocalMap就是一个简易版的`Hash表`。这个Hash表`key`为`WeakReference<ThreadLocal<?>>`，`value`为`Object` 。**熟悉范性的同学都应该知道`ThreadLocal<任意类型>`都可以赋值给ThreadLocal<?>类型，相当于实现了`范性的多态`。**



ThreadLocal调用`set`方法的时候，最终都会`以当前ThreadLocal的弱引用为key`对应的`set的值为value`存入当前调用线程的`threadLocals`成员属性中去。

**因为如果在`不同线程`对`同一ThreadLocal`进行set方法调用，在set方法内获取的是`不同Thread的threadLocals成员属性`，因此达到了同一个ThreadLocal对象的在不同线程中进行值独立存储的要求。**



下面是Thread的部分源码：

```java
class Thread implements Runnable {
...
    ThreadLocal.ThreadLocalMap threadLocals = null;
...
}
```



# 最核心的类---ThreadLocalMap

前面已经说了，这就是一个简易版的在`Hash表`。**这里key为ThreadLocal的弱引用因此ThreadLocal是不会导致内存泄漏的。只要某个ThreadLocal的强引用没有了，GC时就ThreadLocalMap对应的ThreadLocal的弱引用也会被回收。**

**这个过程会产生`不新鲜值`也就是ThreadLocalMap中某个位置的`弱引用的值为null`，但其对应的value不为null。**

不新鲜值在每次set方法调用的时候一定会进行相应的检测清除，get方法只有在遇到不新鲜值的时候才会进行相应的清除。对应的算法就不进行详解了。



这个简易版的Hash表以`16`作为`初始容量`，然后`扩容因子`为`2 / 3`。以`2倍`进行扩容。**因此其容量始终是2的幂次方。这样的好处和HashMap一样的可以通过`与运算`轻松的获取存储的值该放在Hash表的哪个位置。**然后如果遇到Hash冲突，这里使用的是`索引加一法`进行冲突的解决。



下面是ThreadLocal.ThreadLocalMap的部分源码:

```java
public class ThreadLocal<T> {
	...
	static class ThreadLocalMap {
		...
		private Entry[] table;
		
		static class Entry extends WeakReference<ThreadLocal<?>> {
    	/** The value associated with this ThreadLocal. */
    	Object value;

    	Entry(ThreadLocal<?> k, Object v) {
      	super(k);
        value = v;
      }
    }
		...
	}
	...
}
```



# ThreadLocal的实现

```java
public class ThreadLocal<T> {
...
    public void set(T value) {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null)
            map.set(this, value);
        else
            createMap(t, value);
    }
    
    public T get() {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null) {
            ThreadLocalMap.Entry e = map.getEntry(this);
            if (e != null) {
                @SuppressWarnings("unchecked")
                T result = (T)e.value;
                return result;
            }
        }
        return setInitialValue();
    }
  

    ThreadLocalMap getMap(Thread t) {
        return t.threadLocals;
    }
...
}
```

通过上面部分源码应该很容易发现**ThreadLocal就是借助于Thread的`threadLocals`整个Hash表进行不同线程值的独立存储的。**





# InheritableThreadLocal---可以继承的ThreadLocal

在Thread的源码中应该可以发现其实Thread有两个ThreadLocal.ThreadLocalMap类。

下面是部分源码：

```java
class Thread implements Runnable {
...
    ThreadLocal.ThreadLocalMap threadLocals = null;

    ThreadLocal.ThreadLocalMap inheritableThreadLocals = null;
    
    public Thread() {
        init(null, null, "Thread-" + nextThreadNum(), 0);
    }
    
    private void init(ThreadGroup g, Runnable target, String name,
                      long stackSize, AccessControlContext acc,
                      boolean inheritThreadLocals) {
...
        if (inheritThreadLocals && parent.inheritableThreadLocals != null)
            this.inheritableThreadLocals =
                ThreadLocal.createInheritedMap(parent.inheritableThreadLocals);
...
    }
...

}
```

**threadLocals就是使用普通的ThreadLocal类用到的；inheritableThreadLocals这个成员属性主要就是用于继承创建当前Thread的父Thread中的inheritableThreadLocals。**

init方法是继承的具体实现。



下面是InheritableThreadLocal部分源码：

```java
public class InheritableThreadLocal<T> extends ThreadLocal<T> {

    protected T childValue(T parentValue) {
        return parentValue;
    }

    ThreadLocalMap getMap(Thread t) {
       return t.inheritableThreadLocals;
    }

    void createMap(Thread t, T firstValue) {
        t.inheritableThreadLocals = new ThreadLocalMap(this, firstValue);
    }
}
```

InheritableThreadLocal就是ThreadLocal的子类，很简单就是**使用线程的threadLocals改成了使用inheritableThreadLocals进行进行存储。**



# 总结

总的来说，ThreadLocal.ThreadLocalMap就是一个通过`索引加一法`解决冲突的Hash表。

然后每个Thread中有两个ThreadLocal.ThreadLocalMap类型的成员属性：

* threadLocals用于对`ThreadLocal`进行存储的。

* inheritableThreadLocals用于对`InheritableThreadLocal`进行存储的。其中inheritableThreadLocals会在Thread的init方法调用的时候继承父Thread的inheritableThreadLocals。