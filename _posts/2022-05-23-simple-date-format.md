---
layout: post
title: "SimpleDateFormat 的錯誤用法"
meta: java
author: "Kai-Sheng"
permalink: /articles/simple-date-format
categories: [Java]
image: /assets/image/simple-date-format.png
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

從 SimpleDateFormat 的[原始碼](https://developer.classpath.org/doc/java/text/SimpleDateFormat-source.html)中也可以看到它是有狀態的，而且其中 calendar 被宣告為成員變數，因此呼叫 `format`, `parse` 等 method 時會多次存取此 calendar。在高併發環境下，將會造成 race condition，結果值就會不符預期，甚至拋出 exception。

幸運的是，已有許多解決方案: 

## **正確用法 1. 每次都 new**

```java
public class DateUtil {

    public static String format(Date date) {
        SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd hh:mm:ss");
        return simpleDateFormat.format(date);
    }
}
```

這是最簡單的做法，只要每次都宣告區域變數就可以了，區域變數是 thread-safe。若專案對於效能要求不高，也許可以考慮這個解法，或直到出現效能問題時再考慮其他方法。畢竟至少這個做法能正確運作，而且簡單的作法往往是較好的。

## **正確用法 2. 使用 synchronized**
```java
public class DateUtil {
    private static SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd hh:mm:ss");

    synchronized public static String format(Date date) {
        return simpleDateFormat.format(date);
    }
}
```

首先宣告 SimpleDateFormat 成員變數，避免重複 new 而造成效能問題。再加上關鍵字 `synchronized` 就能確保同一時刻只有一個 thread 能執行 `format` (mutual exclusion)。雖然這個方式不會出錯，但在高併發場景下使用也有可能造成效能不佳的問題。 

## **正確用法 3. 使用 ThreadLocal 容器**
ThreadLocal 容器是一種讓程式達到 thread-safety 的手段，它相當於給每個 thread 都開了一個獨立的存儲空間，既然 thread 之間互相隔離，自然解決了 race condition 的問題，也讓 thread 能重複使用 SimpleDateFormat 實例。程式碼如下:

```java
public class DateUtil {
    // 可以把 ThreadLocal<SimpleDateFormat> 視為一個全域 Map<Thread, SimpleDateFormat>，key 就是 current thread
    // 意義上相當於 currentThread 專屬、獨立的 cache。
    private static ThreadLocal<SimpleDateFormat> local = new ThreadLocal<>();

    private static SimpleDateFormat getDateFormat() {
        // currentThread 從自己的 ThreadLocalMap 取得 SimpleDateFormat。
        // 如果是 null，則建立 SimpleDateFormat 並放入自己的 ThreadLocalMap 中。
        SimpleDateFormat dateFormat = local.get();
        if (dateFormat == null) {
            dateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
            local.set(dateFormat);
        }
        return dateFormat;
    }

    public static String format(Date date) {
        return getDateFormat().format(date);
    }
}
```

舉例來說，如果 thread pool 有 10 個 thread，程式就會建立 10 個 SimpleDateFormat 實例，這些 thread 們在每次的任務中重複使用各自的 SimpleDateFormat。但要注意一點，該 thread 能夠重複被使用(例如 server 在處理完一次 request 後，thread 會再回到 thread pool 待命)，否則效果會和方法1差不多。這個方法的缺點是程式會變得較複雜。

## **正確用法4. 改用 DateTimeFormatter(推薦)**

雖然有點文不對題，畢竟這個問題困擾很多人許久了，因此在 Java 8 版本後官方就提供了 `DateTimeFormatter` 物件用來代替 `SimpleDateFormat`。就像官方文件中說的:

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

## **References**
- [Migrating to the New Java 8 Date Time API](https://www.baeldung.com/migrating-to-java-8-date-time-api)
- [Why is Java's SimpleDateFormat not thread-safe?](https://stackoverflow.com/questions/6840803/why-is-javas-simpledateformat-not-thread-safe)
- [When and how should I use a ThreadLocal variable?](https://stackoverflow.com/questions/817856/when-and-how-should-i-use-a-threadlocal-variable)

## **更多你可能會感興趣的文章**
- [如何寫出優秀的單元測試 (Best Practice)](/articles/good-unit-test)
- [常見的 Interface 錯誤用法](/articles/anti-pattern-of-java-interface-impl-style)