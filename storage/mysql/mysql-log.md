---
description: 这里主要介绍数据库的redo log与binlog日志。
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

# MySQL日志篇

## Redo log（重做日志）

　　这里先引用原文中的MySQL逻辑架构图。查询流程与更新流程不同，更新流程涉及到redo log与binlog日志模块。在执行语句前，会进行数据库的连接这是连接器的工作。更新一个表时，查询缓存会全部失效。

![MySQL逻辑架构图](<../../.gitbook/assets/image (11) (1).png>)

　　**redo log是InnoDB引擎所特有的，是物理日志，记录的是“在某个数据页上面做了什么改动”，采用的是WAL技术（Write-Ahead logging)，先写进日志，再写到磁盘，一般是在空闲时写入磁盘，空间固定会写完，如果写完时，需要暂停，进行檫除之前日志**。redo日志的概念图是环状的，循环写入的，如下图所示，有了redo日志，redo不能记录历史的日志，不能做归档，所以不可以单靠redo日志来进行数据的恢复，即使数据库发生异常重启，也不会导致之前的提交记录的丢失，这种能力叫**crash-safe**。

![](<../../.gitbook/assets/image (12).png>)

　　write pos日志的写入点，check point日志的檫除点，数据檫除前，需要将数据写入到数据文档中，两者中间部分表示还剩余的空间可以用来写日志。如果write pos追上check point，不能执行新的更新，需要停下来顺时针檫除。

　　innodb\_flush\_log\_at\_trx\_commit 这个参数设置成 1 的时候，表示每次事务的 redo log 都直接持久化到磁盘。

## Binlog（归档日志）

　　**Binlog日志是Server层实现的，所有引擎都可以使用，Binlog是逻辑日志，采用的是追加写**的模式**Binlog有两种模式，statement 格式的话是记sql语句， row格式会记录行的内容，记两条，更新前和更新后都有。**

　　\*\*\*\*　　sync\_binlog 这个参数设置成 1 的时候，表示每次事务的 binlog 都持久化到磁盘。这个参 数我也建议你设置成 1，这样可以保证 MySQL 异常重启之后 binlog 不丢失。

## 两阶段提交

```
update T set c=c+1 where ID=2;
```

　　关于与update的执行流程如下图所示：

![update执行流程](<../../.gitbook/assets/image (13) (1).png>)

　　redolog拆分为两部分，prepare和commit即\*\*“两阶段提交"，两阶段提交是为了保证两份日志的逻辑一致。\*\*

　　**update**的内部执行流程：

　　首先执行器找到引擎查找ID=2这一行，ID是主键，直接通过树搜索进行查找，如果这一行的数据页在内存中就，直接返回给执行器；否则需要先从磁盘进行读入内存，然后返回给执行器。

　　执行器拿到引擎给的数据，把当前值+1，得到新的一行数据，然后调用引擎接口写入数据。

　　引擎将这行数据更新到内存中，同时将这个更新操作记录到redo log里面，此时redo log处于prepare阶段，然后告诉执行器已经执行完成，随时可以进行事务的提交。

　　执行器生成这个操作的binlog，并把binlog写入磁盘，然后调用引擎的提交事务接口，把刚刚写入的redo log改成提交状态，更新完成。

## 提问

1.redo log和binlog怎么关联起来的？

　　它们有一个共同的数据字段，叫 XID。崩溃恢复的时候，会按顺序扫描 redo log：

　　如果碰到既有 prepare、又有 commit 的 redo log，就直接提交；

　　如果碰到只有 parepare、而没有 commit 的 redo log，就拿着 XID 去 binlog 找对应 的事务。

2\. 如何知道binlog是完整的？

binlog有两种格式，statement格式：最后有COMMIT，row格式：最后有XID EVENT。
