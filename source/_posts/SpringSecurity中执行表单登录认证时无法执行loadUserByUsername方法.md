---
title: SpringSecurity中执行表单登录认证时无法执行loadUserByUsername方法
date: 2021-05-13T00:00:00.000Z
updated: '2025-07-07 16:07:10'
categories: Java
tags:
  - Java
  - SpringSecurity
  - 开发问题
---
# 项目场景：


执行表单登录认证时配置了loginProcessUrl和loginPage。但是执行登录认证时并不执行UserDetailsService接口的loadByUsername方法。导致认证失败。



# 问题描述：


## 1. 表单登录页面


![](/images/e4f073eefc5b95a2dc2b93c4d34e8ea5.png)



## 2. 配置类


![](/images/dc5c87b56d8b3e21b98552bc400a1bc4.png)



## 3. loadUserByUsername方法


![](/images/20bafec4b0763263b81ff4dd71564df3.png)



#### 所有都配置好了，但是进行登录认证的时候还是认证失败跳回登录页。并且控制台未打印loadUserByUsername方法中的日志。


![](/images/7aab12af02050d127d5824fbfb7e01d6.png)  
![](/images/ece5f3211e2df492370e136220ec3a1e.png)



# 原因分析：


因此判断是loginProcessUrl方法的问题。进入loginProcessUrl方法内部发现。关键信息  
![](/images/99dab0d0806c7de104736b19ecc83504.png)  
登录表单申请方式必须为post才行，springsecurity才会进行登录认证。

---

# 解决方案：


## 将登录表单中提交方法更改为post方式即可
