---
description: 介绍并发编程中的各种样式的锁。
---

# 各种各样的“锁”

## 1.锁分类

![锁分类](<../../.gitbook/assets/image (29).png>)

## 2.悲观锁与乐观锁

**悲观锁**：每次同步资源都会上锁，同步结束后才释放锁，适用于“写”操作场景。

例子：synchronized（也是可重入锁）关键字与Lock相关类，如ReentrantLock，常见的流程是先加锁lock()，同步资源结束后释放锁unlock()。

synchronized关键字的实现原理，利用monitor锁实现，获得monitor锁的唯一途径就是进入由这个锁保护的代码块，在线程进入synchronized保护的代码块之前自动获得锁，无论正常退出，或者是抛异常退出都会释放锁，

所以这里会有两处释放锁的指令。

```
public class SynTest {
// synchronized修饰方法
public synchronized void method() {
    method body
}

// 等价于下面方法    intrinsicLock就是monitor锁
public void method() {

    this.intrinsicLock.lock();

    try{
        method body
    }
    finally {

        this.intrinsicLock.unlock();
    }
 }
}
```

使用javac SynTest.java 编译得到SynTest.class字节码文件，再使用javap -verbose SynTest.class可以看到对应的反汇编内容，发现有个ACC\_SYNCHRONIZED标志，当某个线程要访问某个方法的时候，会首先检查方法是否有 ACC\_SYNCHRONIZED 标志，如果有则需要先获得 monitor 锁，然后才能开始执行方法，方法执行之后再释放 monitor 锁。

synchronized修饰代码块与修饰方法不同：

```
public class SynTest {
    public void synBlock() {
        synchronized (this) {
            System.out.println("yuncrush");
       }
    }
}
```

同样，我们查看反编译的代码，发现多了monitorenter与monitorexit，可以把执行 monitorenter 理解为加锁，执行 monitorexit 理解为释放锁，每个对象维护着一个记录着被锁次数的计数器。未被锁定的对象的该计数器为 0。monitorexit 的作用是将 monitor 的计数器减 1，直到减为 0 为止。代表这个 monitor 已经被释放了，已经没有任何线程拥有它了，也就代表着解锁，所以，其他正在等待这个 monitor 的线程，此时便可以再次尝试获取这个 monitor 的所有权。

**乐观锁**：每次同步资源时并不会上锁，但是在更新时会与之前获取到的资源信息进行比对，判断在这期间资源是否有被修改过，如果不一致，则重试或报错，一致就更新，适用于“读操作”场景。

例子：原子类AtomicInteger，多个线程同时操作一个原子变量，CAS锁。

CAS缺点：1.高并发场景长时间自旋消耗CPU；2.“ABA问题”；3.作用范围不能灵活控制(CAS往往针对的是某一个变量，而对于多个不同类型的变量，变量是独立的，不能进行同时的CAS操作，如果把多个原子操作单纯的组合一起，并不具备原子性，因此我们想对多个变量进行CAS操作，并保证线程安全的话是困难的)

作用范围不能灵活控制解决方案：引入一个新类，将多个变量进行整合到类中，然后再利用 atomic 包中的 AtomicReference 来把这个新对象整体进行 CAS 操作，这样就可以保证线程安全。

## 3.
