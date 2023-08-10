---
layout:
  title:
    visible: false
  description:
    visible: false
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
---

# JVM核心与实战

### 常用理论

1. `.java` 编译后得到`.class`再通过类加载器加载到内存中，JVM就会从这个“main()”方法开始执行里面的代码。
2. 一个类的生命周期`加载 -> 验证 -> 准备 -> 解析 -> 初始化 -> 使用 -> 卸载`。a.加载到JVM。b.验证是否符合字节码规范，及JDK版本。c.为类变量以及静态变量分配内存空间，以及默认的值。d.符号引用替换为直接引用，e.初始化，真正的为变量赋值，初始化一个类的时机，如`new ReplicaManager()`或者`main`方法的主类，需要遵守双亲委派模型。f.使用。g.卸载
3. 类加载器：`Bootstrap ClassLoader` ，`Extension ClassLoader`，`Application ClassLoader` ，自定义类加载器
4. 双亲委派机制:向上委派需要加载的类的信息，直到启动类加载器，若父类没有相关的类，自己再进行加载。
5.  内存区域划分：1.8之前的方法区改为元空间（meta space），主要存放当前加载的类的信息，元空间是共享的，static变量存放其中，元空间使用的是本地内存，由于加载的类的信息，以及字符串常量池也在元空间，大小不易确定，容易引起性能问题，使用的是本地内存。

    字节码加载到内存中后，需要用执行引擎解析字节码。程序计数器： 记录当前执行的字节码指令的位置的，每个线程一个程序计数器。执行Java方法时，需要用到Java虚拟机栈，调用本地方法时用到本地方法栈。创建对象时用到堆内存。元空间与堆内存是线程共享的，Java虚拟机栈，本地方法栈，程序计数器是私有的。Java虚拟机栈的存放是一个个栈帧，栈帧包括:局部变量表，方法返回信息，操作数栈。方法使用完毕后栈帧被销毁。在局部变量表中，使用一个引用类型来存放在堆内存中对象的地址。 最后需要注意堆外内存，可以用作提升性能，是不属于JVM的，通过NIO中的`allocateDirect`这种API，可以在Java堆外分配内存空间。然后，通过Java虚拟 机里的`DirectByteBuffer`来引用和操作堆外内存空间。

    **思考题**：Tomcat这种Web容器中的类加载器应该如何设计实现？ Tomcat是打破双亲委派模型的

    // todo 为何要打破双亲委派模型？

    Tomcat自定义了Common、Catalina、Shared等类加载器，其实就是用来加载Tomcat自己的一些核心基础类库的。 然后Tomcat为每个部署在里面的Web应用都有一个对应的WebApp类加载器，负责加载我们部署的这个Web应用的类 至于Jsp类加载器，则是给每个JSP都准备了一个Jsp类加载器。每个WebApp负责加载自己对应的那个Web应用的class文件，也就是我们写好的某个系统打包好的war包中的所有class文 件，不会传导给上层类加载器去加载。



    **思考题**：创建的那些对象，到底在Java堆内存里会占用多少内存空间呢？

    对象本身的一些信息，对象的实例变量存储数据占用的空间。如对象头，如果在64位的linux操作系统上，会占用16字节，如果实例对象的内部存在Int类型实例变量占用4个字节等等。JVM在这块优化有对齐机制，指针压缩等

    **思考题**：自定义类加载器如何实现？

    写一个类，继承ClassLoader类，重写类加载的方法，然后在代码里面可以用自己的类加载器去针对某个路径下的类 加载到内存里来

    <figure><img src="https://cdn.jsdelivr.net/gh/yunCrush/yc-image/image/tomcat-%E7%B1%BB%E5%8A%A0%E8%BD%BD%E5%99%A8.png" alt=""><figcaption></figcaption></figure>
