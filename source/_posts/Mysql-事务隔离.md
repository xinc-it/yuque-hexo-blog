---
title: Mysql-事务隔离
date: '2022/9/7 19:17:06'
updated: '2023-01-14 16:02:16'
categories: 数据库
tags:
  - Mysql
  - 数据库
  - InnoDB
---
## 1. 隔离性和隔离级别


### 1. 事务特性
1. **<font style="color:#F5222D;">原子性</font>**
2. **<font style="color:#F5222D;">持久性</font>**
3. **<font style="color:#F5222D;">隔离性</font>**
4. **<font style="color:#F5222D;">一致性</font>**





### 2. 事务隔离级别
1. **<font style="color:#F5222D;">读已提交（事务未提交时，变更能被其它事务看到）</font>**
2. **<font style="color:#F5222D;">读未提交（事务提交后，其它事务才能看到变更）</font>**
3. **<font style="color:#F5222D;">可重复读（一个事务执行过程中，总是跟启动时看到的数据是一致的。未提交的变更对其它事务不可见）</font>**
4. **<font style="color:#F5222D;">串行化（同一行记录写会加写锁，读会读锁。当出现读写锁冲突时，后访问的事务，必须等前一个事务完成，才能执行）</font>**

**<font style="color:#F5222D;"></font>**

![](/images/a25a301342ce1c6b8bd640338e5846a5.png)



 v1=v2=1,v3=2

四种事务得到的结果

1. 读未提交： v1、v2、v3=2
2. 读已提交: v1=1 ，v2、v3=2
3. 可重复读：v1=v2=1，v3=2
4. 可串行化 ： v1=v2=1,v3=2

**<font style="color:#F5222D;"></font>**



## 2. 事务隔离的实现


当一个值从1按顺序变成2，3，3那么他的日志里面就会有类似的回滚段记录

![](/images/53a97205821d25c178b3c6316ae8a1c4.png)

如图事务A、B、C修改记录导致一个字段对应多个值。



通过MVCC 来实现一个数据的多个版本，通过undo log 实现数据版本的回滚

通过MVCC和undo log 来实现事务的隔离

具体可以参考



[Mysql实战45讲](https://www.yuque.com/sanxingalaxys9/gwxct1/lq01qk)



当回滚段日志过长时，系统会自动清除。







## 3. 事务的启动方式




1. set autocommit=0关闭自动提交，只要执行 select 语句自动开启事务
2. begin 或是start transaction



## 4. 小结
1. 四大特性和隔离级别
2. 事务隔离的实现（通过MVCC和 undo log)
3. 事务的启动方式
    1. 自动启动 select 语句执行时自动启动
    2. 手动启动 begin、 start transaction

