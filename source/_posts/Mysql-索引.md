---
title: Mysql-索引
date: '2022/9/9 15:17:06'
updated: '2023-01-26 22:46:21'
categories: 数据库
tags:
  - Mysql
  - 数据库
  - InnoDB
---
## 1.索引常见模型


### 1. 哈希表




![](/images/1bffd82544a6b798450988802b4daff8.png)



通过hash算法将key换算成确定的位置，然后把value放入到这个数组的位置。如果数组已经存在值，

则通过拉链法，拉出一条链表。





优点：等值查询的情况下查询效率高

缺点：范围查询效率低下



### 2. 有序数组




![](/images/25505b9811e4839a32bd8b793d654343.png)







优点：等值和范围查询效率高

缺点：插入、删除数据效率低下





场景：只适用于静态存储引擎





### 3. 二叉搜索树


![](/images/3dff5e855236507df7b3a936922d946d.png)





缺点：数据量大的情况下，导致树很高，需要进行多次磁盘读取数据，比较浪费时间。









## 2. InnoDB索引模型


B+树

![](/images/a9930fed33f2277261d9840f9768877e.png)







## 3.  主键索引和非主键索引




**InnoDB里面主键索引也被称为聚簇索引**



<font style="background-color:#FADB14;">非主键索引的叶子节点存储的是主键的值</font>



<font style="background-color:#F5222D;">主键索引和非主键索引的区别</font>

1. 主键查询： 只需要查询主键索引的B+树,查询出对应的数据
2. 非主键索引：查询非主键索引的树，获得对应的主键值，然后通过主键值查询主键索引的B+树









## 4. 索引维护


概念： 当数据页中数据存储满了，会生成一个新的数据页，然后原有数据也中的部分数据会移到新的数据也中。这种过程叫做页分裂。



同理页合并是两个数据页中的数据太少了，合并到一个数据页中。











## 5. 索引覆盖


### 1. 回表


<font style="color:rgb(53, 53, 53);"> select * from T where k between 3 and 5</font>

![](/images/4d6f1ba45bbf2d20a8ae67dc2db443e2.png)

执行流程



1. <font style="color:rgb(53, 53, 53);">在 k 索引树上找到 k=3 的记录，取得 ID = 300；</font>
2. <font style="color:rgb(53, 53, 53);">再到 ID 索引树查到 ID=300 对应的 R3；</font>
3. <font style="color:rgb(53, 53, 53);">在 k 索引树取下一个值 k=5，取得 ID=500；</font>
4. <font style="color:rgb(53, 53, 53);">再回到 ID 索引树查到 ID=500 对应的 R4；</font>
5. <font style="color:rgb(53, 53, 53);">在 k 索引树取下一个值 k=6，不满足条件，循环结束</font>





**回表：非主键搜索完后，回到主键索引树进行搜索的过程**。（步骤2、4）







### 2. 索引覆盖
<font style="color:rgb(53, 53, 53);">select ID from T where k between 3 and 5</font>

<font style="color:rgb(53, 53, 53);"></font>

<font style="color:rgb(53, 53, 53);"></font>

<font style="color:rgb(53, 53, 53);">对于上述通过非主键索引查询主键索引信息，由于非主键索引存放了主键的信息。</font>

<font style="color:rgb(53, 53, 53);">导致我们不需要进行回表。</font>**<font style="color:rgb(53, 53, 53);">由于索引k的值覆盖了查询的字段，我们称之为覆盖索引</font>**

**<font style="color:rgb(53, 53, 53);"></font>**

**<font style="color:rgb(53, 53, 53);"></font>**

**<font style="color:rgb(53, 53, 53);">优点：减少树的搜索次数，提升查询效率</font>**

<font style="color:rgb(53, 53, 53);"></font>

<font style="color:rgb(53, 53, 53);"></font>

## 6. 最左前缀原则


```json
CREATE TABLE `tuser` (
  `id` int(11) NOT NULL,
  `id_card` varchar(32) DEFAULT NULL,
  `name` varchar(32) DEFAULT NULL,
  `age` int(11) DEFAULT NULL,
  `ismale` tinyint(1) DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `id_card` (`id_card`),
  KEY `name_age` (`name`,`age`)
) ENGINE=InnoDB
```

![](/images/62098dba32fe88d15b6405212a66abc9.png)







<font style="color:rgb(53, 53, 53);">如果你要查的是所有名字第一个字是“张”的人，你的 SQL 语句的条件是"where name like ‘张 %’"。这时，你也能够用上这个索引，查找到第一个符合条件的记录是 ID3，然后向后遍历，直到不满足条件为止。</font>

**<font style="color:rgb(53, 53, 53);"></font>**

**<font style="color:rgb(53, 53, 53);">联合索引以第一个索引为基准进行排序，然后通过第一个索引的值来进行查询</font>**

[最左前缀原则](https://www.cnblogs.com/ljl150/p/12934071.html)

**<font style="color:rgb(53, 53, 53);"></font>**

**<font style="color:rgb(53, 53, 53);"></font>**

**<font style="color:rgb(53, 53, 53);"></font>**

## 7. 索引下推
Mysql5.6引入索引下推，是在最左前缀的条件小，过滤调不不符合条件的记录减少回表的次数



```json

mysql> select * from tuser where name like '张 %' and age=10 and ismale=1;
```

![](/images/78f91371122c3b1a9de16efcb4193dec.png)





![](/images/992a1791532a870124f83ef0a58ba456.png)





## 8. 小结


1. 索引的常见模型
    1. 哈希表 （不支持范围查询）
    2. 有序数组（插入删除效率低）
    3. 二叉搜索树（数据量大，导致树的高度很高，需要进行多次磁盘读取）
2. InnoDB 索引模型
    1. B+树支持范围查询，树的高度不会太高
    2. 查找数据消耗磁盘读取的时间相同
3. 主键索引和非主键索引
    1. 在InnoDB主键索引存储索引行对应的数据
    2. 非主键索引存储的是主键索引的值，需要通过回表的方式，去查询一遍主键索引
4. 索引覆盖：通过非主键索引查询主键索引的值，导致不需要回到主键索引树再次进行查询的过程。
5. 最左前缀原则

