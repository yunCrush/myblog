---
description: 介绍关于多线程中的常见问题
---

# 线程常见问题

## 线程基础

### 如何正确的停止一个线程？

　　1.正确停止线程的方法： thread.interrupt()，通知线程中断；&#x20;

　　2.线程内逻辑需配合响应中断：1）正常执行循环中使用 Thread.currentThread().isInterrupted()判断中断标识; 2)若含有sleep()等Waiting操作,会唤醒线程，抛出interruptedException，抛出后中断标识会重置。对于中断异常，要么正确处理，重新设置中断标识；要么在方法上声明抛出异常以便调用方处理。

　　3.为什么用 volatile 标记位的停止方法是错误的例如 生产-消费模式，含有阻塞put操作时，volatile 标记变量改变也无法唤醒阻塞中的生产者线程。

　　4.stop()、suspend() 和 resume()是已过期方法，有很大安全风险，它们强行停止线程，有可能造成线程持有的锁或资源没有释放

### wait/notify 和 sleep 方法的异同？

相同点：

　　　它们都可以让线程阻塞。 它们都可以响应 interrupt 中断：在等待的过程中如果收到中断信号，都可以进行响应，并抛出 InterruptedException 异常。

不同点：

　　1. wait 方法必须在 synchronized 保护的代码中使用，而 sleep 方法并没有这个要求。&#x20;

　　2.在同步代码中执行 sleep 方法时，并不会释放 monitor 锁，但执行 wait 方法时会主动释放 monitor 锁。&#x20;

　　3.sleep 方法中会要求必须定义一个时间，时间到期后会主动恢复，而对于没有参数的 wait 方法而言，意味着永久等待，直到被中断或被唤醒才能恢复，它并不会主动恢复。&#x20;

　　4.wait/notify 是 Object 类的方法，而 sleep 是 Thread 类的方法。

### Synchronized与Lock区别？

相同点：

　　都是用来保证线程安全的，都可以保证可见性，synchronized 和 ReentrantLock 都拥有可重入的特点。ReentrantLock是Lock接口最主要的一个实现类。

不同点：

　　1.synchronized可以用在同步代码块与方法，无需指定对象，而Lock接口必须显示加锁和解锁，lock()与unlock()解锁。

　　2.Lock的加锁与解锁不用有序，而synchronized的解锁顺序必须按照加锁的逆序进行解锁。

　　3.synchronized锁被获取了，其它线程B想获得synchronized锁就只能被阻塞，如果持有锁的线程需要很长时间才会释放，则整体效率会比较慢，并且如果持有锁的线程一直不释放锁，线程B会一直等待下去，而Lock类在等锁的过程中，如果使用的lockinterruptibly（）方法，觉得等待的时间太长，不想继续等待，可以中断退出，也可以使用tryLock方法尝试获取锁，如果获取不到锁，也可以做其它事情。

　　4.synchronized锁只能同时被一个线程持有，而Lock锁则没有这个限制，如读写锁中的读锁。sychronized锁不可以设置公平与非公平，而ReentrantLock等Lock实现类则可以根据自己的需要来设置公平与非公平。

　　5.synchronized 是内置锁，由 JVM 实现获取锁和释放锁的原理，还分为偏向锁、轻量级锁、重量级锁。Lock 根据实现不同，有不同的原理，例如 ReentrantLock 内部是通过 AQS 来获取和释放锁的。

### 实现生产者消费者模式有几种方式？

