# yunCrush/Redis

### Redis

1. 基础\
   　　3.0是单线程，4.0支持混合持久，多线程异步删除，6.0支持多线程io redis单线程是指网络的IO与键值的读写是一个线程执行的读取socket->解析请求->处理->写入socket. 3.0单线程快的原因：使用多路复用与非阻塞IO，多路复用监听多个socket连接客户端，这样可以使用一个线程来处理多个请求，避免上下 文的切换，单线程模型避免了加锁阻塞，省去了时间和性能上面的开销。大key删除导致阻塞，引入了异步概念。\
   　　6.0的redis通过使用多个线程处理网络IO，单个线程处理主要的工作，既解决了网络IO问题，也保证了不用上下文切换的开销。默认是关 闭多线程的`io-threads-do-reads no` 可开启\

2. 基础
   * 2.1 mysql与redis一致：先写(update)mysql,从mysql查询出数据再将查询结果写入到redis(先操作mysql再操作redis),查询的时候 先查询redis,有则直接返回，无则查询mysql;查询mysql无，直接返回null,有则更新redis，保证下一次的缓存命中率。 `redis-cli --raw`解决客户端乱码
   *   2.2 借鉴DCL思想，解决缓存击穿问题

       ```
       public User findUserById2(Integer userId) {
               User user = null;
               //缓存key
               String key = CACHE_KEY_USER + userId;
               //1 查询redis
               user = (User) redisTemplate.opsForValue().get(key);
               //redis无，进一步查询mysql
               if (user == null) {
                   // DCL思想 保证第一个线程进来
                   synchronized (UserService.class) {
                       user = (User) redisTemplate.opsForValue().get(key);
                       if (user == null) {
                           user = userMapper.selectByPrimaryKey(userId);
                           if (user == null) {
                               return user;
                           } else {
                               // 写到Redis
                               redisTemplate.opsForValue().setIfAbsent(key, user, 7L, TimeUnit.DAYS);
                           }
                       }
                   }
               }
               return user;
       ```
