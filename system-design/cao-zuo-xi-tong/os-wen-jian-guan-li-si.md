---
description: 操作系统文件管理部分
---

# OS文件管理（四）

常见的文件类型：普通、目录、可执行、管道、Socket、软链接、硬链接。ls -F 查看。

三种文件系统：早期的 FAT（File Allocate Table） 格式 基于 inode 的传统文件系统 日志文件系统（如 NTFS, EXT2、3、4）

FAT 通过一个链表结构解决了文件和物理块映射的问题。为了改进 FAT 的容量限制问题，引入索引节点Inode，当文件导入内存的时候，先导入索引节点（inode），然后索引节点中有文件的全部信息，包括文件的属性和文件物理块的位置。

硬连接： b.txt与c.txt同一个文件拥有不同的名称，在不同目录下，删除其中一个另外一个正常运行，如果要删除inode，则需要都删除。

![硬链接示意图](<../../.gitbook/assets/image (19).png>)

```
# 为a创造一个硬链接b
ln a b
```

软连接： 拥有自己的inode，但是文件内容就是一个快捷方式，如果删除了源inode，自己inode不会消失，指向的空地址。

![软连接示意图](<../../.gitbook/assets/image (20).png>)

```
# 将b设置为a的软链接,b是a的快捷方式
ln -s a b 
```

日志文件系统： 日志结构简单、容易存储、按时间容易分块，这样的设计非常适合缓冲、批量写入和故障恢复。
