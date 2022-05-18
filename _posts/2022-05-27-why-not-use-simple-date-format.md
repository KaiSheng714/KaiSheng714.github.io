---
layout: post
title: "為何不要使用 SimpleDateFormat"
meta: java
author: "Kai-Sheng"
permalink: /articles/why-not-use-simple-date-format
categories: [Java]
--- 

SimpleDateFormat is non-thread safe
在低流量環境使用通常不會出錯，但到了高流量的環境可能會出現 parse error



## 解法 1. 每次都 new

## 解法 2. ThreadLocal

## 解法3. 改用  DateTimeFormatter(推薦)


如果是Java8应用，可以使用DateTimeFormatter代替SimpleDateFormat，这是一个线程安全的格式化工具类。就像官方文档中说的，这个类 simple beautiful strong immutable thread-safe。


```java
//解析日期
String dateStr= "2016年10月25日";
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy年MM月dd日");
LocalDate date= LocalDate.parse(dateStr, formatter);

//日期转换为字符串
LocalDateTime now = LocalDateTime.now();
DateTimeFormatter format = DateTimeFormatter.ofPattern("yyyy年MM月dd日 hh:mm a");
String nowStr = now .format(format);
System.out.println(nowStr);
```

### **References**
 https://developer.aliyun.com/article/756625

 https://www.baeldung.com/java-simple-date-format

 https://www.baeldung.com/migrating-to-java-8-date-time-api