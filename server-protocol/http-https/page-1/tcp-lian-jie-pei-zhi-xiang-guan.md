# TCP连接配置相关

### TCP连接配置相关

　　tcp连接如图：\
[<img src="https://user-images.githubusercontent.com/46308363/204945959-c5362f2e-3490-4e2f-9a20-761d0f14aec2.png" alt="image" data-size="original">](https://user-images.githubusercontent.com/46308363/204945959-c5362f2e-3490-4e2f-9a20-761d0f14aec2.png)\
　　客户端发起connect()后，就开始第一次握手，tcp的第一次握手失败，会进行重试，重试次数取决于客户端的配置`net.ipv4.tcp_syn_retries = 2` 重试次数小一点，可以减少阻塞等待时间。\
　　**半连接，即收到了 SYN 后还没有回复 SYNACK 的连接**.每收到一个连接请求，会将半连接放入队列中syn queue。队列的长度是由`tcp_max_syn_backlog` 决定，如果超过了这个数值后，新的syn就会抛弃。对服务器而言应当尽可能的调大这个值。`net.ipv4.tcp_max_syn_backlog = 16384`\
　　防止泛洪攻击，Linxu内核引入了SYN Cookies机制，即，收到的第一次握手收到的syn包并不会分配资源去保存client信息，而是根据syn包信息，算出一个cookie值，返回给Client,对于正常的连接，这个cookie值会在下次ACK请求中带上，来判断是否是正常的请求。开启Cookies机制：`net.ipv4.tcp_syncookies = 1`\
　　SYN ACK也可能丢失，服务端的重试次数配置：`net.ipv4.tcp_synack_retries = 2`.完成3次握手后，产生了一个 TCP 全连接（complete），它会被添加到全连接队列（accept queue）中。然后 Server 就会调用 accept() 来完成 TCP 连接的建立。\
　　\*\*全连接队列的长度也是有限制的，避免Server调用accept（）来不及，造成资源的浪费，对于超过了队列长度的Client会进行抛弃，正常是直接抛弃，可根据配置项决定是否给客户端发送reset，发送reset通知客户端不需要重试了。默认配置是0，`net.ipv4.tcp_abort_on_overflow = 0` 不通知Client，相当于给了客户端重试的机会。\


<figure><img src="https://user-images.githubusercontent.com/46308363/204949249-26783590-69e1-4960-8dde-f2f219a8278c.png" alt=""><figcaption></figcaption></figure>

### TCP断开配置相关

\
　　**额外关注三个红色的状态**，除了CLOSE\_WAIT没有配置来控制外，其余的两个都可以进行配置控制。`net.ipv4.tcp_fin_timeout = 2` FIN\_WAIT\_2 状态，本端没有收到FIn包， 避免对端，太忙无法close()，消耗系统资源。超时时间默认60s，设置小一点，超过超时时间自动销毁掉连接。限制处在TIME\_WAIT的个数：`net.ipv4.tcp_max_tw_buckets = 10000`可以设置小一点。 TCP端口最多只有65535个，Client与Server断开连接后，很可能需要再次连接，如果不复用TIME\_WAIT状态，可能出现再次就连接不上。打开配置：`net.ipv4.tcp_tw_reuse = 1`\


<figure><img src="https://user-images.githubusercontent.com/46308363/204952332-49d543d9-7f40-4ba2-9c57-743024e13484.png" alt=""><figcaption></figcaption></figure>

<figure><img src="https://user-images.githubusercontent.com/46308363/204949436-382d1d7e-713b-4d0b-8398-7f54e263cc69.png" alt=""><figcaption></figcaption></figure>
