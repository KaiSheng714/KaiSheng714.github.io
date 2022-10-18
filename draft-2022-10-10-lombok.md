---
layout: post
title: "Java Lombok 這招就很夠用"
author: "Kai-Sheng"
permalink: /articles/lombok
categories: [Java]
image: /assets/image/lombok-cover.png
--- 


Lombok包提供了簡單的標注來幫助我們簡化消除一些必須有但是臃腫的java 代碼，比如getter、setter和構造函數等，這些函數一般我們通過IDE自動生成，有了lombok三兩個標注就輕鬆搞定。

Lombok是一款Java開發插件，可以通過它定義的註解來精簡冗長和繁瑣的代碼，主要針對簡單的Java模型對象（POJO）。

好處就顯而易見瞭，可以節省大量重復工作，特別是當POJO類的屬性增減時，需要重復修改的Getter/Setter、構造器方法、equals方法和toString方法等。

而且Lombok針對這些內容的處理是在編譯期，而不是通過反射機制，這樣的好處是並不會降低系統的性能。

本篇介紹一種實用方法，可以輕鬆應用於大部分場景。 

Project Lombok是一個有助於減少樣板代碼的 Java 庫。Java 是一種冗長的語言，可以避免像 getter、setter 等重複代碼。Lombok 使用其註釋減少了樣板代碼，這些註釋在構建過程中被插入。




懶人包: 只要使用這兩個 @Annotation 就足以應付大部分的狀況了

- `@Data`
- `@Accessors(fluent=true)`


```java
@Data
@Accessors(fluent=true)
public class Student {
    private String id;
    private String name;
    private String address;
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
在 class 標註 `@Data` 後，就相當於標註下列五種 Annotation:
### **@Getter**
為每一個 field 產生一個 getter
### **@Setter**
為每一個 non-final field 產生有一個 setter
### **@ToString()**
在java.lang.Object中有個實例方法toString，這個方法的作用是一個對象的自我描述。在源碼中有這樣一句註釋，It is recommended that all subclasses override this method.即推薦所有的子類重新該方法。因為該方法在Object中的實現是返回字符串——類名和該對象的hashCode用“@”符連接起來的字符串，不具有可讀性。所以，需要重寫該方法，使得該方法能夠清楚地表述自己的每一個成員變量。  

不再需要自己 Override `toString`。預設情況下，它會將每個 field 按順序並以逗號分隔，最後輸出成 String，大幅提高了可讀性，在 debug 時很好用。

### **@EqualsAndHashCode**
 註釋一個類時，Lombok 會為我們生成 equals()和hashCode()方法：
By default, Lombok uses all non-static and non-transient fields when generating equals and hashCode - in our case name and salary:

### **@RequiredArgsConstructor**
A constructor that takes one parameter for each final or non-null field with no initial value.
@RequiredArgsConstructor generates a constructor requiring an argument for the final and @NonNull fields.


## **@Accessors(fluent=true)**
A more fluent API for getters and setters.

加了這個後，在呼叫 setter 時可以直接使用 fluent style 而不需要寫 set 前綴。

```java
Student student = new Student()
    .id("A123")
    .name("KaiSheng")
    .address("Taiwan Taipei ...");
```


可能有些人會問我，用 `@Builder` 不就好了嗎? 例如:

```java
Student student = Student.builder()
    .id("A123")
    .name("KaiSheng")
    .address("Taiwan Taipei ...")
    .build();
```
用 `@Builder` 確實能達到同樣目的。但我的經驗是，因為我有時會用 Factory 來建立物件，這時如果在工廠裡再用 Builder Pattern，會有點多此一舉。
而且用我的方式也不需要再呼叫 `builder`, `build`。另外，我通常不會在 model/entity 使用繼承等複雜的操作，所以不考慮多型的狀況下，直接用 `new` 並不會有問題。


## **需要注意的地方:　StackOverflowError**

與 @ToString註解類似，如果兩個類都使用@EqualsAndHashCode，類之間的雙向關係可能會導致java.lang.StackOverflowError：
調用toString方法會StackOverflowError的原因和解決方案

請注意，每個類都有對另一個的引用。當我們調用Employee的hashCode()時，它會給出 java.lang.StackOverflowError。因為每個類都調用其他類的hashCode()方法。



Java的方法參數(method parameters)物件參照或方法內的原始型別變數會存放在JVM的stack記憶體，當thread呼叫一個方法時stack會被建立，而該方法的參數及在方法內產生的本地原始型別變數會被存放在JVM的stack區，如果此時stack記憶體不足便會發生StackOverFlowError錯誤。

注意僅有原始型別資料(primitive type data)及物件參照(object reference address)會存在stack區，而方法內產生的物件則是存在heap區。當方法執行結束後stack變會被釋放。

而通常引起StackOverFlowError的原因是方法被


### **References**
- [Project Lombok](https://projectlombok.org/)
- [Improper equals() and hashCode() Implementations - Baeldung](https://www.baeldung.com/java-memory-leaks#3-improper-equals-and-hashcode-implementations)
- [Why do I need to override the equals and hashCode methods in Java? - StackOverflow](https://stackoverflow.com/a/2265637/5485454)

## **更多你可能會感興趣的文章**
- [常見的 Interface 錯誤用法](/articles/anti-pattern-of-java-interface-impl-style)
- [如何提高程式碼的可測試性 (Testability)](/articles/testability)
- [軟體設計原則 YAGNI (You aren't gonna need it!)](/articles/yagni-principle)