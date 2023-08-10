# Redis核心技术

### 数据类型

![](https://cdn.jsdelivr.net/gh/yunCrush/yc-image/image/redis-%E5%BA%95%E5%B1%82%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84.png)

#### RedisObject

redis不同数据类型都有些相同的元数据要记录（比如最后一次访问的时间、被引用的次数等 ,这里结合缓存淘汰策略的LFU算法)

value是Long类型Ptr就直接指向value,不再额外申请SDS空间. Value是String类型时,小于44字节,则将SDS与指针申请连续的内存空间.对应的编码名称如图所示.

![](https://cdn.jsdelivr.net/gh/yunCrush/yc-image/image/redis-RedisObject.png)

redis采用的是Hash表进行存储的,采用了拉链法解决Hash冲突,Hash表其实就是一个数组,每个元素时一个hash桶,在每个桶节点延升一个链表来解决hash冲突,存放多个dictEntry,每个dictEntry有3个8字节的指针,共24字节,而jemalloc分配的是32字节的内存.

![](https://cdn.jsdelivr.net/gh/yunCrush/yc-image/image/redis-hashtable.png)

解释1亿张图片id与图片存储对象ID,pictureId与OSSId都是10位数,耗费6.4GB内存?

10 位数的图片 ID 和图片存储对象 ID 是 Long 类型整数，所以可以直接用 int 编码 的 RedisObject 保存,每个 int 编码的 RedisObject 元数据部分占 8 字节，指针部分被直 接赋值为 8 字节的整数了,此时，每个 ID 会使用 16 字节，加起来一共是 32 字节.另外32字节是jemalloc为每个dictEntry分配的,\*key, \*value, \*next 各占8各字节，jemalloc按照2^n来进行分配.这样\<pictureId,OSSId> 占用了64字节,而实际的有效信息只有16字节,即key与value的redisObject中的ptr.

![](https://cdn.jsdelivr.net/gh/yunCrush/yc-image/image/redis-string-64.png)

#### String

\<key,String>

C 语言中可以使用 char\* 字符数组来 实现字符串,Redis 设计了简单动态字符串（Simple Dynamic String，SDS）的结构,提升字符串的操 作效率，并且可以用来保存二进制数据.

char\*字符串的不足,需要遍历一次字符串,再进行拼接(且需要手动编程分配空间),求长度等操作,复杂度都很高,且因为是通过尾部的\0来判断是否截止,若出现在中部,则提前结束,不可,无法完整表示包含“\0”的数据，

SDS 把目标字符串的空间检查和扩容封装在了 sdsMakeRoomFor 函数中，并且 在涉及字符串空间变化的操作中，如追加、复制等，会直接调用该函数,避免了开发人员因忘记给目标字符串扩容，而导致操作失败

SDS 一共设计了flags 5 种类型，分别是 sdshdr5、sdshdr8、sdshdr16、sdshdr32 和 sdshdr64

![](https://cdn.jsdelivr.net/gh/yunCrush/yc-image/image/redis-sds-simple.png)

**紧凑型字符串结构的编程技巧**

　　由flags类型决定了len和alloc的大小.sdshdr5已经废弃,sdshdr8(alloc和len是uint8\_t类型,8位无符号整形 )、sdshdr16、sdshdr32 和 sdshdr64(unit64\_t),分别是8位,16,32,64位,则len和alloc分别占用1,2,4,8个字节.可以根据需要存储的字符串的长度选择不同的flags类型,这样len和alloc则不同,占用的字节数就更小,假设只存储10位的字符串,.若都用unit64\_t.alloc和len一起占用了16个字节了,比字符串还大,不合适.

　　除此之外,Redis 在编程上还使用了专门的编译优化来节省内存空间,如sdshdr8,正常是按照8个字节进行存储,redis的优化,

`struct __attribute__ ((__packed__)) sdshdr8`在编译时采用不要使用字节对齐的方式，而是采用紧凑的方式分配内存.实际占用多少空间就分配多少空间.

```
#include <stdio.h>
        int main() {
            struct __attribute__((packed)) s2{
                char a;
                int b;
            } ts2;
            printf("%lu\n", sizeof(ts2)); // 优化后只会占用5个字节,优化钱会分配8字节空间
            return 0;
        }

```

**实现一个性能优异的Hash表**

解决Hash冲突问题和ReHash问题: 使用链式哈希和渐进式哈希解决这两个问题.

Hash冲突就是多个键被映射到同一个位置.可以使用链式hash解决,但是链式长度不能太长,会影响性能,当链式哈希的链长达到一定长度时，我们可以使用 rehash,但是rehash开销较大,可以使用渐进式hash.对hash表的实现,在dict.h和dict.c文件中

Hash表实现:

　　 在 dict.h 文件中，Hash 表被定义为一个二维数组（dictEntry \*\*table），这个数组的每个 元素是一个指向哈希项（dictEntry）的指针,为了实现链式哈希， Redis 在每个 dictEntry 的结构设计中，除了包含指向键和值的 指针，还包含了指向下一个哈希项的指针。

实现 rehash:

　　指扩大 Hash 表空间,Redis 准备了两个哈希表，用于 rehash 时交替保存数据.

**渐进式 rehash 实现**

键拷贝时，由于 Redis 主线程无法执行其他请求，所以键拷贝会阻塞主线程，这 样就会产生 rehash 开销,为了降低开销,提出了渐进式hash方法,渐进式hash即每次拷贝一个hash槽的数据.

![](https://cdn.jsdelivr.net/gh/yunCrush/yc-image/image/redis-%E6%B8%90%E8%BF%9B%E5%BC%8Fhash.png)

#### List

\<key,list>

#### Hash

\<key, entry> key field val field2 val2

```
hash-max-ziplist-entries：表示用压缩列表保存时哈希集合中的最大元素个数。512
hash-max-ziplist-value：表示用压缩列表保存时哈希集合中单个元素的最大长度。 64
Hash 集合中写入的元素个数超过了 hash-max-ziplist-entries，
或者写入的单个元素大小超过了 hash-max-ziplist-value，Redis 就会自动把 Hash 类型的实现结构由压缩列表转为哈希表
```

#### Set

#### SortedSet

### 数据结构

#### 简单动态字符串(SDS)

如上String

#### 压缩列表(ziplist)

List、Hash 和 Sorted Set 这三种数据类型，都可以使用压缩列表 （ziplist）来保存数据。压缩列表的函数定义和实现代码分别在 ziplist.h 和 ziplist.c 中

表头有三个字段 zlbytes、zltail 和 zllen，分别表示列表长 度、列表尾的偏移量，以及列表中的 entry 个数。压缩列表尾还有一个 zlend，表示列表结束。 压缩列表能节省内存，在于用一系列连续的 entry 保存数据,节省指针连接诶所占用的空间。

![](https://cdn.jsdelivr.net/gh/yunCrush/yc-image/image/redis-ziplist.png)

prev\_len: 前一个entry 小于 254字节 取1字节,前一个entry大于255字节时,取5字节

len: 表示自身长度，4 字节

encoding：表示编码方式，1 字节；

content：保存实际数据

因此一个图片的存储对象 ID 所占用的内存大小是 14 字节（1+4+1+8=14），实际分配 16 字节

第一个entry和最后一个entry可通过表头三个字段定位到,时间复杂度O(1).

#### 跳表

增加了多级索引，通过索引位置的几个跳转，实现数据的快速定位

![](https://cdn.jsdelivr.net/gh/yunCrush/yc-image/image/redis-%E8%B7%B3%E8%A1%A8%E6%9F%A5%E6%89%BE.png)

**用集合类型保存单值的键值对**

### 网络模型

### 内存管理

### 最佳实践

#### 内存碎片

**内因：内存分配器的分配策略**

Redis 可以使用 libc、jemalloc、tcmalloc 多种内存分配器来分配内存，默认使用 jemalloc。

按照一系列固定的大小划分内存空间,申请20字节,但是分配了32字节,如果写入10字节数据就不用分配内存空间了,但是每次申请的都不一样导致饿了内存碎片.

**外因：键值对大小不一样和删改操作**

![](https://cdn.jsdelivr.net/gh/yunCrush/yc-image/image/redis-%E5%86%85%E5%AD%98%E7%A2%8E%E7%89%87.png)

如果E想要4字节的内存空间,jemalloc无法分配连续的4个字节.

**判断是否有内存碎片**

使用`info memroy` 命令

used\_memory: Redis 为了保存数据实际申请使用的空间

used\_memory\_rss 是操作系统实际分配给 Redis 的物理内存空间，里面就包含了碎片.

```
内存碎片率: mem_fragmentation_ratio = used_memory_rss/ used_memory 
```

大于1小于1.5是合理的.

![](https://cdn.jsdelivr.net/gh/yunCrush/yc-image/image/redis-info-memory.png)

**清理内存碎片**

从 4.0-RC3 版本以后,Redis提供了自动清理碎片的机制: “搬家让位，合并空间” ,对于数据的拷贝顺序有要求,且需要拷贝很多数据是一个耗时的操作,单线程的Redis只能阻塞等着.

开启自动清理碎片

```
config set activedefrag yes
示例:
127.0.0.1:6379> config set activedefrag yes
OK
127.0.0.1:6379>
```

真正开始清理内存碎片需**同时满足**配置文件中的条件,清理过程中一个不满足了就不清理了:

　　`active-defrag-ignore-bytes 100mb`：表示内存碎片的字节数达到 100MB 时，开始 清理；

　　`active-defrag-threshold-lower 10`：表示内存碎片空间占操作系统分配给 Redis 的 总空间比例达到 10% 时，开始清理

减少对redis的影响,会监控内存清理对占用CPU的时间.

　　`active-defrag-cycle-min 25`： 表示自动清理过程所用 CPU 时间的比例不低于 25%(CPU分配给Redis的时间)，保证清理能正常开展；

　　 `active-defrag-cycle-max 75`：表示自动清理过程所用 CPU 时间的比例不高于 75%(CPU分配给Redis的时间)，一旦超过，就停止清理，从而避免在清理时，大量的内存拷贝阻塞 Redis，导致 响应延迟升高。

#### 缓冲区

缓冲区即用一块内存空间来暂时存放命令数据,避免因为命令的发送速度大于处理速度,导致的命令丢失,但是缓冲区的大小有限超过阈值会造成缓冲区溢出.发生溢出就会丢数据

缓冲区的两个场景:

1. Redis作为c/s架构,缓冲区作用在客户端与服务端,缓冲客户端发送给服务端的命令.
2. 主从节点间进行数据同步时，用来暂存主 节点接收的写命令和数据

大体流程如下所示:

![](https://cdn.jsdelivr.net/gh/yunCrush/yc-image/image/redis-%E8%BE%93%E5%85%A5%E7%BC%93%E5%86%B2%E5%8C%BA.png)

**查看输入缓冲区的内存使用情况**

```shell
127.0.0.1:6379> client list
id=3 addr=127.0.0.1:60054 laddr=127.0.0.1:6379 fd=9 name= age=201 idle=0 flags=N db=0 sub=0 psub=0 multi=-1 qbuf=26 qbuf-free=40928 argv-mem=10 obl=0 oll=0 omem=0 tot-mem=61466 events=r cmd=client user=default redir=-1
127.0.0.1:6379> 

cmd，表示客户端最新执行的命令。这个例子中执行的是 CLIENT 命令。
qbuf，表示输入缓冲区已经使用的大小。这个例子中的 CLIENT 命令已使用了 26 字节
大小的缓冲区。
qbuf-free，表示输入缓冲区尚未使用的大小, qbuf-free + qbuf 即分配给当前客户端的输入缓冲区大小

```

没有办法通过参数调整输入缓冲区的大小,在代码中规定了阈值是1G.可以从客户端的命令发送着手,避免写入大Key,避免redis主线程阻塞

**应对输出缓冲区溢出**

三种情况导致的:

服务器端返回 bigkey 的大量结果； 执行了 MONITOR 命令； 缓冲区大小设置得不合理.

```
127.0.0.1:6379> monitor
OK
1689676069.665450 [0 127.0.0.1:60056] "COMMAND"
1689676074.563954 [0 127.0.0.1:60056] "get" "name"
不要在线上生产环境中持续使用 MONITOR
```

通过 `client- output-buffer-limit` 配置项，来设置缓冲区的大小。

```
普通客户端设置输出缓冲区
client-output-buffer-limit normal 0 0 0
订阅客户端设置输出缓冲区,8mb 表示输出缓冲区的大小上限
为 8MB，一旦实际占用的缓冲区大小要超过 8MB，服务器端就会直接关闭客户端的连
接；2mb 和 60 表示，如果连续 60 秒内对输出缓冲区的写入量超过 2MB 的话，服务器端
也会关闭客户端连接
client-output-buffer-limit pubsub 8mb 2mb 60
```

**复制缓冲区的溢出问题**

在全量复制过程中,既会向从节点发送RDB,也会接收新的请求放入缓冲区,主节点为每一个从节点维护一个缓冲区,来保证主从数据的同步.

```
slave 参数表明该配置项是针对复制缓冲区的。512mb 代表将缓冲区大小的上限设
置为 512MB；128mb 和 60 代表的设置是，如果连续 60 秒内的写入量超过 128MB 的
话，也会触发缓冲区溢出
config set client-output-buffer-limit slave 512mb 128mb 60
```

**复制积压缓冲区**

增量复制使用的叫复制积压缓冲区,是一个大小有限的环形缓冲区,缓冲区写满后,会覆盖以前的数据,若从节点未来得及同步被覆盖的数据,会触发全量同步. 设置参数`repl_backlog_size` 可控制缓冲区大小.

#### 消息队列

消息队列在存取消息时，必须要满足三个需求，分别是消息保序、处理重复的消息 和保证消息可靠性

**基于 List 的消息队列解决方案**

生产者可以使用 LPUSH 命令把要发送的消息依次写入 List，而消费者则可以使 用 RPOP 命令.生产者并不会通知消费者消息到达,需要消费者轮询,一直RPOP,为了避免CPU被打满,可以使用BRPOP命令,阻塞式读取,没有数据到达消息队列时,自动阻塞,直到有数据到达.

使用BRPOPLPUSH留存消息,保证消费者能消费到消息,消费者从消息队列右端取出(BRPOP),又放到(LPUSH)另一个List(mqback)中,若消费者在消费过程中宕机,可以从mqback中读取,再次消费,保证消息的可靠性.

生产者发送消息过快,导致的消息累积.推出了Streams解决方案,多个消费者构成一个消费者组.

**基于 Streams 的消息队列解决方案**

```
XADD：插入消息，保证有序，可以自动生成全局唯一 ID；
XREAD：用于读取消息，可以按 ID 读取数据；
XREADGROUP：按消费组形式读取消息；
XPENDING 和 XACK：XPENDING 命令可以用来查询每个消费组内所有消费者已读取
但尚未确认的消息，而 XACK 命令用于向消息队列确认消息处理已完成
```

```
# mqstream 消息队列名称 *: 自动生成一个全局唯一的ID  repo: 键 5: 值
XADD mqstream * repo 5
"1599203861727-0"

#没有消息到达BLOCK 100ms 读取
XREAD BLOCK 100 STREAMS mqstream 1599203861727-0

#$:读取最新的消息 一直没有消息到达,阻塞10s后返回空值
XREAD block 10000 streams mqstream $

# 创建一个消费者组group1 消费的队列时mqstream
XGROUP create mqstream group1 0
```

**对比主流消息队列的不足**

　　生产者和消费者保证消息不丢失,但是在队列中间件本身Redis无法保证: AOF异步写入,主从复制也是异步的,主从切换时,从库还未完成主库发来的数据,就被提成主库.

　　消息积压: Redis在面对大量消息时,会直接丢弃,且大量消息对齐对内存压力过大.

#### 异步机制

**Redis 实例阻塞点**

![](https://cdn.jsdelivr.net/gh/yunCrush/yc-image/image/redis-blocking.png)

网络IO,采用了多路复用,避免了无效等待,集合全量查询和聚合操作 O(N)操作.

　　1. 删除数据的本质是释放键值对占用内存空间,也会阻塞主线程,尤其是大Key和清空数据库,释放内存只是第一步，为了更加高效地管理内存空间，在应用程序释放内存时，操作系统需要把释放掉的内存块插入一个空闲内存块的链表，以便后续进行管理和再分配。这个过程本身需要一定时间，而且会阻塞当前释放内存的应用程序.(可子线程异步)

　　2. 记录AOF日志时,需要写回到磁盘,根据不同的写回策略同步操作的耗时是1-2ms,如果有大量的日志,则会阻塞主线程了.采用的是子进程的方式生成RDB快照.(可子线程异步)

　　3. 从库接收主库的RDB文件,并清空数据库,根据RDB文件恢复数据,加载到内存,清空数据库会阻塞,若RDB文件过大,加载到内存时同样会耗时.要想堆外提供服务,必须把RDB加载完,无法使用异步实现.

　　４.部署切片集群时,迁移大Key时,同样会阻塞主线程

　　大key删除,清空数据库,AOF日志不属于关键路径,可异步实现.Hash 类型的 bigkey 删除，你可以使用 HSCAN 命令，每次从 Hash 集合中 获取一部分键值对（例如 200 个），再使用 HDEL 删除这些键值对

**异步的子线程机制**

主线程将任务放入队列,直接返回结果,子线程从任务队列(任务队列由链表构成)中获取任务,开始执行.

```
# 异步清空数据库命令
FLUSHDB ASYNC
FLUSHALL AYSNC
# 键值对删除,集合中包含大量元素,建议使用UNLINK命令
```

![](https://cdn.jsdelivr.net/gh/yunCrush/yc-image/image/redis-%E5%AD%90%E7%BA%BF%E7%A8%8B%E5%BC%82%E6%AD%A5.png)

#### 旁路缓存

**同步直写** : 数据库与缓存都操作完毕再响应接口,提供了数据可靠性保证

**异步写回** : 数据的操作是在缓存中进行的,响应快,但是无法保证数据的可靠,并没有写回数据库.

#### 淘汰策略

建议把缓存容量设置为总数据量的 15% 到 30%，兼顾访问性能和内 存空间开销. 设置redis容量大小`CONFIG SET maxmemory 4gb`

在容量满后,会触发相关策略,所以可能删除过期的键,也可能删除还没有过期的键.

默认不进行数据淘汰: noeviction

设置了过期时间的数据中进行淘汰，包括 volatile-random、volatile-ttl、volatile- lru、volatile-lfu（Redis 4.0 后新增）四种

所有数据范围内进行淘汰，包括 allkeys-lru、allkeys-random、allkeys-lfu（Redis 4.0 后新增）三种

volatile-ttl: 根据过期时间的先后进行删 除，越早过期的越先被删除

volatile-random: 设置了过期键中进行随机删除.

　　这也就是说，如果一个键值对被删除策略选中了，即使它的过期时间还没到，也需要被删 除。当然，如果它的过期时间到了但未被策略选中，同样也会被删除。

\*\*LRU算法(\*\*最近最少使用 **)**

　　使用链表实现,刚被写入的或者刚被访问的移动到最右端放在MRU左端,其他的依次向右移动.大量key被访问时,需要移动很多节点,较大开销,降低了性能.Redis简化了lru算法: RedisObject 中的 lru 字段记录 每个数据最近被访问的时间戳. Redis 在决定淘汰的数据时，第一次会随机选出 N 个数据，把它们作为一个候选集合。接下来，Redis 会比较这 N 个数据的 lru 字段，把 lru 字段值最小的数据从缓存中淘汰.再次淘汰数据时,需要挑选数据进入第一次淘汰的集合,挑选标准是数据的时间戳小于集合中的最小值.

```
# 配置数据个数N
CONFIG SET maxmemory-samples 100
```

![redis-lru](https://cdn.jsdelivr.net/gh/yunCrush/yc-image/image/redis-lru.png)

使用建议: 数据进行冷热区分则优先使用allkeys-lru,若业务中有置顶要求则使用volatile-lru.同时不给这些置顶数据设置过期时间。这样一来，这些需要置顶的数据一直不会被删除，而其他数据会在过期时根据 LRU 规则进行筛选 .

在使用redis时,数据被修改后就应当写回数据库,redis在删除时,对于脏数据依旧是直接删除.

#### 缓存一致性

先更新数据库,再删除缓存,对于可能删除缓存失败,可放入消息队列,采取重试机制.

先删除缓存,再更新数据库,采用延时双删,由于网络波动A线程在删除缓存后,更新数据库叫慢,线程B直接读取了数据库中的数据写回缓存.但是延时时间并不好控制.

```
// 延时双删延时
redis.delKey(X)
db.update(X)
Thread.sleep(N)
redis.delKey(X)
```

#### 缓存雪崩

设置N(相同过期时间)+n(随机过期时间)过期时间,或者将服务降级进行处理.非核心数据时,直接返回提前预定义的数据,如果是核心数据时,允许查询缓存,若缓存缺失可查询数据库.

除了大量数据同时失效会导致缓存雪崩,Redis 缓存实例发生故障宕机了，无法处理请求，这就会导致大量请求一下子积压到数据库层， 从而发生缓存雪崩

#### 缓存击穿

热点数据突然失效(本来有到无),大量请求直接访问了数据库. 对于访问特别 频繁的热点数据，我们就不设置过期时间了

#### 缓存穿透

[缓存穿透解决方案](https://www.yuncrush.com/system-design/high-concurrency-design#huan-cun-pian)

#### 缓存污染

有些数据被访问的次数非常少，甚至只会被访问一 次。当这些数据服务完访问请求后，如果还继续留存在缓存中的话，就只会白白占用缓存 空间。这种情况，就是缓存污染。 缓存污染严重时,需要将多余的缓存移除,带来额外的开销.

在明确知道数据被再次访问的情况下，volatile-ttl 可以有 效避免缓存污染。在其他情况下，volatile-random、allkeys-random、volatile-ttl 这三 种策略并不能应对缓存污染问题

只看数据的访问时间，使用 LRU 策略在处理扫描式单次查询操作时，无 法解决缓存污染。所谓的扫描式单次查询操作，就是指应用对大量的数据进行一次全体读 取，每个数据都会被读取，而且只会被读取一次.

因此提出了LFU淘汰策略.LFU 策略中会从两个维度来筛选并淘汰数据：一是，数据访问的时效性 （访问时间离当前时间的远近）；二是，数据的被访问次数

LFU: 为每个数据增加了一个计数器,根据访问次数淘汰数据,若访问次数相同,则根据访问时间,淘汰距离当前时间更久的数据淘汰.优先根据访问次数进行比较淘汰,可以避免缓存污染问题.

LFU实现: Redis 在实现 LFU 策略的时候，只是把原来 24bit 大小的 lru 字段，又进一 步拆分成了两部分 ,如下,所以先比较后8位,再比较前.

ldt 值：lru 字段的前 16bit，表示数据的访问时间戳；

counter 值：lru 字段的后 8bit，表示数据的访问次数但是8位的计数,最大值只有255,若被访问了1024也只能记录255.

因此redis采用了一个更优化的计数规,而不是直接counter+1.简 单而言就是通过一个配置项 lfu\_log\_factor 避免counter变为255. lfu\_log\_factor设置为100,可以解决实际访问次数小于 10M 的不同数据,都可以通过 counter 值区分出来

#### Redis实现分布式锁

redis在应对并发操作时,一个客户端线程设置一个变量是,读取-修改-写回redis.不是一个原子操作,可以通过`INCR/DECR`实现增减

```
INCR id
DECR id
使用Lua脚本的EVAL,脚本lua.script内容如下
local current
current = redis.call("incr",KEYS[1])
if tonumber(current) == 1 then
	redis.call("expire",KEYS[1],60)
end
执行
redis-cli --eval lua.script keys , args
```

分布式锁的加锁和释放锁的过程需要保证原子性.保证锁的可靠性,避免因为共享存储系统宕机,客户端无法操作锁.

**单个 Redis 节点实现分布式锁**

加锁逻辑: 读取锁变量,判断变量值,以及把锁变量值设置为1. 保证原子性可以使用Redis的单命令操作或者是Lua脚本.

```
// 加锁,key不存在会被创建,并把val设置为value,存在则不操作
SETNX key value
// 业务逻辑
do something();
// 释放锁
DEL key
```

☆☆☆风险: 加锁后宕机,其他线程无法获取锁,需要对锁添加一个有效期.同时需要确保不会误释放锁,保证只能加锁的线程释放锁,所以需要添加一个唯一标志符. PX表示10s后过期

```
// 加锁, unique_value作为客户端唯一性的标识
SET lock_key unique_value NX PX 10000
```

```
//释放锁 比较unique_value是否相等，避免误释放KEYS[1]表示lock_key，ARGV[1]是当前客户端的唯一标识，这两个值都是执行 Lua 脚本时作为参数传入的。
if redis.call("get",KEYS[1]) == ARGV[1] then
	return redis.call("del",KEYS[1])
else
	return 0
end


```

**基于多个 Redis 节点实现高可靠的分布式锁**

RedLock思路

1. 客户端获取当前时间
2.  客户端按顺序依次向 N 个 Redis 实例执行加锁操作

    客户端在和一个 Redis 实例请求加锁时，一直到超时都没有成功，那么此时，客户端 会和下一个 Redis 实例继续请求加锁。加锁操作的超时时间需要远远地小于锁的有效时 间，一般也就是设置为几十毫秒
3.  客户端完成了和所有 Redis 实例的加锁操作，客户端就要计算整个加锁过 程的总耗时

    加锁成功条件:

    1. 客户端从超过半数（大于等于 N/2+1）的 Redis 实例上成功获取到了锁
    2. 加锁总耗时小于锁的有效期

满足两个条件后,重新设置锁的有效期,原有效期时间 - 加锁总耗时, 若新的有效期来不及完成共享数据的操作可释放锁

#### 事务机制

**原子性**:redis单个命令可以保证原子性

Redis分为三步实现,MULTI(开启事务) -> crud -> EXEC(提交事务)

redis会将命令暂存入命令队列可以使用DISCARD,自动放弃事务,清空命令队列.

如果在执行EXEC命令时,失败,开起了AOF日志的话,可以通过redis-check-aof将已经执行完的部分AOF日志删除.在执行EXEC时,才会真正的将命令队列中的命令取出执行,写到AOF日志中.

命令入队时没报错,在执行时报错,是无法保证原子性的,事务只会执行成功的语句,且无法回退事务.

命令在入队时报错,可以保证原子性.

**一致性**: 执行事务的前后,数据库的数据应当保持一致.

从命令入队报错,命令入队不报错,执行EXEC实例发生故障(是否开起了RDB和AOF)分析,都可保证一致性

　　入队报错,事务失败,保证一致性,命令不报错,正确的命令得到执行,错误的命令不执行,保证一致性.

　　未开启RDB/AOF,发生故障重启后,数据都没了,保证一致性

　　开启RDB/AOF,在执行事务时,不会发生RDB,AOF日志可通过redis-check-aof工具,清理执行完的部分AOF日志,可保证一致性.

**隔离性**: 数据库在执行一个事务时,其它操作无法存取到正在执行事 务访问的数据.

　　事务执行又可以分成命 令入队（EXEC 命令执行前）和命令实际执行（EXEC 命令执行后）两个阶段

命令实际执行可以保证隔离性.

命令入队需要判断是否使用了Watch机制,Watch在事务执行前，监控一个或多个键的值变化情况，当事务调用 EXEC 命令执行时，WATCH 机制会先检查监控的键是否被其它客户端修改了。如果修改 了，就放弃事务执行，避免事务的隔离性被破坏.客户端再次执行事务即可.

**持久性**

　　无论怎么实现都无法保证持久性.都会存在数据丢失,执行完事务,但是下一次的RDB还未执行.开启AOF,可能存在丢失1s的数据,或者在执行完命令后,写AOF日志到硬盘宕机.

#### 主从同步

导致数据不一致,主从库间的命令复制是异步进行的.从库可能受网络延迟,接收主库的命令慢,还可能受其他耗时阻塞命令导致,来不及处理主库的命令.

可以开发一个监控程序,监控进度. Redis 的 INFO replication 命令可以查看主库接收写命令的进度信息.大于max\_offset设定的阈值后,客户端不再从从库读取数据,减少读取到的脏数据,并且可减缓从库压力.

![](https://cdn.jsdelivr.net/gh/yunCrush/yc-image/image/redis-master-slave-sync.png)

**读取过期数据**

Redis使用了惰性删除(在有请求来读写数据时,判断数据是否过期,导致大量过期数据留在内存中,减少删除操作对CPU的利用)和定期删除策略(每隔一段时间,随机删除部分数据).redis尽量使用3.2版本,对于从库读取到的过期数据,并不会删除.会返回null在3.2之前并不会判断过期.

EXPIRE 和 PEXPIRE：它们给数据设置的是从命令执行时开始计算的存活时间； EXPIREAT 和 PEXPIREAT：它们会直接把数据的过期时间设置为具体的一个时间点

　　使用EXPIREAT/PEXPIREAT 设置的是 时间点，所以，主从节点上的时钟要保持一致，具体的做法是，让主从节点和相同的 NTP 服务器（时间服务器）进行时钟同步.

**不合理配置项导致的服务挂掉**

protected-mode 配置项,设置为yes后,哨兵实例不可被其他节点访问,哨兵无法判断主库是否下线,若主库宕机,无法进行主从切换.

cluster-node-timeout 配置项设置了 Redis Cluster 中实例响应心跳消息的超时时间,若从实例切换主实例时间较长会导致实例的心跳超时,进而会被Redis Cluster判断为异常.建议设置为10-20s.

#### 脑裂:数据丢失

在主从集群中，同时有两个主节点，它们都能接收写请求,客户端不知道应该往哪个主节点写入数据.

　　可以利用master\_repl\_offset和slave\_repl\_offset的差值来判断是否是因为数据未同步完全,主节点挂掉,导致从节点升级为主节点导致的数据丢失.

　　原主库假故障导致的脑裂，部分客户端依旧和原主库进行连接，写入数据．

![主库假故障示意图](https://cdn.jsdelivr.net/gh/yunCrush/yc-image/image/redis-%E8%84%91%E8%A3%82.png)

　　主从切换后，从库升级为新主库，哨兵让原主库执行 slave of 命令，和新主库重新进行全量同步。而在全量同步执行的最后阶段，原主库需要清空本地的数据(降级为从库,需要清空数据)，加载新主库发送的 RDB 文件，原主库在主从切换期间保存的新写数据就丢失了

**解决脑裂问题**

　　min-slaves-to-write：这个配置项设置了主库能进行数据同步的最少从库数量N；

　　min-slaves-max-lag：这个配置项设置了主从库间进行数据复制时，从库给主库发送 ACK 消息的最大延迟时间T（以秒为单位 ),原主库无法和哨兵通信,自然无法和从库通信,不满足时间T,因此就不允许客户端往原主库写入数据,等到主从切换完毕,新主库接收客户端请求即可.

#### 数据倾斜

大key导致的,可以将大key进行拆分成很多个小的集合类型

**Slot 分配不均衡导致倾斜**

slot是存放数据的单位,Slot迁移方案:

### 运维
