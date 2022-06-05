---
layout: post
title: "SimpleDateFormat 的錯誤用法"
meta: java
author: "Kai-Sheng"
permalink: /articles/simple-date-format
categories: [Java]
--- 

開發 Java 專案時經常操作時間、日期與字串的互相轉換，最常見簡單的方式是使用 SimpleDateFormat，想必大家對它不陌生。雖然它簡單易用，如果沒有正確使用，在一般環境下使用通常不會出錯，但在高併發（High Concurrency）的環境下就可能會出現異常。

![why-simple-date-format-is-bad.png](/assets/image/simple-date-format.png?size=full)

我們都知道在程式中應盡量少使用 `new SimpleDateFormat`，因為若頻繁實例化，則需要花費較多的成本，因此我們盡可能共用同一個實例。假設有一個轉換日期時間的 `DateUtil` 程式碼如下
 
```java
public class DateUtil {

    private static final SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd hh:mm:ss");
        
    public static String format(Date date) {
        return simpleDateFormat.format(date);
    }
}
```
不幸的是，`共用 SimpleDateFormat` 就是最典型的錯誤用法。官方文件提到:

> Date formats are not synchronized. It is recommended to create separate format instances for each thread. If multiple threads access a format concurrently, it must be synchronized externally.

從 SimpleDateFormat 的[原始碼](https://developer.classpath.org/doc/java/text/SimpleDateFormat-source.html)中也可以看到，calendar 被宣告為成員變數，因此呼叫 `format`, `parse` 等 method 時會多次存取此 calendar。在高併發環境下，將會造成 race condition，結果值就會不符預期，甚至拋出 exception。

幸運的是，已有許多解決方案: 

### **正確用法 1. 每次都 new**

```java
public class DateUtil {

    public static String format(Date date) {
        SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd hh:mm:ss");
        return simpleDateFormat.format(date);
    }
}
```

這是最簡單的做法，只要每次都宣告區域變數就可以了，區域變數一直都是 thread-safe。若專案對於效能要求不高，也許可以考慮這個解法，畢竟至少這個做法能正確運作，而且簡單的作法往往是較好的。


### **正確用法 2. 使用 ThreadLocal 容器**
ThreadLocal 容器是一種讓程式達到 thread-safety 的手段，ThreadLocal 顧名思義就是專屬於該 thread 的區域變數，因此其他 thread 是無法存取的，這個容器除了可以保證 thread-safety 外，也可以讓開法者避免使用 `synchronized`，進而影響效能。SimpleDateFormat 正好是最常見的例子，若將 SimpleDateFormat 放入 ThreadLocal 中，自然而然解決了 race condition 的問題，也讓 thread 能重複使用 SimpleDateFormat。程式碼如下:


```java
public class DateUtil {

    private static final ThreadLocal<SimpleDateFormat> formatter = new ThreadLocal<SimpleDateFormat>() {
        @Override
        protected SimpleDateFormat initialValue() {
            return new SimpleDateFormat("yyyy-MM-dd hh:mm:ss");
        }
    };

    public static String format(Date date) {
        return formatter.get().format(date);
    }
}
```

這段程式碼將 SimpleDateFormat 儲存在 ThreadLocal 中，讓該 thread 重複使用 SimpleDateFormat 實例，而不必如同方法1般每次都執行 `new SimpleDateFormat`，
缺點是程式會變得比較複雜一些。但要注意的是，前提是該 thread 能夠重複被使用(例如 server 在處理完一次 request 後，thread 會再回到 thread pool 待命)，而不是用完後就被銷毀。

### **正確用法3. 改用 DateTimeFormatter(推薦)**

雖然有點文不對題，畢竟這個問題困擾很多人許久了，因此 Java 8 版本後官方就提供了 `DateTimeFormatter` 物件用來代替 `SimpleDateFormat`。就像官方文件中說的:

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
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy年MM月dd日 hh:mm");
System.out.println(now.format(formatter));
```

我覺得其中一種不錯的 use case 是爬蟲程式。利用`Jsoup`, `LocalDate` 與該網站的 queryString 去抓取某一個日期區間的資料:
```java
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy/MM/dd");
for (LocalDate date = LocalDate.of(2022, 1, 1); date.isBefore(LocalDate.of(2022, 5, 20)); date = date.plusDays(1)) {
    String url = "https://www.demo.test?date=" + date.format(formatter);
    Document doc = Jsoup.connect(url).get();
    // do something
}
```

### **References**
- [Migrating to the New Java 8 Date Time API](https://www.baeldung.com/migrating-to-java-8-date-time-api)
- [Why is Java's SimpleDateFormat not thread-safe?](https://stackoverflow.com/questions/6840803/why-is-javas-simpledateformat-not-thread-safe)
- [When and how should I use a ThreadLocal variable?](https://stackoverflow.com/questions/817856/when-and-how-should-i-use-a-threadlocal-variable)