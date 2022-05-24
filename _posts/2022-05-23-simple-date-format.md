---
layout: post
title: "如何正確使用 SimpleDateFormat"
meta: java
author: "Kai-Sheng"
permalink: /articles/simple-date-format
categories: [Java]
--- 

開發 Java 專案時經常使用時間、日期與字串的互相轉換，最常見簡單的方式是使用 SimpleDateFormat，想必大家對它不陌生。雖然它簡單易用，在低流量環境使用通常不會出錯，但到了高流量、多執行緒的環境就可能會出現異常。本文介紹幾種解決方案。

![why-simple-date-format-is-bad.png](/assets/image/why-simple-date-format-is-bad.png)

我們都知道在程式中應盡量少使用 `new SimpleDateFormat` 因為若頻繁使用，則需要花費較多的成本，因此我們盡可能共用同一個實例。假設有一個轉換日期時間的 `DateUtil` 程式碼如下
 
```java
public class DateUtil {

    private static final SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd hh:mm:ss");
        
    public static String format(Date date) {
        // ...
        return simpleDateFormat.format(date);
    }
}
```
不幸的是，這就是最典型的錯誤用法。

官方文件提到:

> Date formats are not synchronized. It is recommended to create separate format instances for each thread. If multiple threads access a format concurrently, it must be synchronized externally.

從 SimpleDateFormat 的[原始碼](https://developer.classpath.org/doc/java/text/SimpleDateFormat-source.html)中也可以看到，calendar 被宣告為成員變數，因此呼叫 `format`, `parse` 等 method 時會多次存取此 calendar。在高流量、多線程環境下，將會造成 race condition，結果值就會不符預期，甚至拋出 exception。幸運的是，有許多解決方法可以克服 : 


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

這是最簡單的做法，只要每次都宣告區域變數就可以了，區域變數一直都是 thread-safe。但有[資料](https://askldjd.wordpress.com/2013/03/04/simpledateformat-is-slow/)表示，一直`new SimpleDateFormat` 是成本很高的事。但因為資料有點久遠，若機器效能允許的話，也許可以考慮這個解法，畢竟簡單的作法往往是較好的。

## **解法 2. ThreadLocal**
ThreadLocal 最典型的用法就是處理 non-thread safe object，且無法使用 `synchronized` 的情況，SimpleDateFormat 正好是最常見的例子。ThreadLocal 為每個執行緒建立一個 SimpleDateFormat 的副本，每個執行緒可以獨立 `set`, `get`, `remove` 這個變數，並且執行緒之間不會發生衝突，自然而然解決了 race condition 的問題。程式碼如下:

```java
public class DateUtil {
    private static final ThreadLocal<SimpleDateFormat> formatter = new ThreadLocal<SimpleDateFormat>() {
        @Override
        protected SimpleDateFormat initialValue() {
            return new SimpleDateFormat("yyyyMMdd HHmm");
        }
    };

    public static String format(Date date) {
        return formatter.get().format(date);
    }
}
```

此方法也解決方法1.效能的問題。缺點是程式會變得比較複雜、難理解，而且 ThreadLocal 在使用上有較多需要注意的地方，若使用不慎，可能造成更多問題，例如 memory leak。

## **解法3. 改用 DateTimeFormatter(推薦)**

畢竟這個問題困擾很多人許久了，因此 Java 8 版本後官方就提供了 `DateTimeFormatter` 用來代替 `SimpleDateFormat`。就像官方文件中說的:

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
- [When and how should I use a ThreadLocal variable?](https://stackoverflow.com/questions/817856/when-and-how-should-i-use-a-threadlocal-variable)