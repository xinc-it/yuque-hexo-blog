---
title: LeetCode-汉明距离
date: '2021/7/23 9:42:11'
updated: '2025-07-07 15:58:52'
categories: 算法
tags:
  - 算法
  - LeetCode
---
# 题目描述


![](/images/8939266a575d3dcd66563a105764ccce.png)



# 一、解题思路


这里有两种解题思路



## 1. 位运算法


**位运算**如果你不了解的话，可以看下第二种解题思路。但是整体代码有点冗余，而且效率也不高。  
`^`的运算规则二进制位中各个位上数就是相同为0，不同为1。  
`&`的运算规则是都为1时结果为1，反之全为0。  
`|`的运算规则是只要有1结果为1，反之为0。  
`>>`当前数二级制向右移一位，同时最高位根据数的正负来补0或1  
`<<` 当前数二进制向左移一位，同时最低位补0。



**位运算法**



1.  将两个数`^`运算 
2.  判断运算结果中1的个数。即为两个数二进制中各个位的值不同的个数  
这里有两种计算1个数的方法  
2.1.1  将运算结果按2求余，如果为1就个数加1。反之就忽略  
2.1.2 将运算结果右移一位。并重复上述步骤32次。  
2.2.1 将运算结果和运算结果-1进行`&`运算，然后将值返回作为新的运算结果。1的个数加1。  
2.2.2 直到运算结果为0时跳出循环 



## 2. 暴力破解法


1. 将上述两个数将其二进制位分别放入到不同的List容器中
2. 判断两个容器的长度，将长度较大的容器作为遍历次数
3. 获取两个容器中当前索引的值，并判断是否相等，如果不等距离加1
4. 如果较小容器遍历完了的话，就判断较大容器中的值是否等于0。如果不等距离加1
5. 最后得到的距离即为结果



# 二、代码


## 1.位运算


```java
   public int hammingDistance(int x, int y) {
        int res=x^y;
        int times=0;
      while(res!=0){
          res=res&(res-1);
          times++;
      }
      return times;
    }
```



## 2.暴力破解


```java
 public int hammingDistance(int x, int y) {
         int i;
        ArrayList<Integer> xArray=new ArrayList<>();
        do{
        xArray.add(x%2);
        x=x/2;
      }while(x!=0);

        ArrayList<Integer> yArray=new ArrayList<>();
       
      do{
        yArray.add(y%2);
        y=y/2;
      }while(y!=0);
  
   

         int distance=0;
        //当y的二进制更长时
        if(yArray.size()>=xArray.size()){
             for (int j=0;j<yArray.size();j++){
                 if(j<xArray.size()){
                    if(yArray.get(j)!=xArray.get(j)){
                        distance++;
                    } 
                 }else{
                     if(yArray.get(j)!=0){
                         distance++;
                     }
                 }
             }
        }else{//x的二进制更长
             for (int j=0;j<xArray.size();j++){
                if(j<yArray.size()){
                    if(yArray.get(j)!=xArray.get(j)){
                        distance++;
                    } 
                 }else{
                     if(xArray.get(j)!=0){
                         distance++;
                     }
                 }
             }
        }
       
        return distance;
    }
```



# 总结


我一开始使用的暴力破解法。但觉得暴力破解法太麻烦了。所以就去看了官方的解题教程，发现位运算这一方法更为简单和高效。

