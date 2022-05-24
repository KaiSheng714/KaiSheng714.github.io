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

可以看到，在 format() 方法中先將日期存放到一個 Calendar 對象中，而這個 Calender 對象在 SimpleDateFormat 中還是以成員變量存在的。在隨後調用 subFormat() 時會再次用到成員變量 calendar。這就是引發問題的根源。在 parse() 方法中也會存在相應的問題。

試想，在多線程環境下，如果兩個線程都使用同一個 SimpleDateFormat 實例，那麼就有可能存在其中一個線程修改了 calendar 後緊接著另一個線程也修改了 calendar，那麼隨後第一個線程用到 calendar 時已經不是它所期待的值了。


## **解法 1. 每次都 new**

```java
public class DateUtil {
        
    public static String format(Date date) {
        // ...
        SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd hh:mm:ss");
        return simpleDateFormat.format(date);
    }
}
```

這是最簡單的做法了，但也最沒有效率，因為每次都需要 `new SimpleDateFormat`，而有[資料](https://askldjd.wordpress.com/2013/03/04/simpledateformat-is-slow/)表示這一件成本很高的事。但因為資料有點久遠，如果機器效能允許的話，也許可以考慮這個解法，畢竟很簡單。

## **解法 2. ThreadLocal**

ThreadLocal 適合用於處理 non-thread safe object 且不可使用 `synchronized` 的場景
ThreadLocal是執行緒區域性變數，和普通變數的不同在於：每個執行緒持有這個變數的一個副本，可以獨立修改（set方法）和訪問（get方法）這個變數，並且執行緒之間不會發生衝突。

```java
public class DateUtil {
    private static final ThreadLocal<SimpleDateFormat> formatter = new ThreadLocal<SimpleDateFormat>() {
        @Override
        protected SimpleDateFormat initialValue() {
            return new SimpleDateFormat("yyyyMMdd HHmm");
        }
    };

    public String format(Date date) {
        return formatter.get().format(date);
    }
}
```

這個方式可以解決效能的問題，缺點是程式會變得比較複雜、難理解。ThreadLocal 在使用上有較多需要注意的地方，若使用不慎，可能造成更多問題，例如 Memory leak。

## **解法3. 改用  DateTimeFormatter(推薦)**

Java8 提供了 `DateTimeFormatter` 來代替 SimpleDateFormat。就像官方文件中說的:

> DateTimeFormatter in Java 8 is immutable and thread-safe alternative to SimpleDateFormat.

簡單的範例如下:

將字串轉成 LocalDate
```java
String dateStr = "2022/05/24";
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy/MM/dd");
LocalDate date = LocalDate.parse(dateStr, formatter);
```

LocalDateTime 轉成字串
```java
LocalDateTime now = LocalDateTime.now();
DateTimeFormatter format = DateTimeFormatter.ofPattern("yyyy年MM月dd日 hh:mm");
System.out.println(now.format(format));
```

### **References**
- [Migrating to the New Java 8 Date Time API](https://www.baeldung.com/migrating-to-java-8-date-time-api)
- [Why is Java's SimpleDateFormat not thread-safe?](https://stackoverflow.com/questions/6840803/why-is-javas-simpledateformat-not-thread-safe)

https://stackoverflow.com/questions/817856/when-and-how-should-i-use-a-threadlocal-variable