6. 栈帧销毁，堆内存中的对象还存在需要垃圾回收机制，回收对象垃圾。JVM会启动一个后台进程，检查各个内存区域的对象，如果某个实例对象没有任何一个方法的局部变量指向他，也没有任何一个类的静态变量，包括常量等地方在指向他，那么他就会被垃圾回收线程回收。
7. 分代模型：新生代，老年代，永久代。新生代中分：Eden，SurviorFrom,SurviorTo，比值8:1:1。初次分配的对象实例基本都在新生代，若新生代内存空间不足则触发`minor GC` 或者叫`young GC`。经过15次垃圾回收，对象都没被回收掉，则进入老年代，与对象头的4位对应。对象分配机制：新生代垃圾回收之后，因为存活对象太多，导致大量对象直接进入老年代；特别大的超大对象直接不经过新生代就进入老年代；动态对象年龄判断机制（年龄1+年龄2+年龄n的多个年龄对象总和超过了Survivor区 域的50%，此时就会把年龄n以上的对象都放入老年代 ）；空间担保机制（将要实例的对象占用的内存空间大于一半的新生代）。

### 参数设置

```
-Xms：Java堆内存的大小
-Xmx：Java堆内存的最大大小
-Xmn：Java堆内存中的新生代大小，扣除新生代剩下的就是老年代的内存大小了
-XX:PermSize：永久代大小
-XX:MaxPermSize：永久代最大大小
-Xss：每个线程的栈内存大小
-XX:SurvivorRatio 默认8 Eden:survivor
jdk8以后被参数替换：
-XX:MetaspaceSize和-XX:MaxMetaspaceSize
以 –XX：开头为非稳定参数, 专门用于控制 JVM 的行为，跟具体的 JVM 实现有关，随时可能会在 下个版本取消。 -XX：+-Flags 形式, +- 是对布尔值进行开关。-XX：key=value 形式, 指定某个选项的值
```

#### 案例：合理设置内存大小

每日百万交易的支付系统，如何设置`JVM`堆内存大小?栈内存与永久代大小如何设置？

<figure><img src="https://cdn.jsdelivr.net/gh/yunCrush/yc-image/image/jvm-%E6%94%AF%E4%BB%98%E4%B8%9A%E5%8A%A1%E6%B5%81%E7%A8%8B.png" alt=""><figcaption></figcaption></figure>

```

public class Order {
    private Integer userId; // 4Byte
    private Long orderTime; // 8Byte
    private Integer orderId;
    ...
}
```

压力在许多用户提交支付请求，生成许多的订单信息。核心的支付订单类实例变量较多，假设20个左右，每个实例对象大概在500Byte。每日百万的订单量，通过计算在每秒100个订单，这里假设使用3台机器部署，那么意味着每台机器每秒处理30个订单请求，即每秒占用内存是`500Byte *30 = 15KB` ，实例化对象后准备写入数据库，每秒钟创建的实例对象还可能被其他栈内的局部变量引用，假设算上后每秒占用50KB内存空间。请求扩大20倍后，每秒就是1M新生代的内存空间，如果分配1G内存，也就是1000s后，新生代就满了，需要进行`minor GC`。GC会STW，导致系统卡顿，若此时是电商大促，每秒上千订单，系统的资源就会被耗尽，不仅是内存，同样线程资源，CPU资源都会被打满。每秒1000个请求，就意味着每秒生成10MB的对象在新生代，正常为JVM分配1GB堆内存，扣除老年代后新生代就几百M，新生代在10MB/s的速度下，要不了多久就会触发GC。如果每秒请求数量再增加，可能1s就把新生代耗尽。

频繁的`minor GC` 导致部分请求响应慢，在经历一定次数的GC后并不会被回收，因为请求未响应完被栈内的局部变量引用着，因此移动到老年代，依此类推，老年代越来越多，老年代的垃圾回收STW时间更长，响应会越来越慢。

栈内存大小设置，一般也不会特别的去预估和设置的，一般默认就是比如512KB到1MB，就差不多够。

一般永久代刚开始上线一个系统，没太多可以参考的规范，但是一般你设置个几百MB，大体上都是够用的。

### 垃圾回收算法