3. 经典5种数据类型
   * 3.1 String 做点赞+1(incr count),或者踩一下(decr count)
   *   3.2 Hash 结构相当于java的Map\<String,Map\<Object,Object>> 解决简单购物车问题

       ```
       新增商品3344: hset shopcar:uid1024 3344 1
       新增商品3345: hset shopcar:uid1024 3345 1
       增加商品数量: hincrby shopcar:uid1024 3344 1
       商品总数： hlen shopcar:uid1024
       全部选择： hgetall shopcar:uid1024
       ```
   *   3.3 List 双端链表结构长度2^32 -1 大概40亿

       ```
        lpush key value
        rpush key value
       查看列表，可做分页 lrange key start stop
       获取列表中元素个数 len key
       应用：公众号订阅消息，我订阅了A和B两个作者，AB分别发布了a1,b1文章，只要发布文章则 lpush likearticle:yun a1 b1
       查看自己订阅号的全部文章，类似分页: lrange likearticle:yun 0 9 每次展示10条 
       ```

       需求1：用户针对某一商品发布评论，一个商品会被不同的用户进行评论，保存商品评论时，要按时间顺序排序 需要2：用户在前端页面查询该商品的评论，需要按照时间顺序降序排序
   *   3.4 set

       ```
       随机弹出一个元素不删除：srandmember set
       获取总数:scard set
       添加：sadd set 1
       删除：srem set 1
       seta - setb 差集: sdiff seta setb
       seta ∩ setb 交集：sinter seta setb
       seta ∪ setb 并集：sunion seta setb  
       ```

       **应用**:

       *   1.微信抽奖

           ```
           a.点击参加抽奖： sadd key userid
           b.显示多少人参与：scard key
           c.随机抽奖：srandmember key 2 随机弹出2个userid 不删除 spop key 3 随机抽3个，删除
           ```
       *   2.朋友圈点赞

           ```
           a.点赞：sadd pub:ID id1 id2
           b.点赞取消: srem pub:ID id1
           c.显示点赞过的用户: smembers pub:ID
           d.统计多少人点赞了：scard pub:ID
           e.判断某个成员是否点赞：sismember pub:ID id1  
           ```
       *   3.社交关系中

           ```
           共同好友,判断共同关注的人(交集)：sinter me you
           qq内推可能认识的人: sdiff A B , sdiff B A 相互内推对方没有的
           ```
   *   3.5 zset(sorted set)

       ```
       向有序的集合中添加一个元素和元素的分数：zset key score member 
       获取元素的排名小到大：zrank key member
       获取元素的排名大到小：zrevrank key member
       指定分数范围内的元素个数：zcount key min max
       ```

       **应用**:

       *   1.根据商品销售对商品进行排序

           ```
           key为goods:sellsort，分数为商品销售数量
           商品编号1001的销量是9，商品编号1002的销量是15: zadd goods:sellsort 9 1001 15 1002
           有一个客户又买了2件商品1001，商品编号1001销量加2: zincrby goods:sellsort 2 1001
           求商品销量前10名: ZRANGE goods:sellsort 0 10 withscores
           ```
   *   3.6 bitmap\
       　　01表示的数组，底层是String类型，STRLEN求的是字节长度，bitmap默认是8位一组，超过扩容8

       ```
       setbit key1 offset value  value只可为0,1
       setbit key1 1 1  setbit key1 2 1 setbit key1 7 1 使用get 得到的是'a'底层对应的是String类型，"1100001"
       get key1 ==> 1 得到的是1字节，8位。
       getbit key1 2  ==> 1 得到2号位置的值是1
       bitcount key1 判断底层字符串中有多少1
       bitop 用来对一个或多个二进制位进行 位操作，例子：
       0 0 0 0 1 1 1 (第一天哪些用户签到了5,6,7)
       0 0 0 0 1 1 0 (第二天哪些用户签到了5,6)
       -------------
       0 0 0 0 1 1 0  (连续两天签到的用户5,6) bitop and destykey k1 k2 --> bitcout destkey
       ```

       \
       **需求**: 钉钉签到，某部电影是否被某个人点赞过，用户是否登录过app,连续签到等等。
   *   3.7 hyperloglog(基数统计)\
       为什么slot是16384?\
       实质String\
       uv: Unique Visitor,客户端IP,需要去重\
       pv: PageView 页面访问量\
       DAU: daily active user 日活跃(去重复登录的用户)\
       MAU: monthly active user 月活跃

       ```
       pfadd taobao:uv1 111 112 113
       pfadd taobao:uv2 111 112  114 115
       pfcount taobao:uv1   ==> 3
       pfmerge result taobao:uv1 taobao:uv2
       pfcount result       ==> 5
       ```

       * 1.需求\
         　　a.用户搜索的关键词的数量; b.统计用户每天搜索不同词条个数\
         　　bitmap不适合亿级数据量,但是是精确的,hyperloglog误差0.81%
       * 2.实战\
         　　天猫首页亿级UV统计方案:\
         　　淘宝、天猫首页的UV，平均每天是1\~1.5个亿左右,每天存1.5个亿的IP，访问者来了后先去查是否存在，不存在加入\
         **方案1: redis hash**\
         　　redis——hash = \<keyDay,\<ip,1>>\
         　　按照ipv4的结构来说明，每个ipv4的地址最多是15个字节(ip = "192.168.111.1"，最多xxx.xxx.xxx.xxx)\
         　　某一天的1.5亿 \* 15个字节= 2G，一个月60G，redis死定了。o(╥﹏╥)o
   * 3.8 GEO\
     　　地球上的地理位置是使用二维的经纬度表示，经度范围 (-180, 180]，纬度范围 (-90, 90]，只要我们确定一个点的经纬度就可以名曲他在地球的位置。 例如滴滴打车，最直观的操作就是实时记录更新各个车的位置，实质Zset
     *   1.命令

         ```
         添加经纬度
         GEOADD city 116.403963 39.915119 "天安门" 116.403414 39.924091 "故宫" 116.024067 40.362639 "长城"
         获取经纬度
         GEOPOS city 天安门 故宫
         获取对应hash值
         GEOHASH city 天安门
         获取距离
         GEODIST city 天安门 故宫 m
         GEORADIUS 以半径查找
         ```

         ```
         georadius 以给定的经纬度为中心， 返回键包含的位置元素当中， 与中心的距离不超过给定最大距离的所有位置元素。   
         GEORADIUS city 116.418017 39.914402 10 km withdist withcoord count 10 withhash desc
         116.418017 39.914402 自己经纬度

         WITHDIST: 在返回位置元素的同时， 将位置元素与中心之间的距离也一并返回。 距离的单位和用户给定的范围单位保持一致。
         WITHCOORD: 将位置元素的经度和维度也一并返回。
         WITHHASH: 以 52 位有符号整数的形式， 返回位置元素经过原始 geohash 编码的有序集合分值。 这个选项主要用于底层应用或者调试， 实际中的作用并不大
         COUNT 限定返回的记录数。
         ```
     * 2.实战\
       　美团地图位置附近的酒店推送\
       　摇一个妹纸，共享单车
