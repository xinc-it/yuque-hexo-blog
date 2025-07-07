---
title: Java8新特性
date: '2022/12/22 20:48:28'
updated: '2023-01-15 11:35:30'
categories: Java
tags:
  - Java
---
# Lambda表达式
本质:**作为函数式接口的实例**

## 格式
->:lambda操作符()

->右边:lambda形参列表(抽象方法的形参列表)

->左边:lambda体(抽象方法的方法体)



## 使用格式
lambda表达式六种使用情况：

1. 形参类型可以省略，如果参数只有一个，可省小括号
2. 方法体只有一条语句，可省略大括号（语句为return语句时，可省略return和大括号关键字）

```java
//语法格式一：无参、无返回值
Runnable runnable = () -> System.out.println("123");
//语法格式二：有一个参数，无返回值
Consumer<String> s1 = (String s) -> {
	System.out.println(s);
};
s1.accept("123");

//语法格式三：数据类型可以省略，编译器可以推断得出（类型推断）

Consumer<String> s2 = (s) -> {
	System.out.println(s);
};
s2.accept("23");


//语法格式四：只需要一个参数，小括号可以省略
Consumer<String> s3 = s -> {
	System.out.println(s);
};
s3.accept("34");


//语法格式五：多个参数，多个执行语句有返回值
Comparator<Integer> comparator = (o1, o2) -> {
	System.out.println(o1);
	System.out.println(o2);
	return o1.compareTo(o2);
};
comparator.compare(12, 23);


//语法格式六：只有一条语句，有return时可省略return和大括号
Comparator<Integer> comparator2 = (o1, o2) ->
	o1.compareTo(o2);
comparator2.compare(12, 11);
```





# 函数式接口
常见函数式接口

![](/images/fd47625d49b542f9e816dfc85285d1c1.png)

![](/images/8b91970182618ebfc78579e85feb8a0d.png)









# 方法引用和构造器引用




方法引用的三种情况

![](/images/f002d21b22597cfc080669fba4811b11.png)



引用要求：  
抽象方法和方法体内部的调用方法

1. 返回值相同
2. 形参列表相同



![](/images/0cbde35e14660ba78d3839db81310ce4.png)

![](/images/e13b4fb7b9cdf49099a372005aa35fd2.png)

![](/images/5e3bdf1bdd4eb0e52e70afe115b172f8.png)

![](/images/8609619444ee9e88ab67f898468506c4.png)







# Stream API


用于操作数据源（集合、数组）

## 特点
1. 不存储元素
2. 不该元数据
3. 操作延迟执行

## Stream操作三步
1. 创建Stream
2. 中间操作
3. 终止操作

![](/images/481fdd446beb5af422feb313c9a251a8.png)