1. 软引用在触发`minor GC` 后，依旧存在要内存溢出的情况则对软引用对象进行回收。弱引用则是发生GC就进行回收。GC过程：清理Eden区中不再被引用的对象，将存活对象复制到surviroFrom中，surviorTo幸存的对象也可能复制到surviorFrom中。
2. 通过可达性分析判断一个对象是否可回收，就是判断是否有对象引用堆内存中的对象实例,GC Roots通过引用关系向下搜寻，搜寻的路径叫引用链，如果一个对象不可达，没有引用链，则该对象可被回收。可作为GC Roots的有： 堆外内存中的有引用堆内对象实例的，都会被加入GC Roots集合，栈内局部变量，方法区（元空间）中的常量引用的对象，类静态属性引用的对象，本地方法栈的引用对象，活跃线程的引用对象。类的实例变量并不是GC Roots, 引用计数法，商用JVM很少使用。
3. 大对象直接进入老年代，避免躲过每次GC，还需要在surviorFrom与surviorTo之间来回复制。Minor GC后的对象太多无法放入Survivor区，也直接移入老年代。
4. 老年代空间担保原则：在执行任何一次Minor GC之前，JVM会先检查一下老年代可用的可用内存空间，是否大于新生代所有对象的总大小，避免survior空间不够用，全部移入老年代，若老年代再不够用，判断`-XX:-HandlePromotionFailure` 是否开启，开启则直接触发Full GC,即对老年代进行垃圾回收，如果Full GC 后还是不够内存空间来应对，“可能”的这种情况，最后就会导致OOM了。
5.  垃圾回收器：Serial单线程，停止我们系统工作进行垃圾回收，现在几乎不用。ParNew与CMS都是多线程并发垃圾回收的机制，性能更好，现在一般是线上生产系统的标配组合



    <figure><img src="https://cdn.jsdelivr.net/gh/yunCrush/yc-image/image/jvm-%E5%9E%83%E5%9C%BE%E5%9B%9E%E6%94%B6%E5%99%A8.png" alt=""><figcaption></figcaption></figure>
6. 新生代用复制算法（占用两倍内存空间），老年代用标记整理（包含标记清除与复制算法的优点）算法，标记清楚算法有内存碎片。

#### 案例：多久塞满新生代

这样一个系统：系统就是会不停的从MySQL数据库以及其他数据源里提取大量的数据，加载到自己的JVM内存里来进行计算处理 ，

每台机器上部署的实例，每分钟会执行100次数据计算任务，每次是1万条数据需要计算10秒的时间，那么我们来看看每 次1万条数据大概会占用多大的内存空间，假设每条数据1KB，为`JVM`分配4G内存，新生代老年代各1.5G。

<figure><img src="https://cdn.jsdelivr.net/gh/yunCrush/yc-image/image/jvm-%E6%A1%88%E4%BE%8B2.png" alt=""><figcaption></figcaption></figure>

每次计算1W条数据就是`1W * 1KB = 10MB` ,新生代按照8:1:1的比例，Eden区就是1.2G,S1与S2各100M左右，1分钟执行100计算，差不多1分钟左右时间就会将Eden区塞满。

**思考**：触发Minor GC的时候会有多少对象进入老年代 ？

在触发minor GC前会判断老年代空间是否够容纳所有的新生代对象。第一次判断：老年代1.5GB > 1.2GB，所以直接触发minor GC,每次计算10s钟，可以理解为对象在Eden区的存活时间是10s钟，10s钟过后，就不会有引用指向这部分对象就变成了可回收垃圾，每分钟执行100次，根据之前估算，差不多1分钟就会塞满Eden区触发minor gc,那么对于在一分钟的最后那几秒开始计算的任务，因为计算时间大于10s，所以在进行minor gc时，是可以撑过去的，这里就估算80个任务完成了，还剩下20个任务未完成，那就是200MB的Eden区被占用着，根据刚刚计算S1与S2只有100MB左右，不够存放，所以会将200MB的存活对象 移入老年代，然后清空Eden区。老年代1.5GB，最多7次，第8次时，在判断内存空间是否够将200MB存活对象，移入到老年代，因为第8次时，老年代只有100MB，所以此时触发Full GC。这样相当于7-8分钟就得执行一次Full GC了。导致的关键点就是：Eden区满了后，执行minor gc后，200MB的存活对象之间移入到了老年代，因为S1区与S2区不够存放只有100MB左右，不够存放200MB。增加新生代的内存比例，3GB左右的堆内存，其中2GB分配给新生代， 1GB留给老年代 这样Survivor区大概就是200MB，每次刚好能放得下Minor GC过后存活的对象了。