4.  布隆过滤器\
    　　它实际上是一个很长的二进制数组+一系列随机hash算法映射函数，主要用于判断一个元素是否在集合中。 现有50亿个电话号码，现有10万个电话号码，如何要快速准确的判断这些电话号码是否已经存在\
    **解决缓存穿透问题**\
    　　返回存在就不一定存在(因为hash冲突导致的)，返回不存在就一定不存在，布隆过滤器可以添加元素，不可以删除元素会增加误判。**添加元素时，把某个位置置为1**

    布隆过滤器3步骤：初始化，添加，判断是否存在\
    　**1.初始化:** 布隆过滤器 本质上 是由长度为 m 的位向量或位列表（仅包含 0 或 1 位值的列表）组成，最初所有的值均设置为 0\
    　**2.添加:** 当我们向布隆过滤器中添加数据时，为了尽量地址不冲突，会使用多个 hash 函数对 key 进行运算，算得一个下标索引值，然后对位数组长度进行取模运算得到一个位置，每个 hash 函数都会算得一个不同的位置。再把位数组的这几个位置都置为 1 就完成了 add 操作。\
    　**3.判断是否存在:** 向布隆过滤器查询某个key是否存在时，先把这个 key 通过相同的多个 hash 函数进行运算，查看对应的位置是否都为 1， 只要有一个位为 0，那么说明布隆过滤器中这个 key 不存在； 如果这几个位置全都是 1，那么说明极有可能存在； 因为这些位置的 1 可能是因为其他的 key 存在导致的，也就是前面说过的hash冲突\
    **为什么不可删除：** 因为每个位置是共同使用过的，如果删除了对其他的key有误判，其他key,存在，但是已经被置为0了 **优点：** 高效插入和查询，占用空间少\
    **缺点：** 不可删除，会造成误判\
    google guava：默认0.03误判率，100W个数，占用950W bit位，5个hash函数。**仅单机** **白名单架构**\
    \
    **布隆过滤器** \
    **黑名单使用**&#x20;
5. 缓存击穿-缓存雪崩-缓存穿透
   * 1.缓存击穿：热key突然失效，暴打Mysql\
     　a.访问频繁key,不设置过期时间\
     　b.互斥更新，差异失效时间\
     　c.互斥独占锁防止击穿
   * 2.缓存雪崩：大量可以失效\
     　 a. 限流、降级、redis集群
   *   3.缓存穿透：redis和mysql都没有\
       　a.空对象缓存或者缺省值\
       　b.Guava缓存google布隆过滤器\
       　c. redisson布隆过滤器\
       **聚划算高并发解决缓存击穿**\
       　7\*24高并发，实时。分层类别，需求排期，每个时间段，展示不同商品

       ```
       1.100%高并发，绝对不可以用mysql实现
       2.先把mysql里面参加活动的数据抽取进redis，一般采用定时器扫描来决定上线活动还是下线取消。
       3.支持分页功能，一页20条记录
       redis里面什么样子的数据类型支持上述功能？
       ```

       解决方案：&#x20;
