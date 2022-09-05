---
description: 介绍关于HashMap的常见问题。
---

# HashMap

## 1.提问

首先抛出一些关于HashMap的常见问题，然后通过学习源码来进行解决。

HashMap的数据结构

HashMap是线程安全的吗？

HashMap插入数据原理？

HashMap1.7与1.8不同之处？

如何解决Hash冲突的？

HashMap是如何扩容的？

HashMap如何解决多线程环境下的问题？

　　HashMap基于哈希表的 Map 接口的实现，并允许使用 null 值和 null 键。JDK7中HashMap数组（桶）+链表，JDK8中由数组+链表+红黑树，当桶中节点的长度大于8转化为红黑树，当节点的长度低于6的时候转化为链表，初始容量为16，加载因子为0.75，即只要桶中数据达到16\*0.75即触发扩从，HashMap扩容时，按照两倍桶容量进行扩容，如果传入的初始容量值是K，则HashMap的初始容量是大于K的2的整数倍。

```
 // cap传入的初始容量,返回一个大于cap且是2的幂的数
 static final int tableSizeFor(int cap) {
        int n = cap - 1;
        //位或操作，只要有一个为1，即为1
        //每次逻辑逻辑右移后与n进行位或运算是将第一个1后面的位变为1。
        n |= n >>> 1;
        n |= n >>> 2;
        n |= n >>> 4;
        n |= n >>> 8;
        n |= n >>> 16;
        return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
    }
```

　　若:加载因子越大，填满的元素越多，好处是空间利用率高了，但：冲突的机会加大了，链表长度会越来越长，查找效率降低。反之，加载因子越小，填满的元素越少，好处是：冲突的机会减小了，但:空间浪费多了。表中的数据将过于稀疏（很多空间还没用，就开始扩容了），冲突的机会越大,则查找的成本越高，因此,**必须在 "冲突的机会"与"空间利用率"之间寻找一种平衡与折衷。**

![插入元素流程图](<../../.gitbook/assets/image (27).png>)

　　**Hash函数的设计:** 先拿到key的hashcode值，是一个32位的整数，然后让高16位与低16位进行异或操作，降低hash碰撞，扰动函数：高16位与低16位混合原始哈希码的高低位，低位携带高位特征，加大随机性。JDK7做了4次扰动，JDK8只做了一次

右移16位：高位16位覆盖低位16位，高位补0。异或运算：相同为0，不同为1。

```
// hash函数设计 位运算效率高 
static final int hash(Object key) {
        int h;
// hashCode()是本地native方法，返回32位int值，h>>>16高16位覆盖低16位         
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    }
```

　　**不同之处:**  JDK7中的头插法（插入新节点时，原始节点作为新节点的后继节点，新节点放到数组中），JDK8中改用尾插法（新节点放到最后）；扩容的时候1.7需要对原数组中的元素进行重新hash，定位在新数组中的位置，1.8采用的是位置不变或索引+旧容量大小的方法；插入的时候1.7先判断是否需要扩容再插入，1.8先插入，插入完成再考虑是否需要扩容。

数组的长度都是2的幂，（2^n） -1 表示第一个数字为0，后面跟n个1， （length-1）\&hash，希望每一位都参与运算，降低hash碰撞。

```
// resize扩容函数 oldCap数组长度 减一
// (n-1)&hash 是桶的下标
if ((e.hash & oldCap) == 0) {
.....
}
```

　　数组中的原始数据在扩容时，在1.8中不需要重新hash，如果原始数据高位不是1，扩容后的数据与扩容前一致，如果原始数据高位是1，扩容后的数据比扩容前大16。10101十进制21即为原位置5+原数组长度16，扩容两倍的原因就在于每次都是和数组长度减一进行&操作，所以影响原数据的位置的取决高位是否是1，是1则原位置+原数组长度。

![扩容元素位置分析图](<../../.gitbook/assets/image (28).png>)

**线程安全及解决：**会出现数据覆盖问题，假设存在AB两个线程，如果A线程在判断Index位置是空后，挂起，B线程在index位置插入数据，A线程恢复，同样在index位置插入数据，就会覆盖B线程的节点数据，还有常见的“i++”问题。Java中HashTable,ConcurrentHashMap可以实现线程安全的Map，HashTable 是直接在方法上加synchronized关键字，锁住整个数组，粒度较大，ConcurrentHashMap使用分段锁，降低了锁的粒度，提高并发。

　　JDK1.7中ConcurrentHashMap实现：ReentrantLock+Segment+HashEntry实现，减小锁的粒度，提高并发，ConcurrentHashMap将整个HashMap分为若干个Segment段，每个段都是一个子hashmap，在加入元素时，只需要对每个segment加锁即可，在多线程的环境中，加入的元素不在同一个segment中，就可以实现真正的多线程运行，在加入元素时，需要两次定位，先定位segment,再定位子hashmap，所以理论上支持16个线程操作。

　　JDK1.8取消了segment分段锁的数据结构，取而代之的是数组+链表+红黑树，采用对每个数组元素加锁，这里采用了CAS的一种无锁算法，CAS有三个操作数，内存值V，旧的内存值A，要修改的新值为B，只有当V与A相同时才会将V修改为B，否则什么都不做，是一种乐观锁，CAS与典型的“i++”模型相关。当多个线程尝试用CAS更新变量的值时，只有一个线程更新成功，其他线程失败，失败并不会挂起，只是这次失败，可以再次尝试，CAS没有锁的开销与线程调度的开销，CAS并不完美，涉及到"ABA问题"，解决“ABA"问题涉及到下面方法。

```
# AtomicStampedReference类 compareAndSet方法
import java.util.concurrent.atomic.AtomicStampedReference;
  public boolean compareAndSet(V   expectedReference,
                                 V   newReference,
                                 int expectedStamp,
                                 int newStamp)
```



