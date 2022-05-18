---
layout: post
title: "為何不要使用 SimpleDateFormat"
meta: java
author: "Kai-Sheng"
permalink: /articles/why-simple-date-format-is-bad
categories: [Java]
--- 

開發專案經常會用到時間，Java 中最常見簡單的方式是 SimpleDateFormat。但在某些場景反而會出錯

---

![why-simple-date-format-is-bad.png](/assets/image/why-simple-date-format-is-bad.png)


SimpleDateFormat is non-thread safe 在低流量環境使用通常不會出錯，但到了高流量的環境可能會出現 

有問題的程式碼如下
 
```java
public class SimpleDateFormatExample {

    private static final SimpleDateFormat simpleDateFormat = new SimpleDateFormat("mm:ss");

    public static void main(String[] args) {
        
        ExecutorService threadPool = Executors.newFixedThreadPool(10);
        // 执行 10 次时间格式化
        for (int i = 0; i < 10; i++) {
            int finalI = i;
            // 线程池执行任务
            threadPool.execute(new Runnable() {
                @Override
                public void run() {
                    // 创建时间对象
                    Date date = new Date(finalI * 1000);
                    // 执行时间格式化并打印结果
                    System.out.println(simpleDateFormat.format(date));
                }
            });
        }
    }
}
```


> Date formats are not synchronized. It is recommended to create separate format instances for each thread. If multiple threads access a format concurrently, it must be synchronized externally.



## 解法 1. 每次都 new

## 解法 2. ThreadLocal

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
 - [你真的會使用SimpleDateFormat嗎？](https://developer.aliyun.com/article/756625)