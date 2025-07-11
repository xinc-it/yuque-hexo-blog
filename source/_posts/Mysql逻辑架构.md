---
title: Mysql逻辑架构
date: '2023/1/11 20:48:28'
updated: '2023-01-20 09:44:53'
categories: Mysql
tags:
  - 数据库
  - MySQL
---
# 架构剖析


## 请求流程
![](/images/4fda56bc1ba730b69e66496ec02b6a34.png)

## Mysql架构
![](/images/4bd4d001e4378671baf33b9b2446b0a1.png)

### 1. 连接层
#### 1. Connection pool
作用：

1. 客户端与服务器建立TCP连接
2. 查询用户对应权限，判定用户能够进行哪些操作
3. 控制连接个数和连接的复用

![](/images/36798cab9c612b125609c915fb793425.png)





### 2. 服务层
#### 1. Sql接口
1. 接收SQL命令
2. 返回查询结果



#### 2. 解析器
1. 解析SQL语句
2. 生成解析树
3. 验证用户权限







#### 3. 优化器
1. 生成执行计划
2. 明确索引使用
3. 采用选取-投影-连接进行查询





#### 4. 查询缓存
1. 记录查询结果
2. key-value形式存储





### 引擎层
1. 数据存储提取
2. 维护底层数据执行

