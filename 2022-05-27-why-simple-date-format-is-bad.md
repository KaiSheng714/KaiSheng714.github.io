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

這是最簡單的做法了，但也最沒有效率，因為每次都需要 `new SimpleDateFormat`，而有[資料](https://askldjd.wordpress.com/2013/03/04/simpledateformat-is-slow/)表示這一件成本很高的事。

## 解法 2. ThreadLocal
```java
static ThreadLocal<SimpleDateFormat> formatter = new ThreadLocal<SimpleDateFormat>() {
    @Override
    protected SimpleDateFormat initialValue() {
        return new SimpleDateFormat("yyyy-MM-dd");
    }
};

public String formatDate(Date date) {
    return formatter.get().format(date);
}
```

可以解決效能的問題，缺點是程式會變得比較複雜、難理解

## 解法3. 改用  DateTimeFormatter(推薦)

Java8 提供了 `DateTimeFormatter` 來代替 SimpleDateFormat。就像官方文件中說的:

> DateTimeFormatter in Java 8 is immutable and thread-safe alternative to SimpleDateFormat.

簡單的範例如下

```java
String dateStr = "2022/05/24";
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy/MM/dd");
LocalDate date = LocalDate.parse(dateStr, formatter);
```

日期轉成字串
```java
LocalDateTime now = LocalDateTime.now();
DateTimeFormatter format = DateTimeFormatter.ofPattern("yyyy年MM月dd日 hh:mm");
System.out.println(now.format(format));
```
```

### **References**
 

 https://www.baeldung.com/java-simple-date-format

 - [Migrating to the New Java 8 Date Time API](https://www.baeldung.com/migrating-to-java-8-date-time-api)

 - [Why is Java's SimpleDateFormat not thread-safe?](https://stackoverflow.com/questions/6840803/why-is-javas-simpledateformat-not-thread-safe)