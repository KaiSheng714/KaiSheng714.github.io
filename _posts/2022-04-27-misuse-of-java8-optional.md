---
layout: post
title: "[Java] 多此一舉! 不要這樣用 Optional"
tagline: ""
meta: design,clean-code,java
author: "Kai-Sheng"
permalink: /articles/misuse-of-java-8-optional
categories: [Design, Java]
--- 

![java8-optional](/assets/image/optional.png?size=large)
 
Java 8 中加入了 Optional 新類別來解決 NullPointerException 與繁瑣的 null 檢查，但我看到了不少錯誤的用法，反而讓 Optional 顯得多此一舉。今天就來聊聊錯誤的用法，以及如何正確使用。

## 1. **if isPresent判断，再以 get 取值**
假設有一個 service 用 id 來查詢學生，回傳 `Optional<Student>`，而我們需要取得他的姓名，但如果查無此人，則回傳空字串
### 錯誤
```java
public String readNameById(String id) {
    Optional<Student> student = service.readById(id);
    if (student.isPresent()) {
        return student.get().getName();
    } else {
        return "";
    }
}
```

這應該是最常見的錯誤用法了，可以看到上面的寫法完全不僅多此一舉，還增加了不必要的複雜度，傳統寫法 `if (student != null)` 可能還比較好用。正確使用 Optional 方式改寫如下:

### 正確
```java
public String readNameById(String id) {
    return service.readById(id)
        .map(Student::getName)
        .orElse("");
}
```
其實 Optional 是與 Java 8 lambda 寫法相輔相成的，所以使用 Optional 時，不可使用**傳統逐行指令式**的思考模式下去寫。

## 2. 用在參數

Optional 設計的目的是要讓 method 能夠明確的表示會回傳 **有值** / **沒有值**，而不是 null。但有些錯誤的寫法會將 Optional 作為參數，這會讓邏輯更加複雜，


```java
public int readNameById(Optional<String> id) {
    // ...
}
```

因為這會讓參數有三種狀態:
1. Optional 非 null，且有內容值
2. Optional 非 null，但沒有內容值
3. 整個 Optional 是 null

這種情況下請不要使用 Optional，改用我們平常用的最**純真**的類別即可:

```java
public int readNameById(String id) {
    if (!Strings.isBlank(id)) {

    }
    // my logic
}
```

## 3. 宣告在 class property / field 

```java
public class Student {

    private Optional<String> name;

    // ...
}
```

Optional 是用來設計給 function 的回傳型態，因此它並沒有實作序列化 `Serializable` 介面 ，因此在特定狀況下(如網路傳輸)，需要物件序列化時會出現問題。如果真的是可能是空值的情況下，應該將 Optional 當作 getName 的回傳型態。

```java
public class Student {

    private String name;
    
    public Optional<String> getName() {

    }

    // ...
}
```


## **Reference**

https://stackoverflow.com/questions/71856929/i-want-to-return-a-exception-while-my-return-type-is-dto

http://dolszewski.com/java/java-8-optional-use-cases/

https://rules.sonarsource.com/java/tag/clumsy/RSPEC-3553