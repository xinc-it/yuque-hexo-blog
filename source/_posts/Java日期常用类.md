---
title: Java日期常用类
date: '2021/12/24 20:48:28'
updated: '2023-01-14 16:06:14'
categories: Java
tags:
  - Java
---
# SimpleDateFormate类（日期格式化类）
Date类大部分被废弃（不利于国际化），用日期格式化类来格式化日期

作用：格式化日期

```java
/**
* 1. SimpleDateFormat的两种操作
* 格式化：日期---》字符串
* 解析：字符串---》日期
*/

/**
*2. SimpleDateFormat的实例化
* 常用带格式的实例化，进行初始化
*/
SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd hh:mm:ss");
/**
* 3. 日期>>>字符串
*/
String format = dateFormat.format(new Date());
System.out.println(format);
/**
* 4. 字符串》》》日期
*/
Date parse = dateFormat.parse("2019-09-01 12:22:23");
System.out.println(parse);
```



## 日期常用格式化模式：
| 格式 | 结果 |
| --- | --- |
| "yyyy.MM.dd G 'at' HH:mm:ss z" | 2001.07.04 AD at 12:08:56 PDT |
| "EEE, MMM d, ''yy" | Wed, Jul 4, '01 |
| "h:mm a" | 12:08 PM |
| "hh 'o''clock' a, zzzz" | 12 o'clock PM, Pacific Daylight Time |
| "K:mm a, z" | 0:08 PM, PDT |
| "yyyyy.MMMMM.dd GGG hh:mm aaa" | 02001.July.04 AD 12:08 PM |
| "EEE, d MMM yyyy HH:mm:ss Z" | Wed, 4 Jul 2001 12:08:56 -0700 |
| "yyMMddHHmmssZ" | 010704120856-0700 |
| "yyyy-MM-dd'T'HH:mm:ss.SSSZ" | 2001-07-04T12:08:56.235-0700 |
| "yyyy-MM-dd'T'HH:mm:ss.SSSXXX" | 2001-07-04T12:08:56.235-07:00 |
| "YYYY-'W'ww-u" | 2001-W27-3 |




![](/images/fd267fc1a94782f623f269fbae4df941.png)





# 两个Date类的使用
三天打鱼两天晒网练习题

从1990-01-01到任意日期需要打多少天鱼和晒多少天网？

```java
 public void countDays() throws ParseException {
        String dateStr1 = "1990-01-01";
        String dateStr2 = "1990-01-07";
        SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd");
        Date date1 = dateFormat.parse(dateStr1);
        Date date2 = dateFormat.parse(dateStr2);
        long dayNums = (date2.getTime() - date1.getTime()) / (1000 * 60 * 60 * 24);
        long times = dayNums / 5;
        long leftDays = dayNums % 5;
        long fishDays = times * 3;
        long netDays = (times * 2);
        if (leftDays > 3) {
            fishDays += 3;
            netDays += leftDays - 3;
        } else {
            fishDays += leftDays;
        }
        System.out.println("打鱼总共："+fishDays+"  晒网："+netDays);
    }
```









## JDK的Date类和Calendar类的问题
可变性：日期和时间的类应该不可变

偏移性：Date中的年份从1990开始计算，月份从0开始

格式化:Calendar不能格式化

线程不安全











# JDK8的新增日期API
## LocalDate、LocalTIme、LocalDateTime类
### 常用API
![](/images/4c608c8c1e9349070ced1b138b2ff5b1.png)



```java
/**
* Local时间、日期类的两种实例化方式
* now（）获取当前日期、时间
*/
LocalDate localDate = LocalDate.now();
LocalDateTime localDateTime = LocalDateTime.now();
LocalTime localTime = LocalTime.now();

System.out.println(localDate);
System.out.println(localDateTime);
System.out.println(localTime);
System.out.println("-------------------");
/**
* of()获取指定日期、时间
*/
LocalDate localDate1 = LocalDate.of(2000, 1, 1);
LocalTime localTime1 = LocalTime.of(23, 59);
LocalDateTime localDateTime1 = LocalDateTime.of(localDate1, localTime1);
System.out.println(localDate1);
System.out.println(localTime1);
System.out.println(localDateTime1);
```



## Instant类
**记录从1970年到指定时间经历的毫秒数 **

### 常用API
![](/images/5cafa0b25e51c06a8fcda39df74267ed.png)

```java
/**
* Instant类的使用
*/
Instant instant = Instant.now();
//获取本初子午线的时间
System.out.println(instant);
//偏移指定时间
OffsetDateTime offsetDateTime = instant.atOffset(ZoneOffset.ofHours(8));
System.out.println(offsetDateTime);
```





## DateTimeFormatter日期时间格式化类


### 常用API
![](/images/b11874b402b816dd40be9ace2c838a82.png)



```java
// 方式一：预定义的标准格式。如：ISO_LOCAL_DATE_TIME;ISO_LOCAL_DATE;ISO_LOCAL_TIME
DateTimeFormatter formatter = DateTimeFormatter.ISO_LOCAL_DATE_TIME;
//格式化:日期-->字符串
LocalDateTime localDateTime = LocalDateTime.now();
String str1 = formatter.format(localDateTime);
System.out.println(localDateTime);
System.out.println(str1);//2019-02-18T15:42:18.797

//解析：字符串 -->日期
TemporalAccessor parse = formatter.parse("2019-02-18T15:42:18.797");
System.out.println(parse);

//        方式二：
//        本地化相关的格式。如：ofLocalizedDateTime()
//        FormatStyle.LONG / FormatStyle.MEDIUM / FormatStyle.SHORT :适用于LocalDateTime
DateTimeFormatter formatter1 = DateTimeFormatter.ofLocalizedDateTime(FormatStyle.LONG);
//格式化
String str2 = formatter1.format(localDateTime);
System.out.println(str2);//2019年2月18日 下午03时47分16秒


//      本地化相关的格式。如：ofLocalizedDate()
//      FormatStyle.FULL / FormatStyle.LONG / FormatStyle.MEDIUM / FormatStyle.SHORT : 适用于LocalDate
DateTimeFormatter formatter2 = DateTimeFormatter.ofLocalizedDate(FormatStyle.MEDIUM);
//格式化
String str3 = formatter2.format(LocalDate.now());
System.out.println(str3);//2019-2-18


//       重点： 方式三：自定义的格式。如：ofPattern(“yyyy-MM-dd hh:mm:ss”)
DateTimeFormatter formatter3 = DateTimeFormatter.ofPattern("yyyy-MM-dd hh:mm:ss");
//格式化
String str4 = formatter3.format(LocalDateTime.now());
System.out.println(str4);//2019-02-18 03:52:09

//解析
TemporalAccessor accessor = formatter3.parse("2019-02-18 03:52:09");
System.out.println(accessor);

```







## 日期API
![](/images/33e190b4398b37e0fc52469ad4b20bf2.png)







日期时间类转换

![](/images/c8539a93c279b7d946972765ebb7c55f.png)

