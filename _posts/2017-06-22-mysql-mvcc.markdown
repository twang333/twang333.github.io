---
layout: post
title:  "MySQL MVCC: 多版本并发控制"
date:   2017-06-22 09:00:05 +0800
categories: bootstrap
---

#### 数据库存储引擎并发控制策略：
* MVCC(Multi-Version Concurrent Control)
  * 读不加锁，读写不冲突。在读多写少的OLTP应用中，读写不冲突是非常重要的，极大的增加了系统的并发性能。
* LBCC(Lock-Based Concurrent Control)

#### MVCC: 快照读和当前读
* 快照读：读取的是记录的可见版本(有可能是历史版本)，不用加锁。比如：
  * select \* from table where?;
* 当前读：读取的是记录的最新版本，会加锁以保证其他事务不会再并发修改这条记录。比如：
  * select \* from table where ? lock in share mode; 共享锁
  * select \* from table where ? for update; 排他锁
  * insert into table values(…); 排他锁
  * update table set ? where ?; 排他锁
  * delete from table where ?; 排他锁

#### <<高性能MySQL>>的引用
MySQL的大多数事务型存储引擎实现的都不是简单的行控制。基于提升并发性能的考虑，它们一般都同时实现了多版本并发控制(MVCC)。可以认为MVCC是行级锁的一个变种，但是它在很多情况下避免了加锁操作，因此开销更低。虽然实现机制有所不同，但大都实现了非阻塞的读操作，写操作也只锁定必要的行。

MVCC的实现，是通过保存数据在某个点的快照来实现的。
* 不管需要执行多长时间，每个事物看到的数据都是一致的。
* 根据事务开始的时间不同，不同事务对同一张表，同一时刻看到的数据可能是不一样的。

InnoDB中的MVCC：
下面看一下在Repeatable read隔离级别下，MVCC具体是如何操作的。
* Select: InnoDB会根据以下两个条件检查每行记录：
 1. InnoDB只查找版本早于当前事务版本的数据行（也就是，数据行的系统版本号小于或者等于事务的系统版本号），这样可以确保事务读取的行，要么是在事务开始前已经存在的，要么是事务自身插入或者修改过的。
 2. 数据行的删除版本要么未定义，要么大于当前事务版本号。这可以确认事务读到的行，在事务开始之前未被删除。
* Insert：InnoDB为新插入的每一行保存当前系统版本号作为行版本号。
* Delete：InnoDB为删除的每一行保存当前系统版本号作为行删除标识。
* Update：InnoDB为插入一行新记录，保存当前系统版本号为行版本号，同时保存当前的系统版本号到原来的行作为行删除标识。

MVCC只在RR和RC两个隔离级别下工作。其他两个隔离级别都和MVCC不兼容。
* RU总是读到最新的数据行，而不是符合当前事务版本的事务行。
* Serializable则会对所有读取的行都加锁。

#### 问题：
mysql是如何做到RR的？比如事务1读一条记录，事务2写同一条记录并提交，事务1再读这条记录，依然读不到这条记录的最新改动。

答案：MVCC. 
* transaction2只能看到第一条记录；
* transaction3只能看到第二条记录。
![Screen Shot 2017-06-20 at 16.53.22.png]({{ site.url }}/assets/AA9F22476705EB12A1B1086927124F19.png)

####  参考：
[http://hedengcheng.com/?p=771\#\_Toc374698316](http://hedengcheng.com/?p=771#_Toc374698316) 锁分析，例子挺好的。