　　据我目前掌握的知识而言，3中，BlockingQueue阻塞队列，Condition，以及Wait,notify实现，本质都是阻塞队列的思想。相关代码参考 :point\_right: [https://github.com/yunCrush/thread/tree/main/src/basic\_thread](https://github.com/yunCrush/thread/tree/main/src/basic\_thread)

## 线程安全问题

### 有几类线程安全问题？

　　1.错误的时间或地点发布或初始化造成的线程安全问题，如在构造函数中新建线程进行成员对象的初始化操作，因为线程的启动需要时间，可能数据初始化并没有完成，造成"NPE"。

　　2.活跃性问题：死锁（资源互斥，请求与保持，不可剥夺，环路等待）；活锁（正在运行的程序并没有阻塞，一直运行但是得不到结果，报错重试机制，消息队列优先将消息放至头部重试，一直重试报错循环）；饥饿（长时间得不到某个锁，或者因为线程的优先级很低，长时间得不到CPU的资源）。

### 哪些场景需要注意线程安全？

　　主要从"原子性"，"可见性"，"有序性"三个方面思考，常见的“共享变量与资源的访问”，“i++”问题；时序操作，如下: 两个线程分别判断包含了key，两个都会执行remove(),因为时序问题，只有一个执行成功，导致线程安全问题；ArrayList不适合用于多线程的读写。

```
if(map.containsKey(key){
    map.remove(obj)
}
```

### 多线程为什么会带来性能问题？

　　线程的调度角度：线程的调度会引起上下文切换，上下文的切换会保存当前线程的状态，寻找下一处即将执行的代码，如果我们执行的内容花费的时间本就很短，只进行简单的计算，上下文切换的性能开销远比线程本身性能开销大；对于线程访问过的数据，会加入缓存，切换线程，导致缓存失效，又需要重新的写入缓存。导致上下文切换的问题：频繁的竞争锁，由于IO读写等导致的频繁阻塞等问题。

　　线程协作的角度：为了保证线程安全，多线程时，为了避免数据错乱，将线程的工作内存的数据flush到主内存，将主内存数据refresh到其它线程的工作内存，还可能禁止CPU的指令重排序。线程安全的优先级高于性能的优先级。

## 线程池

### 线程池的好处？

　　降低资源消耗，线程可以复用，并且线程是创建好的，时刻准备执行任务，提升响应速度，可以根据内存和CPU的资源灵活的控制线程的数量，方便管理线程，统计已经执行过的任务数量。

　　线程池参数：核心线程（corePoolSize），核心线程满后放入工作队列（workQueue），线程工厂创建新线程（ThreadFactory），工作队列满后创建线程至最大线程数量（maxPoolSize），无任务时空闲线程存活时间（KeepAliveTime+时间单位），达到最大线程数量并且工作队列满后的拒绝策略（Handler）。

　　线程池的4中拒绝策略：

![线程池4种拒绝策略](<../../.gitbook/assets/image (32).png>)

　　DiscardPolicy：提交的新任务直接拒绝，可能造成数据丢失；  DiscardOldestPolicy: 拒绝工作队列中存活时间最长的任务。

　　CallerRunsPolicy：拒绝后给提交任务的线程执行，执行任务需要时间，提交任务的线程被占用，减缓了任务提交的节奏，并且新提交的任务不会丢失。

　　AbortPolicy：直接抛出运行时异常，RejectedExecutionException。

### 线程池常用的阻塞队列？

　　线程池的内部结构：线程池管理器（管理线程池的创建销毁等），工作线程（执行任务），任务队列（作为一种缓冲机制，多线程安全要求较高，采用BlockingQueue），任务（任务要求实现统一的接口，以便工作线程可以处理和执行）。

![常见阻塞队列](<../../.gitbook/assets/image (33).png>)

　　FixedThreadPool线程池，固定大小，所以需要一个无线容量的阻塞队列；singleThreadExecutor同理，只有一个线程，如果线程发生异常，也会重新创建一个线程来执行任务。

　　CachedThreadPool线程池，无限扩增线程，所以阻塞队列只需要同步任务即可。

　　ScheduledThreadPool与SingleThreadScheduledExecutor线程池都是定时的，所以采用延时工作队列作为阻塞队列，延时工作队列采用的是堆结构，按照延迟的时间长短进行排序。

### 为什么为什么不应该自动创建线程池？

　　从两方面考虑，一是使用Executors.new...ThreadPool（）方式创建的线程池，我们通过查看源码发现，有的是采用了一个无界阻塞队列，如果大量的任务堆积到队列中，耗用大量内存，最终会导致OOM；令一方面，在设置最大线程数量时，默认设置的是Integer.MAX\_VALUE，当任务数量很多的时候，会创建很多的线程，最终超过了操作系统的上限，无法创建新线程，或者导致内存不足。

### 合适的线程数量，线程数与CPU核心数的关系？

　　首先，我们应当明白我们设置合适的线程数是为了最大程度上的利用CPU与内存等资源。合适的贤臣数量指的是corePoolSize。对于CPU密集型任务，我们应当根据服务器目前有哪些程序占用CPU，再进行设置，这里涉及到服务器自身的程序会和线程争抢CPU，线程切换导致性能下降，对于IO密集型任务，我们可以将线程的数量设置大一点，在等待IO（文件IO，网络IO等）阻塞时，可以切换到其它线程执行任务，提高资源的利用率。

```
# Java并发编程实战》的作者 Brain Goetz 推荐的计算方法：
  线程数 = CPU 核心数 *（1+平均等待时间/平均工作时间）
```

### 如何正确关闭线程池？shutdown 和 shutdownNow 的区别?

　　shutdown()：拒绝新提交的任务，等到线程池中的任务和队列中等待的任务被执行完毕后关闭线程池。

　　isShutdown()：返回true与false来判断是否开始了线程池的关闭，并没有关闭，只是开始了关闭的流程。

　　isTerminated()：真正检测线程池是否关闭，返回true表示线程池内任务全部执行完，包括工作队列中。

　　shutdownNow()：立即关闭线程池的意思，首先给线程发送Interrupt中断信号，尝试中断任务的执行，然后将没有执行完的任务放入List，并返回，进行相关的补救。

## 锁

　　锁的分裂参考 :point\_right: [各种各样的“锁”](ge-zhong-ge-yang-de-suo.md)

### Lock常用的方法？

　　lock()：最基础的获取锁，如果得不到锁的话，会阻塞，需要显示释放锁unlock()；

　　tryLock()：获取锁成功就返回true，失败就返回false，该方法会立即返回，不会阻塞等待，没有获取到锁，可以尝试等待，或者跳过这个任务，可以用来解决死锁。

　　lockInterruptibly()：是可以响应中断的，相比于不能响应中断的 synchronized 锁，lockInterruptibly() 可以让程序更灵活，可以在获取锁的同时，保持对中断的响应，并且不会超时。