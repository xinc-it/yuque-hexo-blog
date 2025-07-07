---
title: 了解Mysql查询语句执行流程
date: '2022/1/11 20:55:45'
updated: '2023-01-20 22:59:59'
categories: 数据库
tags:
  - 数据库
  - Mysql
  - InnoDB
---
**MySQL逻辑架构图**

![](/images/fbeaf3e36ca91791765c969e44ec143b.png)



## 1. Server层
#### 
Server层包括连接器、分析器、优化器、执行器、查询缓存。



### 1. 连接器


作用：

1. **<font style="color:#F5222D;">负责跟客户端建立连接</font>**
2. **<font style="color:#F5222D;">获取权限</font>**
3. **<font style="color:#F5222D;">维持和管理连接</font>**



连接指令

```sql
mysql -h$ip -P$port -u$user -p
```

获取权限：通过TCP握手后，连接器验证身份，通过输入的用户名和密码。查询MySQL数据库中的user表。查询对应的用户权限。



**长连接和短链接**



长连接：连接成功后，如果客户端持续有请求，则一直使用该连接



短连接：每次执行完几次查询就断开连接，下次查询在重新建立连接。





建议使用长连接，因为建立连接过程比较复杂，尽量减少建立连接的动作。



但是建立长连接也会导致内存消耗增加。因为Mysql在执行时使用的内存是在连接对象里面的。这些资源会在断开时释放。

如果长连接累计下来，可能导致内存占用太大。



解决方法

1. 定期断开长连接
2. 执行mysql_reset_connectionc初始化连接资源。针对5.7以后。









### 2. 查询缓存


**<font style="color:#F5222D;">作用：用于做缓存</font>**

Mysql拿到请求后会先去缓存中，查看是否有对应的查询语句。缓存以key-value的形式存储。

查询语句作为key ,查询结果作为value存入缓存中。

但是不建议使用查询缓存。因为缓存数据失效的很频繁。

**8.0之后MySQL删掉了查询缓存**

****

```sql
//缓存查询

select  SQL_CACHE *  from T ;

// 不查询缓存
select   DEMAND *  from T ; 

```

****

### 3. 分析器


**<font style="color:#F5222D;">作用： 进行sql语句的解析</font>**。



```sql
select * from T where ID=1;
```

1. 词法分析



通过关键字 **select **识别为查询语句。将对应的T识别成对应的表名T，将字符串"ID" 识别成列"ID".



2. 语法分析



判断Sql语句是否满足Mysql语法。如果语句不对，则会受到错误提醒。







### 4. 优化器


**<font style="color:#F5222D;">作用：决定执行方案</font>**

1. **<font style="color:#F5222D;">决定使用哪个索引，多个索引情况下。</font>**
2. **<font style="color:#F5222D;">决定各表连接顺序，多个表关联的情况下。</font>**



```sql
select * from t1 join t2 using(ID) where t1.c=10 and t2.d=20;
```

+ <font style="color:rgb(53, 53, 53);">既可以先从表 t1 里面取出 c=10 的记录的 ID 值，再根据 ID 值关联到表 t2，再判断 t2 里面 d 的值是否等于 20。</font>
+ <font style="color:rgb(53, 53, 53);">也可以先从表 t2 里面取出 d=20 的记录的 ID 值，再根据 ID 值关联到 t1，再判断 t1 里面 c 的值是否等于 10。</font>

<font style="color:rgb(53, 53, 53);"></font>

<font style="color:rgb(53, 53, 53);">优化器会决定执行哪种方案。</font>

<font style="color:rgb(53, 53, 53);"></font>

<font style="color:rgb(53, 53, 53);"></font>

### 5. 执行器


**<font style="color:#F5222D;">作用：执行sql语句</font>**



1. 判断用户对这个表有无查询权限
2. 根据表引擎提供的接口执行查询
+ <font style="color:rgb(53, 53, 53);">调用 InnoDB 引擎接口取这个表的第一行，判断 ID 值是不是 10，如果不是则跳过，如果是则将这行存在结果集中；</font>
+ <font style="color:rgb(53, 53, 53);">调用引擎接口取“下一行”，重复相同的判断逻辑，直到取到这个表的最后一行。</font>
+ <font style="color:rgb(53, 53, 53);">执行器将上述遍历过程中所有满足条件的行组成的记录集作为结果集返回给客户端。</font>

<font style="color:rgb(53, 53, 53);"></font>

<font style="color:rgb(53, 53, 53);"></font>

<font style="color:rgb(53, 53, 53);"></font>

### 6.小结


1. 连接器主要作用让客户端和服务端建立连接、获取登录用户权限、维持和管理连接。
2. 查询缓存用于通过查询的sql语句作为key，判断是否存在对应的key,如果存在直接返回缓存中存储的查询结果。
3. 分析器用于**<font style="color:#F5222D;">判断sql语句语法是否正确</font>**！将查询的sql语句的字段名解析成对应的数据库中对应的字段和表
4. <font style="color:rgb(53, 53, 53);">分析器作用</font>**<font style="color:#F5222D;">优化查询条件</font>**<font style="color:rgb(53, 53, 53);">。查询条件判断使用哪种索引进行查询以及各表的连接顺序</font>
5. <font style="color:rgb(53, 53, 53);">执行器用于</font>**<font style="color:#F5222D;">执行sql语句</font>**<font style="color:rgb(53, 53, 53);">。</font>

