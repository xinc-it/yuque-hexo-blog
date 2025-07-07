---
title: Mysql-行锁
date: '2022/9/11 21:17:06'
updated: '2025-07-06 11:10:16'
categories: 数据库
tags:
  - Mysql
  - 数据库
  - InnoDB
---
## 1. 两阶段锁


![](/images/4d74fdbeabaa7ffb81553d744d540a0a.png)

**<font style="color:rgb(53, 53, 53);">下面的操作序列中，如果为两行数据加了行锁事务 B 的 update 语句执行时会是什么现象呢？假设字段 id 是表 t 的主键。</font>****  
**

****

在InnoDB中，行锁是在需要的时候加上，在**<font style="color:#F5222D;">事务结束时释放。（</font>**最有可能造成锁冲突的行的读锁尽量往后放**<font style="color:#F5222D;">）</font>**

**<font style="color:#F5222D;"></font>**

示例：

顾客A在影院B买电影票

操作如下

1. <font style="color:rgb(53, 53, 53);">从顾客 A 账户余额中扣除电影票价；</font>
2. <font style="color:rgb(53, 53, 53);">给影院 B 的账户余额增加这张电影票价；</font>
3. <font style="color:rgb(53, 53, 53);">记录一条交易日志。</font>

<font style="color:rgb(53, 53, 53);"></font>

其中多个顾客买票可能造成B的账户行数据冲突。因此将B的操作放在最后。最大程度减少了事务之间的锁等待。





## 2. 死锁和死锁检测


示例：当行锁同时锁住id=1和2的两行数据

![](/images/11434486207c9845bd8b0d68a7b58d92.png)

在这种情况下事务A等待id=2的行锁，事务B等待id=1的行锁。事务A和事务B互相都在等待对方无法释放的资源，从而进入了死锁。





解决方法

+ 设置获取锁等待时间：设置获取锁超时时间，如果超时则锁住的线程自动退出。通过innodb_lock_wait_timeout来设置等待时间默认为50s
+ 发起死锁检测：发现死锁后主动回滚争抢锁的某一个事务。让其他事务继续执行。通过innodb_deadlock_detect 设置为on表示开启逻辑。





死锁检测的缺点：

1. 确保业务不会出现死锁，关闭死锁检测
2. 控制并发度







## 3. 小结




1. 读锁的两阶段协议：需要的时候加上、事务结束的时候释放。尽可能将影响并发度最大的锁往后放。减少其他锁的等待时间
2. 死锁的形成，事务之间争抢不释放的锁资源。导致一直处于等待状态
3. 死锁的解决方法：1. 设置锁等待时间 2. 开启死锁检测

**<font style="color:#F5222D;"></font>**

**<font style="color:#F5222D;"></font>**

+ <font style="color:rgb(53, 53, 53);">第一种，直接执行 delete from T limit 10000;</font>
+ <font style="color:rgb(53, 53, 53);">第二种，在一个连接中循环执行 20 次 delete from T limit 500;</font>
+ <font style="color:rgb(53, 53, 53);">第三种，在 20 个连接中同时执行 delete from T limit 500。</font>

<font style="color:rgb(53, 53, 53);">你会选择哪一种方法呢？为什么呢？</font>

<font style="color:rgb(53, 53, 53);">第二种，第三种容易造成死锁。第一种锁的范围太大导致获取锁的时间长。</font>