### 垃圾回收器原理

1. ParNew垃圾回收器：充分利用多核，进行多线程的垃圾回收，会STW。`XX:+UseParNewGC` ，默认设置的线程数是与CPU核数相同的，或者使用`-XX:ParallelGCThreads` 设置线程数，一般不建议修改。
2.  CMS垃圾回收器：采取的是垃圾回收线程和系统工作线程尽量同时执行的模式来处理的，CMS收集器是**一种以获取最短回收停顿时间为目标**的收集器。CMS收集器是基于**标记-清除算法**实现的，是一种老年代收集器，通常与**ParNew**一起使用

    2.1. 初始标记，系统停止工作，然后进入STW 状态，标记GC Roots直接引用的变量，而不包括实例变量。

    2.2. 并发标记 ，对老年代所有对象进行GC Roots追踪，其实是最耗时的 ，所以允许创建对象，标记实例变量。

    2.3. 重新标记 ，对于并发标记过程中产生的新对象，以及少量变动的对象进行标记，进入STW状态

    2.4. 并发清理，并发清除垃圾对象

    缺点：对CPU敏感，默认垃圾回收线程数：`(core_num + 3)/4`，与用户线程并行执行阶段，会导致用户请求没有足够的CPU。

    浮动垃圾：

    CMS有一个参数是“-XX:+UseCMSCompactAtFullCollection”，默认就打开了，在Full GC之后要再次进行“Stop the World”，停止工作线程，然后进行碎片整理。“-XX:CMSFullGCsBeforeCompaction”，这个意思是执行多少次Full GC之后再执行一次内存碎片整理的工作，默认是0，意思就是每次Full GC之后都会进行一次内存整理

    ```

    -XX:CMSInitiatingOccupancyFaction 默认92%，老年代内存占用达到一定比例,触发CMS  GC
    ```
3.  G1收集器：对各个堆内存，按照块区域进行划分

    3.1 初始标记

    3.2 并发标记

    3.3 最终标记

    3.4 筛选回收

参数举例

`JAVA_OPTS="-Xms4096m –Xmx4096m -XX:NewRatio=2 -XX:SurvivorRatio=8 -Xloggc:/home/work/log/serviceName/gc.log -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -XX:+PrintGCApplicationStoppedTime -XX:+UseConcMarkSweepGC -XX:+UseParNewGC -XX:CMSInitiatingOccupancyFraction=75 -XX:+UseCMSCompactAtFullCollection -XX:CMSFullGCsBeforeCompaction=10 "`

#### 案例：年轻代垃圾回收参数优化

背景：电商大促，每秒上千下单请求，3台机器部署，每台机器每秒处理300个请求，每个请求按照1KB计算，300个请求就是300KB。算上订单对象连带的订单条目对象、库存、促销、优惠券等等一系列的其他业务对象，一般需要对单个对象开销 放大10倍\~20倍，除了下单很可能还携带着相关的订单查询之类的，再扩大10倍的量。每秒钟会有大概300kb \* 20 \* 10 = 60mb的内存开销，一秒过后，可以认为这60mb的对象就是垃圾了。

机器：4核8G，JVM的内存分配4G，分配给堆3G，新生代老年代各1.5G，其他的占1G。

新生代1.5GB，60MB/s，大概25s就会占满新生代

<figure><img src="https://cdn.jsdelivr.net/gh/yunCrush/yc-image/image/jvm-%E6%A1%88%E4%BE%8B3.png" alt=""><figcaption></figcaption></figure>

Minor GC直接运行，一下子可以回收掉99%的新生代对象，因为除了最近一秒的订单请求还在处理，大部分订单 早就处理完了，所以此时可能存活对象就100MB左右。根据动态年龄控制，超过了150MB的一半，这100MB是一批请求的对象占用的空间，所以会导致部分对象进入老年代。

在进行minor GC前，虚拟机会检查老年代的连续内存空间是否大于新生代所有对象的总和，如果大于，表示此次minor GC 是安全的，如果小于，表示此次有风险。

有风险时，会判断`-XX:HandlePromotionFailure` 是否开启风险担保：

