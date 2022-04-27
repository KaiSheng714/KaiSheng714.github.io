---
layout: post
title: "多此一舉! 不要這樣用 Java 8 Optional"
tagline: "Design, Java"
author: "Kai-Sheng"
permalink: /articles/misuse-of-java-8-optional
categories: [Design, Java]
--- 

Java 8 中新加入了 Optional 類別來避免 NullPointerException 問題與繁瑣的 null 檢查，可以讓程式邏輯看起來更簡潔、易讀，也能清楚表達沒有結果值。但我卻看到了不少錯誤的用法，反而讓 Optional 顯得多此一舉。今天就來聊聊錯誤的用法，以及如何正確使用。

![java8-optional](/assets/image/optional.png?size=large)
 
## **錯誤1. isPresent() and get()**
假設有一個 service 用 id 來查詢學生，回傳 `Optional<Student>`，而我們需要取得他的姓名並轉換成大寫，但如果查無此人，則回傳空字串

```java
public static String readUpperCaseNameById(String id) {
    Optional<Student> student = service.readById(id);
    if (student.isPresent()) {
        if (student.get().getName() != null) {
            return student.get().getName().toUpperCase();
        } else {
            return "";
        }
    } else {
        return "";
    }
}
```

這應該是最常見的錯誤用法了，可以看到上面的 `isPresent()`, `get()`寫法不僅多此一舉，還增加了不必要的複雜度。正確使用 Optional 方式改寫如下:
 
```java
public static String readUpperCaseNameById(String id) {
    return service.readById(id)
        .map(Student::getName)
        .map(String::toUpperCase)
        .orElse("");
}
```
其實 Optional 是與 Java 8 functional programming 寫法相輔相成的，所以使用 Optional 時應搭配如 filter(), map(), flatMap() 的**鏈式**處理方法，不可使用**傳統逐行指令式**的思考模式下去寫。

## **錯誤2. 作為參數**

Optional 設計的目的是要讓 method 能夠明確的表示會回傳 **有值** / **沒有值**。但有些錯誤的寫會將 Optional 作為參數，讓邏輯更加複雜，

```java
public int readNameById(Optional<String> id) {
    // ...
}
```

因為這會讓參數有三種狀態:
1. Optional 非 null，且有內容值
2. Optional 非 null，但沒有內容值
3. 整個 Optional 是 null

因此在這種情況下請不要使用 Optional，改用我們平常用的最**純粹**的型態即可:

```java
public int readNameById(String id) {
    if (!Strings.isBlank(id)) {
      // my logic
    }
}
```

## **錯誤3. 作為 class property**

```java
public class Student {

    private Optional<String> name;
    // ...
}
```

Optional 是用來設計給 function 的回傳型態，因此它並沒有實作序列化 `Serializable` 介面 ，在特定狀況下(如網路傳輸)需要物件序列化時將會出現問題。
再以 `Student` 為例，如果姓名可能是空值的情況下，應該將 Optional 當作 getName 的回傳型態。

```java
public class Student {

    private String name;

    public Optional<String> getName() {
        // ...
    }
}
```

 ### **References**

- [java-8-optional-use-cases](http://dolszewski.com/java/java-8-optional-use-cases/)
- [RSPEC-3553](https://rules.sonarsource.com/java/tag/clumsy/RSPEC-3553)