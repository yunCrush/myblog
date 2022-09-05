---
description: 主要介绍两大典型的使用场景
---

# ThreadLocal的简单使用与原理

### 使用场景一

在使用工具类如SimpleDateFormat，通常都不是线程安全的，保证每个线程都拥有一个隔离的对象，不受其他线程干扰。

### 使用场景二

比如在拦截器中的拦截的用户信息，需要在后续服务中用到，即可将用户信息的对象放到ThreadLocal中，既保证了不同线程间的隔离，也可以避免层层服务传参数的麻烦。

### 不同场景的初始化与时机

场景一的初始化

```java
initialValue()
public static ThreadLocal<SimpleDateFormat> dateFormatThreadLocal = new ThreadLocal<SimpleDateFormat>(){
        @Override
        protected SimpleDateFormat initialValue() {
            return super.initialValue();
        }
    };
或者：ThreadLocal.withInitial(()->{...});
```

场景二的初始化

```java
// 使用set()
UserInfo userInfo = new UserInfo();
 MyThreadLocal2.userInfoContext.set(userInfo);
```

获取对象

```java
// get()方法来获取对象
 MyThreadLocal2.userInfoContext.get();
```

[基本使用代码演示](https://github.com/yunCrush/my\_collections/tree/main/src/main/java/yun/threadlocal)

**初始化时机**

使用withInitial()方法进行初始化时，表示在创建ThreadLocal对象的时候，就可以返回一个工具类对象。 使用set()方法进行初始时，初始化的时间时由我们自己决定的

### Thread ThreadLocal ThreadLocalMap 关系

**每一个Thread都有一个ThreadLocalMap, ThreadLocalMap中存放了多个Entry<>,以ThreadLocal为Key,对应的对象为Value.其中Key 是extends WeakReference**

![img](https://github.com/yunCrush/picture/blob/main/java/thread-threadlocal.png?raw=true)

```java
// Thread.java:
ThreadLocal.ThreadLocalMap threadLocals = null;
// ThreadLocal.java:
static class ThreadLocalMap {
    // 弱引用防止内存泄露
     static class Entry extends WeakReference<ThreadLocal<?>> {
            /** The value associated with this ThreadLocal. */
            Object value; 
            Entry(ThreadLocal<?> k, Object v) {
                super(k);
                value = v;
            }
        }
}
```

为什么ThreadLocal能保证每个线程持有一份隔离的对象信息？

```java
// ThreadLocal.java:
// 可以看到每个线程一个threadlocalmap,且是将当前对象this作为Key，
// 因为threadlocal是Thread中的，所以不同的线程，可以做到拥有隔离的对象
void createMap(Thread t, T firstValue) {
        t.threadLocals = new ThreadLocalMap(this, firstValue);
    }
// ThreadLocalMap构造函数，延迟加载，只有在调用的时候才会放入一个entry
 /**
 * Construct a new map initially containing (firstKey, firstValue).
 * ThreadLocalMaps are constructed lazily, so we only create
 * one when we have at least one entry to put in it.
 */
 ThreadLocalMap(ThreadLocal<?> firstKey, Object firstValue) {
            table = new Entry[INITIAL_CAPACITY];
            int i = firstKey.threadLocalHashCode & (INITIAL_CAPACITY - 1);
            table[i] = new Entry(firstKey, firstValue);
            size = 1;
            setThreshold(INITIAL_CAPACITY);
        }
```

### 主要方法

```java
// 初始化Entry的Value
initialValue()
withInitial()
set()
// 获取Entry中的Value    
get()
// 移除Entry中的Value
remove()
```

### T initialValue()解析

1. 这是一个延迟加载的方法，只有调用get()时才会调用initialValue(); 第一次调用get()方法时，如果之前是有通过**set()** 进行初始化，则不会调用initialValue().

```java
ThreadLocal.java:

public T get() {
        Thread t = Thread.currentThread();
        // 判断当前线程是否有threadLocalMap
        ThreadLocalMap map = getMap(t);
        if (map != null) {
            ThreadLocalMap.Entry e = map.getEntry(this);
            if (e != null) {
                @SuppressWarnings("unchecked")
                T result = (T)e.value;
                return result;
            }
        }
    // 没有进行初始化时，第一次调用get()时  map == null, 此时赋初值
        return setInitialValue();
    }
	// 初始化
   private T setInitialValue() {
       // 第一次初值value == null 
        T value = initialValue();
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null)
            map.set(this, value);
        else
            createMap(t, value);
       // 第一次进入map == null, 直接返回null
        return value;
    }
```

1. 通常initialValue()方法只可调用一次，但是如果在remove()之后，调用get()则可再次调用.一般都使用匿名内部类的写法重写此方法，不然默认返回的是null

### ThreadLocal问题

1. 装箱拆箱导致NPE，ThreadLocal返回的是包装类对象，如果封装的方法返回的是基本类型，会存在NPE问题
2. 共享对象\
   如果ThreadLocal中的对象是static修饰的共享对象，那么即便是使用了ThreadLocal依然会存在线程安全问题，get()得到的依旧是共享的对象，而不是独立的

### ThreadLocal注意点

1. 不使用ThreadLocal就可解决的问题则不需要用，如使用局部变量创建对象在任务数量很小的情况下。
2. 优先使用框架支持的而不是自己创造，如在spring中如果可以使用**RequestContextHolder**，**DateTimeContextHolder**,就不需要维护自己的ThreadLocal，避免忘记使用remove()，而造成内存泄露。

### 保证内存不泄露

内存泄露：即对象没有被回收，一直占用内存，这块内存就不再可用。 首先要明白内存泄露发生的点：底层是Entry结构，Key与value都是对象，都可能造成内存泄露

* 内存泄露场景1 key：threadLocal作为局部变量，我们知道调用方法就会生成一个栈帧，栈帧中存在局部变量表，与方法返回信息，操作数栈等，方法使用完毕后栈帧被销毁，\
  堆上的threadLocal对象就没有了栈帧中对其的引用，只有threadlocalmap中key对threadlocal对象的引用，如果是Entry强引用，即时发生GC，\
  堆上的threadLocal对象也不会回收，就会导致堆上的这部分内存泄露。**所以采用弱引用来保证堆上的threadLocal对象可以被回收**
* 内存泄露场景2 value：避免了场景1的内存泄露，采用弱引用，堆上的threadLocal对象被回收, 此时Entry中的key为null,\
  即:**\<null, Object>** 此时如果Object是一个大对象，依旧会发生内存泄露，因为在线程池中，线程的生命周期没有结束，所以采用**remove()**来回收value，保证内存不被泄露
