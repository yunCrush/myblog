---
description: 操作系统内存管理部分
---

# OS内存管理（三）

## 页

虚拟内存划分为页（page），实际内存划分为块（frame），页表维护了虚拟地址到真实地址的映射，操作系统在内存中划分出小块区域给页表。page---->页表----------->frame，通常页大小与块大小相等。

内存管理单元（Memory Management Unit， MMU）：虚拟内存地址转换为实际内存地址，发生在MMU

页表存储进程的信息，二级页表可以解决大页面时，需要创建很多条目的问题。

转置检测缓冲区（TLB）：存在于MMU内部，缓存的设计，通常是一张表，所以 TLB 也称作快表。TLB 中最主要的信息就是 Page Number到 Frame Number 的映射关系。每一行称为缓存行，或者缓存条目。

![](<../../.gitbook/assets/image (18).png>)

TLB失效：Page Number在TLB没有找到，分为软失效与硬失效。

软失效（soft miss）：Frame在内存中，TLB没有，刷新TLB缓存，如果TLB条目满的话，需要覆盖一个，采用LRU算法进行缓存条目置换。

硬失效（hard miss）：Frame在内存中没有，需要到磁盘读取，操作系统触发缺页中断，原有需要读取内存的线程休眠，中断响应程序开始从磁盘读取对应的Frame到内存，读取完毕后再次触发中断，通知更新TLB，并且唤醒休眠的线程去排队，Hard Miss相对耗时，不可能从休眠到执行。

**TLB缓存设计**

1. 全相联映射：缓存数据存可能在任何缓存条目中，需要遍历所有条目，性能低
2. 直接映射：类似哈希函数形式，命中率低
3. n路组相连映射：类似哈希表的开放寻址法，一个Page Number可以出现在n个位置

高性能场景对内存需求较高，采用大内存分页。

**缓存置换算法：**

1. FIFO（First In First Out）
2. NRU（Not Recently Used）
3. LRU（Least Recently Used）

LFU （Least Frequently Used）的劣势在于它不会忘记数据，累计值不会减少，为了描述最近，采用“老化算法”，8位来描述累计位A，每次当读位的值R到来时，A的值右移动，并将R放置在A最高位。

LRU原始的实现方式是数组，数组中的每一项都有数据的最近使用频次，每次置换需要查询整个数组。

更好的做好：双向链表，将使用到的数据移动到链表首部，每次将链表尾部的数据移走

## 内存回收

GC过程：标记程序（Mark）清除程序（Sweep）变更程序（Mutation）

标记（Mark）就是找到不用的内存，清除（Sweep）就是回收不用的资源，而修改（Muation）则是指用户程序对内存进行了修改

垃圾回收算法： 1.引用计数法（多线程环境下，如果一旦算错一次，将无法纠正），2.可达性分析（Root Tracing），根对象到一个对象走过的路径称为引用，如果一个对象到根对象没有引用，就被回收，解决了内存的循环引用，容错性好。3. 标记清除法

三色标记-清除算法: 白色需要GC，黑色确定不需要GC，灰色可能需要表示被回收但是没有被标记，或者增量任务，开始默认全部白色。

Mutation分3类：创建新对象 删除已有对象 调整已有引用

三色标记没有解决内存碎片的问题，内存整理算法，涉及到Eden,survivor from， surivor to。**如果内存不够用，有两种解决方案。一种是降低吞吐量——相当于 GC 执行时间上升；另一种是增加暂停时间，暂停时间较长，GC 更容易集中资源回收内存。**

