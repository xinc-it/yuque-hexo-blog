---
title: String及其工具类
date: '2021/12/24 20:48:28'
updated: '2023-01-14 16:06:18'
categories: Java
tags:
  - Java
---
# String 类
## String字符串特性
1.  final类，不可被继承
2. 实现Serializable接口，支持序列化
3.  实现Comparable接口，可比较大小
4.  内部使用final char数组存储字符串数据，解释类String不可更改原因
5. String 具有不变性,体现:
    1. 字面量赋值，字符串值存储在字符串常量池中（常量池不会存储相同内容字符串）
    2. 对字符串进行修改（replace）或者是拼接（StringBuilder、StringBuffer、直接+）字符串时，不会修改原值，而是新建一个值，在新值上修改或拼接



```java
public static void main(String[] args) {
        //字符串定义方式
        String s1="abc";
        String s2="abc";
        System.out.println(s1==s2);
        System.out.println("________________");
        //字符串拼接不会修改原字符串值，会新建一个原值并修改
        String s3="abc";
        s3+="hello";
        System.out.println(s3==s1);
        System.out.println("________________");
        //字符串修改同拼接不会修改原值，会新建一个原值并修改
        String s4="abc";
        s4.replace("a","m");
        System.out.println(s4);
        System.out.println(s4==s1);
        System.out.println("________________");

    }
```





## String字面量赋值和构造器赋值区别 
****

使用构造器的变量指向的地址是堆中的地址，堆再指向abc

字面量变量地址指向的是abc

**字符串常量池中不会存在两个相同的字符串**

![](/images/f275dc441402b35ab1b43d61632eda67.png)







## String和基本数据类型、包装类之间转换
### String 转包装类、基本数据类型
调用对应包装类的parseXxx（Str）方法



### 包装类、基本数据类型转String
调用String类的valueOf();





## 面试题：
### 构造器创建的字符串在内存中创建了几个对象？
最多两个，如果字符串常量池中存在该字符串，则只在堆中创建一个空间，存放字符串常量池中对应字符串的地址！！！





### String陷阱


<font style="color:rgb(0,0,0);">String s1 = "a"; </font>

<font style="color:rgb(0,0,0);">说明：在字符串常量池中创建了一个字面量为</font><font style="color:rgb(0,0,0);">"a"</font><font style="color:rgb(0,0,0);">的字符串。 </font>

<font style="color:rgb(0,0,0);">s1 = s1 + "b"; </font>

<font style="color:rgb(0,0,0);">说明：实际上原来的</font><font style="color:rgb(0,0,0);">“a”</font><font style="color:rgb(0,0,0);">字符串对象已经丢弃了，现在在堆空间中产生了一个字符 </font>

<font style="color:rgb(0,0,0);">串</font><font style="color:rgb(0,0,0);">s1+"b"</font><font style="color:rgb(0,0,0);">（也就是</font><font style="color:rgb(0,0,0);">"ab")</font><font style="color:rgb(0,0,0);">。如果多次执行这些改变串内容的操作，会导致大量副本 </font>

<font style="color:rgb(0,0,0);">字符串对象存留在内存中，降低效率。如果这样的操作放到循环中，会极大影响 </font>

<font style="color:rgb(0,0,0);">程序的性能。 </font>

<font style="color:rgb(0,0,0);">String s2 = "ab"; </font>

<font style="color:rgb(0,0,0);">说明：直接在字符串常量池中创建一个字面量为"ab"的字符串。  String s3 = "a" + "b"; </font>

<font style="color:rgb(0,0,0);">说明：</font><font style="color:rgb(0,0,0);">s3</font><font style="color:rgb(0,0,0);">指向字符串常量池中已经创建的</font><font style="color:rgb(0,0,0);">"ab"</font><font style="color:rgb(0,0,0);">的字符串。 </font>

<font style="color:rgb(0,0,0);">String s4 = s1.intern(); </font>

<font style="color:rgb(0,0,0);">说明：堆空间的</font><font style="color:rgb(0,0,0);">s1</font><font style="color:rgb(0,0,0);">对象在调用</font><font style="color:rgb(0,0,0);">intern()</font><font style="color:rgb(0,0,0);">之后，会将常量池中已经存在的</font><font style="color:rgb(0,0,0);">"ab"</font><font style="color:rgb(0,0,0);">字符串 </font>

<font style="color:rgb(0,0,0);">赋值给s4。</font>









```java
 String  s9="hello";
        String  s10="world";
        String  s11="hello"+"world";

        String  s12=s9+"world";
        String  s13="hello"+s10;
        String  s14=s9+s10;
        String  s15="helloworld";
        //true
        System.out.println(s15==s11);
        //false
        System.out.println(s12==s11);
        //false
        System.out.println(s13==s11);
        //false
        System.out.println(s14==s11);
        //false
        System.out.println(s12==s14);
   		String s16=s14.intern();
        System.out.println(s11==s16);
```

总结：

1. 常量与常量的拼接修改结果存在常量池中，且**不会存在相同内容的常量**
2. 只要是变量的拼接或修改，地址为堆中的地址。
3. 调用intern返回的是常量池中存在的地址









# StringBuffer和StringBuilder类


## 三者异同点
  
String：不可变字符串序列  
StringBuffer：线程安全，效率低，可变字符串序列  
StringBuilder：线程不安全，效率高，可变字符串序列 jdk5新增  
原因：append方法是否被synchronized修饰

## 源码分析
  
StringBuffer  sb1=new StringBuffer();//底层创建一个长度为16的char数组  
sb1.append('a');//value[0]=a;  
sb1.length()//等于数组实际存储的字符数，不等于数组长度



## 扩容流程
1. 调用append方法，
2. 调用父类的append方法
3. 调用ensureCapacityInternal，确保数组长度够用
4. 如果不够用进行数组长度*2+2，再调用arraysCopy的方法将原有数组复制到新数组中

![](/images/c73d174862a751fd5d913a3202cb2b7e.png)











