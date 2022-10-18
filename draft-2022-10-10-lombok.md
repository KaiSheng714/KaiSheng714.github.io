---
layout: post
title: "Lombok 學這招就夠用"
author: "Kai-Sheng"
permalink: /articles/lombok?ks=true
categories: [Java]
image: /assets/image/lombok.png
--- 


Lombok包提供了簡單的標注來幫助我們簡化消除一些必須有但是臃腫的java 代碼，比如getter、setter和構造函數等，這些函數一般我們通過IDE自動生成，有了lombok三兩個標注就輕鬆搞定。

Lombok是一款Java開發插件，可以通過它定義的註解來精簡冗長和繁瑣的代碼，主要針對簡單的Java模型對象（POJO）。

好處就顯而易見瞭，可以節省大量重復工作，特別是當POJO類的屬性增減時，需要重復修改的Getter/Setter、構造器方法、equals方法和toString方法等。

而且Lombok針對這些內容的處理是在編譯期，而不是通過反射機制，這樣的好處是並不會降低系統的性能。


![lombok](/assets/image/lombok.png?style=center)




Project Lombok是一個有助於減少樣板代碼的 Java 庫。Java 是一種冗長的語言，可以避免像 getter、setter 等重複代碼。Lombok 使用其註釋減少了樣板代碼，這些註釋在構建過程中被插入。



懶人包: 只要這兩個 @Annotation 就足以應付大部分的狀況了

- `@Data`
- `@Accessors(fluent=true)`


```java
@Data
@Accessors(fluent=true)
public class Student {
    private String id;
    private String name;
    private String address;
    private String phone;
}
```

```java
@Test
 
```


## **Install**

IntelliJ IDEA
The Jetbrains IntelliJ IDEA editor is compatible with lombok without a plugin as of version 2020.3.

```xml
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>1.18.24</version>
</dependency>
```


## **@Data**

The project Lombok authors decided to work on a quick solution to eliminate all this boilerplate code with just one annotation.

Adding @Data annotation right on top of your class informs Lombok to automatically generate:-


Equals and hashCode methods.
Setters for each non-final field.
A toString method.
A constructor that takes one parameter for each final or non-null field with no initial value.
Getters for each field.


## **@Accessors(fluent=true)**


A more fluent API for getters and setters.








## **需要注意的地方**
調用toString方法會StackOverflowError的原因和解決方案
 
## **結語**
 

### **References**
- [Project Lombok](https://projectlombok.org/)

## **更多你可能會感興趣的文章**
- [常見的 Interface 錯誤用法](/articles/anti-pattern-of-java-interface-impl-style)
- [如何提高程式碼的可測試性 (Testability)](/articles/testability)
- [軟體設計原則 YAGNI (You aren't gonna need it!)](/articles/yagni-principle)