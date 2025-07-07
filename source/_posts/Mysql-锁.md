---
title: Mysql-锁
date: '2021/9/10 20:17:06'
updated: '2023-01-14 16:02:15'
categories: 数据库
tags:
  - Mysql
  - 数据库
  - InnoDB
---
## 1. 锁的分类：


**根据加锁范围：**

1. **全局锁**
2. **表级锁**
3. **行锁**

****

****

## 2. 全局锁
全局读锁命令

**<font style="color:rgb(53, 53, 53);">全局锁就是对整个数据库实例加锁，Mysql 提供了一个加全局读锁的命令。可以让整个数据库处于只读状态。</font>**

<font style="color:rgb(53, 53, 53);">导致数据的增删改、建表修改表语句、事务的提交语句失效。</font>

```json
Flush tables with read lock
```

场景：**全库的逻辑备份(****<font style="background-color:#F5222D;">也可以开启可重复读事务级别来进行备份</font>****)**

另一种方式是使用mysqldump工具使用参数-single-transaction进行数据库备份。前提：数据库中所有的表支持可重复读。









场景：用户买课和买课后的余额。不加锁导致两个表的数据前后不一致。

![](/images/f55d1b0f5ff1f31a094ccc2ca142c990.png)

## 3. 表级锁


1. 两种表锁



+ 表锁
+ 元数据锁







2. 表锁



作用：**锁定表只能进行读或读和写操作。不允许操作其他表。（****<font style="color:#F5222D;">处理并发的常用方式</font>****）**

```json
 lock tables t1 read, t2 write;
```



只能对t1表进行读，t2表进行读和写。直到执行**unlock tables**之前不允许对其他表进行读写操作。





3. MDL ( meatadata lock)



作用：保证读写的正确性。防止DDL(<font style="color:#F5222D;">加字段等修改表结构的操作</font>)和DML冲突（<font style="color:#F5222D;">增删改数据</font>）

特点： 自动加上，不需要使用sql语句。在对表进行数据的增删改查操作时加读锁。修改表结构时，加写锁。





读读锁共存、读写锁、写写锁互斥。



**<font style="color:#F5222D;">示例</font>**

**<font style="color:#F5222D;"></font>**

![](/images/e22949ac5a1d435763af790da44f9863.png)



在多个事务开启且未结束的过程中给某个表添加个字段导致读写锁冲突。



解决方法

在information_schema 库中 innodb_trx 表中查询出正在执行事务，可以暂停表变更事务、或者是长事务。



<font style="color:#F5222D;"></font>





## 4. 小结
1. 全局锁主要是用来逻辑备份，有两种方式一种是通过命令 flush tables with read lock;。另一种需要支持库中所有的表支持可重复读事务，使用mysqldump -single-transaction 进行备份。



2. 表锁通过sql语句限制指定表只能进行读、读/写操作,且不允许对其他表操作在解锁前。



3. MDL是自动加上的，不需要通过sql语句。存在读写锁、写写锁互斥情况。锁在事务提交后释放。作用是为了防止DDL和DML操作冲突。