6.  分布式锁

    ```
    Redis除了拿来做缓存，你还见过基于Redis的什么用法？ ==>分布式锁
    Redis 做分布式锁的时候有需要注意的问题？ ==>分布式锁续期
    如果是 Redis 是单点部署的，会带来什么问题？如何解决  ==>宕机
    集群模式下，比如主从模式，有没有什么问题呢？ ==>异步复制一主二从，master down，丢失lock
    你知道 Redis 是怎么解决集群模式也不靠谱的问题的吗？  ==> 多master
    ```

    * 1.分布式锁的条件和刚需
      * a.独占性: OnlyOne，任何时刻只能有且仅有一个线程持有
      * b.高可用: 若redis集群环境下，不能因为某一个节点挂了而出现获取锁和释放锁失败的情况
      * c.仿死锁: 杜绝死锁，必须有超时控制机制或者撤销操作，有个兜底终止跳出方案
      * d.不乱抢: 防止张冠李戴，不能私下unlock别人的锁，只能自己加锁自己释放。
      * e.重入性: 同一个节点的同一个线程如果获得锁之后，它也可以再次获取这个锁。
    * 2.单机Redis单节点实现分布式锁常见问题
      *   a.超卖：synchronized lock 加锁

          ```
          synchronized (this) {
              // todo
              }
          ```
      *   b.nginx分布式微服务架构: 加锁只是针对本地单机层面，仍旧出现超卖，需要redis分布式锁

          ```
            String key = "zzyyRedisLock";
            String value = UUID.randomUUID().toString()+Thread.currentThread().getName();
            Boolean flagLock = stringRedisTemplate.opsForValue().setIfAbsent(key, value);
          ```
      *   c.finally必须关闭锁资源:出异常的话，可能无法释放锁，必须要在代码层面finally释放锁

          ```
          try {
              Boolean flagLock = stringRedisTemplate.opsForValue().setIfAbsent(key, value);
             } finally {
              stringRedisTemplate.delete(key);     
              } 
          ```
      *   d.宕机：部署了微服务jar包的机器挂了，代码层面根本没有走到finally这块，没办法保证解锁，这个key没有被删除，需要加入一个过期时间限定key

          ```
          Boolean flagLock = stringRedisTemplate.opsForValue().setIfAbsent(key, value);
          stringRedisTemplate.expire(key,10L,TimeUnit.SECONDS);    
          ```
      *   e.原子性问题：设置key+过期时间分开了，必须要合并成一行具备原子性

          ```
          Boolean flagLock = stringRedisTemplate.opsForValue().setIfAbsent(key,value,10L,TimeUnit.SECONDS);
          ```
      *   f.误删问题：删除别人的锁，通过value值判断当前是否是自己拥有的锁

          ```
          finally {
              if (stringRedisTemplate.opsForValue().get(key).equals(value)) {
                  stringRedisTemplate.delete(key);
              }
          }
          ```
      *   g.原子性问题: finally块的判断+del删除操作不是原子性的

          ```
          // 使用Lua脚本
          finally {
                    Jedis jedis = RedisUtils.getJedis();
                    String script = "if redis.call('get', KEYS[1]) == ARGV[1] " +
                            "then " +
                            "return redis.call('del', KEYS[1]) " +
                            "else " +
                            "   return 0 " +
                            "end";
                    try {
                        Object result = jedis.eval(script, Collections.singletonList(REDIS_LOCK_KEY), Collections.singletonList(value));
                        if ("1".equals(result.toString())) {
                            System.out.println("------del REDIS_LOCK_KEY success");
                        }else{
                            System.out.println("------del REDIS_LOCK_KEY error");
                        }
                    } finally {
                        if(null != jedis) {
                            jedis.close();
                        }
                    }
                }
          ```
      * h.时间过期：确保redisLock过期时间大于业务执行时间的问题
    * 3.Redis集群实现分布式锁:RedLock之Redisson 　　redlock是一种使用redis实现分布式锁的算法 [http://redis.cn/topics/distlock.html](http://redis.cn/topics/distlock.html)\
      　　 redis单机：CP\
      　　 redis集群：AP redis异步复制造成的锁丢失，比如：主节点没来的及把刚刚set进来这条数据给从节点，就挂了。\
      　　 zookeeper+集群：CP zk重启或者网络故障都会导致集群重新选主，此间client不可注册到zk，所以在大型分布式系统中都很少选用zk.\
      **Redis只支持AP，为了解决CP问题采用多master模式，不是主从或者集群模式**\
      **问题：** 分布式系统受系统时钟影响，如果线程 1 从 3 个实例获取到了锁。但是这 3 个实例中的某个实例的系统时间走的稍微快一点，则它持有的锁会提前过期被释放，当他释放后，此时又有 3 个实例是空闲的，则线程2也可以获取到锁，则可能出现两个线程同时持有锁了。
      *   1.Redisson源码

          ```
          this.lockWatchdogTimeout = 30000L;
          ```
7. Redis的缓存过期淘汰策略
   * 1.数据如何删除的
     * a.立即删除: 到达过期时间，立即删除。对CPU不友好，用处理器性能换取存储空间 （拿时间换空间)
     * b.惰性删除：到达过期时间不做处理，等到下次访问时，判断是否过期，过期则删除，返回null。对内存不友好，如果一个键，过期一直不被访问则一直存在内存中，对redis不友好。
     * c.定期删除：随机抽取，存在漏网之鱼
   *   2.缓存淘汰策略\
       　　由于三种删除策略存在漏洞，诞生了缓存淘汰策略。

       *   a.8种缓存淘汰策略(2个维度\*4个方面)

           ```
           noeviction: 不会驱逐任何key
           allkeys-lru: 对所有key使用LRU算法进行删除
           volatile-lru: 对所有设置了过期时间的key使用LRU算法进行删除
           allkeys-random: 对所有key随机删除
           volatile-random: 对所有设置了过期时间的key随机删除
           volatile-ttl: 删除马上要过期的key
           allkeys-lfu: 对所有key使用LFU算法进行删除
           volatile-lfu: 对所有设置了过期时间的key使用LFU算法进行删除 
           ```
       * b.2个维度
       * c.4个方面

       ```
       LRU 
       LFU 
       TTL 过期时间
       Random 随机
       ```
8.  经典5种数据类型实现

    ```
    1. 单线程如何理解，为什么这样设计？
    2. redis跳跃列表，有什么缺点？
    ```



*   1.跳表：链表+多级索引

    ```
    优缺点：
    1、跳表是一个最典型的空间换时间解决方案，而且只有在数据量较大的情况下才能体现出来优势。而且应该是读多写少的情况下才能使用，所以它的适用范围应该还是比较有限的
    2、维护成本相对要高 - 新增或者删除时需要把所有索引都更新一遍；
    3、最后在新增和删除的过程中的更新，时间复杂度也是O(log n)
    ```

1. Redis-MySQL双写一致性落地
