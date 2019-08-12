---
title: ThreadLocal简单理解
date: 2019-08-12 16:53:23
categories: 线程
tags: 
	- 线程
---
多线程不可避免需要处理状态，处理状态有三种方式：共享可变性、隔离可变性和纯粹不可变。使用ThreadLocal属于隔离可变性的一种方法，但是ThreadLocal使用不当又可能导致内存泄漏，下面简单介绍一下ThreadLocal。
<!-- more -->
> 本文基于JDK 1.8

## 介绍  
> This class provides thread-local variables.  These variables differ from their normal counterparts in that each thread that accesses one (via its {@code get} or {@code set} method) has its own, independently initialized copy of the variable.  {@code ThreadLocal} instances are typically private static fields in classes that wish to associate state with a thread (e.g.,a user ID or Transaction ID).  

> Each thread holds an implicit reference to its copy of a thread-local variable as long as the thread is alive and the {@code ThreadLocal} instance is accessible; after a thread goes away, all of its copies of thread-local instances are subject to garbage collection (unless other references to these copies exist).

核心意思是
> ThreadLocal 提供了线程本地的实例。它与普通变量的区别在于，每个使用该变量的线程都会初始化一个完全独立的实例副本。ThreadLocal 变量通常被private static修饰。当一个线程结束时，它所使用的所有 ThreadLocal 相对的实例副本都可被回收。

总的来说，ThreadLocal 适用于每个线程需要自己独立的实例且该实例需要在多个方法中被使用，也即变量在线程间隔离而在方法或类间共享的场景。

## 原理
从set方法入手  

```java
public void set(T value) {
	// 获取当前线程
    Thread t = Thread.currentThread();
    获取当前线程Map
    ThreadLocalMap map = getMap(t);
    // 如果map存在，则将当前线程对象t作为key，要存储的对象作为value存到map里面去
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
}
```  

set方法中一个核心对象是ThreadLocalMap，跟着源码看一下是什么  

```java
static class ThreadLocalMap {

    static class Entry extends WeakReference<ThreadLocal<?>> {
        Object value;

        Entry(ThreadLocal<?> k, Object v) {
            super(k);
            value = v;
        }
    }
   
    ... ignore
    
}
```  

根据源码可以看到，ThreadLocalMap是ThreadLocal的一个内部类。用Entry类来进行存储，我们的值都是存储到这个Map上的，key是当前ThreadLocal对象。  
当Map存在，直接获取线程Thread中获取。  

```java
ThreadLocalMap getMap(Thread t) {
    return t.threadLocals;
}
```  

可以看出，对象的引用实际是在Thread中。  
因此总结一下就是，Thread为每个线程维护了ThreadLocalMap这么一个Map，而ThreadLocalMap的key是LocalThread对象本身，value则是要存储的对象。
![图示](./原理.jpeg)  


## 内存泄露问题
### 为什么会内存泄漏
ThreadLocal在ThreadLocalMap中是以一个弱引用身份被Entry中的Key引用的，因此如果ThreadLocal没有外部强引用来引用它，那么ThreadLocal会在下次JVM垃圾收集时被回收。这个时候就会出现Entry中Key已经被回收，出现一个null Key的情况，外部读取ThreadLocalMap中的元素是无法通过null Key来找到Value的。如果当前线程的生命周期很长（比如线程池管理的线程），一直存在，那么其内部的ThreadLocalMap对象也会一直生存下去，这些null key就存在一条强引用链的关系一直存在：Thread --> ThreadLocalMap-->Entry-->Value，这条强引用链会导致Entry不会回收，Value不能被回收，导致内存泄漏。
![图示](./内存泄露.jpeg)  

### JDK的优化
当我们调用set() get() remove()，方法的时候，会去清理key为null 的Entry。（但是如果一个ThreadLocal在声明以后再也不调用以上三个方法，也是无法被优化清理的）    

```java
private int expungeStaleEntry(int staleSlot) {
    Entry[] tab = table;
    int len = tab.length;

    // expunge entry at staleSlot
    tab[staleSlot].value = null;
    tab[staleSlot] = null;
    size--;

    // Rehash until we encounter null
    Entry e;
    int i;
    for (i = nextIndex(staleSlot, len);
         (e = tab[i]) != null;
         i = nextIndex(i, len)) {
        ThreadLocal<?> k = e.get();
        if (k == null) {
            e.value = null;
            tab[i] = null;
            size--;
        } else {
            int h = k.threadLocalHashCode & (len - 1);
            if (h != i) {
                tab[i] = null;

                // Unlike Knuth 6.4 Algorithm R, we must scan until
                // null because multiple entries could have been stale.
                while (tab[h] != null)
                    h = nextIndex(h, len);
                tab[h] = e;
            }
        }
    }
    return i;
}
```

### 注意事项  
防止内存泄露，使用完需要调用ThreadLocal的remove()方法

## 常用方法
- get() 返回当前线程的此线程局部变量的副本中的值 
	1. 获取当前线程的属性：ThreadLocal.ThreadLocalMap threadLocals （即：一个map）
	2. map中获取线程存储的K-V Entry键值对
	3. 返回存储的变量
- set(T value) 将当前线程的此线程局部变量的副本设置为指定的值
- remove() 删除此线程局部变量的当前线程的值  

## 简单代码示例
```java 
public class TheadLocalExampleHolder {

    private static final ThreadLocal<User> USER_THREAD_LOCAL = new ThreadLocal<>();

    @Data
    static class User {
        private String userId;
        private String userName;
    }

    public static User get() {
        return USER_THREAD_LOCAL.get();
    }

    public static void set(User user) {
        USER_THREAD_LOCAL.set(user);
    }

    public static void remove() {
        if (USER_THREAD_LOCAL.get() != null) {
            USER_THREAD_LOCAL.remove();
        }
    }

}
```