true: 继续检查最大可用连续空间是否大于历次晋升到老年代的对象的平均大小（这里是取得一个估值，根据历次经验提前判断如果进入老年代，是否有足够内存），如果大于，则尝试进行Minor GC,但是此次依旧是有风险的，如果小于，则进行Full GC。

false: 未开启风险担保，直接进行Full GC。避免老年代也放不下出现OOM。

“-XX:HandlePromotionFailure”参数在JDK 1.6以后就被废弃了，所以现在一般都不会在生产环境里设置这个参数了。在JDK 1.6以后，只要判断“老年代可用空间” < “历次Minor GC升入老年代对象的平均大小”，就可以直接进行Minor GC，不需要提前触发Full GC了。

按照这个模型survior区150MB是不够的，可以将S1调整到200MB

```

-Xms3072M -Xmx3072M -Xmn2048M -Xss1M -XX:PermSize=256M -XX:MaxPermSize=256M -XX:SurvivorRatio=8
```

```
// 完整参数
Xms3072M -Xmx3072M -Xmn2048M -Xss1M -XX:PermSize=256M -XX:MaxPermSize=256M -XX:SurvivorRatio=8 
-XX:MaxTenuringThreshold=5 -XX:PretenureSizeThreshold=1M -XX:+UseParNewGC -XX:+UseConcMarkSweepGC
```

#### 案例：老年代垃圾回收参数优化

根据上面的场景，假设大概就是这个订单系统在大促期间，每隔5分钟会在Minor GC之后有一小批对象进入老年代， 大概200MB(上个案例估计的是100MB)左右的大小，如下图所示：

<figure><img src="https://cdn.jsdelivr.net/gh/yunCrush/yc-image/image/jvm-%E8%80%81%E5%B9%B4%E4%BB%A3%E5%8F%82%E6%95%B0%E4%BC%98%E5%8C%96.png" alt=""><figcaption></figcaption></figure>

所以在半个小时后，老年代可能就会空间不够，进行Full GC。

**思考**：老年代GC的时候会发生“Concurrent Mode Failure”吗？

假设系统运行一段时间后，进入老年代的对象达到了900MB，此时还剩余100MB，则会触发Full GC.在并发清理期间，此时业务系统是正常运行的，如果触发了某个条件，导致有200MB的对象要进入老年代，此时会怎么办？这里就会触发“**Concurrent Mode Failure**”问题，这时会导致立马进入Stop the World，然后切换CMS为Serial Old，直接禁止程序运行，然后单线程进行老年代垃圾回 收，回收掉900MB对象过后，再让系统继续运行，不过这种概率是很小的，不需要对参数进行优化。

```

-Xms3072M -Xmx3072M -Xmn2048M -Xss1M -XX:PermSize=256M -XX:MaxPermSize=256M -XX:SurvivorRatio=8
-XX:MaxTenuringThreshold=5 -XX:PretenureSizeThreshold=1M -XX:+UseParNewGC -XX:+UseConcMarkSweepGC
-XX:CMSInitiatingOccupancyFaction=92
```

对于Full GC后的碎片整理保持默认配置即可。

```

-Xms3072M -Xmx3072M -Xmn2048M -Xss1M -XX:PermSize=256M -XX:MaxPermSize=256M
-XX:SurvivorRatio=8 -XX:MaxTenuringThreshold=5 -XX:PretenureSizeThreshold=1M -XX:+UseParNewGC
-XX:+UseConcMarkSweepGC -XX:CMSInitiatingOccupancyFaction=92 -XX:+UseCMSCompactAtFullCollection
-XX:CMSFullGCsBeforeCompaction=0
```

**思考**：todo `Minor GC`与`Full GC`发生的几种情况

Minor GC :

新生代空间不够用。

Full GC :

1. JDK1.6之后没有了风险担保，在进行Minor GC前，判断"老年代的连续内存空间" < "新生代总对象大小"，则进行`Full GC`
2. 某次Minor GC后，要升入老年代的对象的大小 比 当前老年代的空间大，此时触发`Full GC`
3.  设定了`-XX:CMSInitiatingOccupancyFaction` 参数为92%，则老年代的使用空间超过了92%也会触发`Full GC`

    基本上是老年代在占满或者不够的情况下才会触发Full GC
