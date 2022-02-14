---
description: 介绍关于多线程的核心知识，以及解决一些具有争议性的问题。
---

# 多线程核心知识

## 1.实现多线程的方式

　　官方文档说明只有两种方法：1.通过继承Thread，重写run方法，2.实现Runnable接口，重写run方法。对于线程池等，都逃不过本质区别，也是实现Runnable接口。

![](<../../.gitbook/assets/image (24).png>)

![](<../../.gitbook/assets/image (23).png>)

　　准确而言，就是构造Thread类，并将Runnable实例传入，以实现创建一个线程，并且 都是调用start()方法来启动一个线程（线程真正的运行是执行run方法，因为调用了start()方法后，线程其实并没有真正的执行，而是处于new状态，在等待CPU时间片）。通过查看Thread源代码得知，Thread也是实现Runnable接口，实现了Runnable接口的run方法。

```
 # Thread类run方法
  private Runnable target;
  @Override
    public void run() {
        if (target != null) {
            target.run();
        }
    }
```

　　**为什么不直接调用Thread的run方法，而是需要通过start方法来启动线程呢？**因为只有通过Thread.start()线程才会经历一个完整的生命周期，并且start方法只能调用一次，查看start源码发现，调用start方法时，会先判断线程的状态，如果状态不等于0，表示线程已经启动过，会抛出不合法的线程状态异常，正常的话然后将线程加入到线程组里面，接着调用本地的start0()方法。

　　方法1与方法2的主要区别在于run方法的不同，extends Thread，最终调用的是target.run()，implements Runnable的整个run方法都被重写了。

## 2.停止线程的方式

　　正确的停止线程的方法是使用interrupt方法，通过改变interrupted标记为来响应中断。

```
# 主线程调用interrupt方法，将子线程标记为中断
thread.interrupt()
# 判断线程是否被中断
Thread.currentThread().isInterrupted()
```

　常见的线程中断的情况：

1. 线程在阻塞中响应中断，即run方法内有sleep方法时，需要通过Thread.currentThread().isInterruped()方法判断。
2. 线程在每一次迭代后，如while内每次都调用sleep方法，此时sleep会自己响应中断，无需判断标记位。
3. while内含有try/catch时，会导致中断失效，Thread.sleep()检测到的中断信号抛出的异常被捕获了。

处理中断实践：

1. **传递中断：**优先选择在方法上抛出异常。用 throws Interrupted Exception标记你的方法，不采用try语句块捕获异常，以便于该异常可以传递到顶层，让run方法可以捕获这一异常，例如

```
private void throwInMethod() throws InterruptedException {
        Thread.sleep(2000);
    }
```

由于run方法内无法抛出checked Exception(只能用 try catch)，顶层方法必须处理该异常，避免了漏掉或者被吞掉的情况，增强了代码的健壮性。

&#x20;   2**. 恢复中断**

```
// 在catch子语句中调用Thread.currentThread().interrupt()来恢复设置中断状态
public class RightWayStopThreadInProd implements Runnable {

    @Override
    public void run() {
        while (true) {
            if (Thread.currentThread().isInterrupted()) {
                System.out.println("Interrupted，程序运行结束");
                break;
            }
            reInterrupt();
        }
    }

    private void reInterrupt() {
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
            e.printStackTrace();
        }
    }

    public static void main(String[] args) throws InterruptedException {
        Thread thread = new Thread(new RightWayStopThreadInProd());
        thread.start();
        Thread.sleep(1000);
        thread.interrupt();
    }
}
```

## 3.Volatile关键字

同步一个共享变量，保证这个变量在被读取时时最新的值，相比synchronized，Lock更轻量，volatile不会发生上下文切换，不会让线程阻塞，开销小，所以效果相对较小，保证了共享变量的的可见性，但是无法提供原子性与互斥性，volatile无锁操作，所以没有释放与获取锁的时间，性能高。

还有一个作用就是在一定程度上禁止重排序，单线程指令重排序，执行结果不会改变，但是多线程环境下乱序操作会导致线程安全问题。

不适用场景：“i++”问题

适用场合：a.布尔标记位（共享变量只存在被赋值、读取的操作，没有其他操作）；b.作为触发器，保证其他变量的可见性，利用了Happens-before的传递性。

```
# 触发器
Map configOptions;

char[] configText;

volatile boolean initialized = false;

. . .

// In thread A

configOptions = new HashMap();

configText = readConfigFile(fileName);

processConfigOptions(configText, configOptions);

initialized = true;

. . . 

// In thread B

while (!initialized) 

  sleep();

// use configOptions

```

根据happens-before 关系的单线程规则，线程 A 中 configOptions 的初始化 happens-before 对 initialized 变量的写入，而线程 B 中对 initialzed 的读取 happens-before 对 configOptions 变量的使用，同时根据 happens-before 关系的 volatile 规则，线程 A 中对 initialized 的写入为 true 的操作 happens-before 线程 B 中随后对 initialized 变量的读取。
