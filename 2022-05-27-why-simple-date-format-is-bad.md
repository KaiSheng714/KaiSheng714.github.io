---
layout: post
title: "為何不要使用 SimpleDateFormat"
meta: java
author: "Kai-Sheng"
permalink: /articles/why-simple-date-format-is-bad
categories: [Java]
--- 

開發專案經常會將時間、日期轉換成字串，Java 中最常見簡單的方式是 SimpleDateFormat。雖然它簡單易用，在低流量環境使用通常不會出錯，但到了高流量的環境可能會出現異常。本文介紹幾種解決方案。

---

![why-simple-date-format-is-bad.png](/assets/image/why-simple-date-format-is-bad.png)


有一個轉換日期、時間的 DateUtil，程式碼如下
 
```java
public class DateUtil {

    private static final SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd hh:mm:ss");
        
    public static String format(Date date) {
        // ...
        return simpleDateFormat.format(date);
    }
}
```

問題就在 `static` final SimpleDateFormat


> Date formats are not synchronized. It is recommended to create separate format instances for each thread. If multiple threads access a format concurrently, it must be synchronized externally.



## 解法 1. 每次都 new


```java
public class DateUtil {
        
    public static String format(Date date) {
        // ...
        SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd hh:mm:ss");
        return simpleDateFormat.format(date);
    }
}
```

比較花費效能，因為每次都需要 `new SimpleDateFormat`，

## 解法 2. ThreadLocal
比較複雜
## 解法3. 改用  DateTimeFormatter(推薦)
DateTimeFormatter in Java 8 is immutable and thread-safe alternative to SimpleDateFormat.


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
 

 https://www.baeldung.com/java-simple-date-format

 - [Migrating to the New Java 8 Date Time API](https://www.baeldung.com/migrating-to-java-8-date-time-api)

 - [Why is Java's SimpleDateFormat not thread-safe?](https://stackoverflow.com/questions/6840803/why-is-javas-simpledateformat-not-thread-safe)