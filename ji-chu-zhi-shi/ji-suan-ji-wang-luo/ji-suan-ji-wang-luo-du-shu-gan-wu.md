---
description: 这部分关于《计算机网络-自顶向下方法》的阅读感悟。
---

# 计算机网络读书感悟

## 1.应用层

## 2.传输层

　　多路复用与多路分解：将主机间的交付扩展到进程间的交付。

　　套接字是应用层的进程与运输层传递数据的门户，所以可以理解为套接字存在应用层与运输层之间，每一个套接字会绑定一个端口，一个主机可能会存在很多歌个套接字，所以每个套接字都有唯一的标志符。源主机从不同的套接字收集数据块，并为每个数据块添加首部，首部包括主机的源IP、源端口号、目的IP、目的端口号，由此生成报文段，然后将报文段传递给网络层，这些工作称为多路复用。

　　运输层拿到报文段解析，通过端口号，找到对应的套接字，将数据交付给套接字，称之为多路分解。



## 3.网络层