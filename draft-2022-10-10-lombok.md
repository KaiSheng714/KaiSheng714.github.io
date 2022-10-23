---
layout: post
title: "Java Lombok @Data 教學與注意事項"
author: "Kai-Sheng"
permalink: /articles/lombok
categories: [Java]
image: /assets/image/lombok-cover.png
--- 

Project Lombok 是實用且被廣泛使用的 library，它可以減少類別的樣板程式（Boilerplate Code），例如 getter, setter, constructor 等，我們只需幾個簡單的 annotation 例如 @Data，就能讓 class 變得五臟齊全，幫助我們省下大量重複的工作，讓開發者更專注於關鍵的邏輯，進而提高開發效率。本文介紹一種 Lombok 實用的使用方法，可用於大部分場景。另外我認為過度使用 Lombok 可能會導致問題，使程式更加複雜。
 

## **@Data 分析**
Lombok 中的 `@Data` 應該是最常被使用的 annotation，它其實是下列五種 annotation 的組合：
### **@Getter**
為每一個 field 產生一個 getter。
### **@Setter**
為每一個 non-final field 產生一個 setter。
  
### **@RequiredArgsConstructor**
建立一個 constructor，其參數為 class 中所有 `@NotNull`, `final` field。

### **@ToString()**
在 java.lang.Object 中有個實例方法 toString()，這個方法的作用是一個對象的自我描述。如果不 override，則會回傳一個 hashCode，不具備可讀性。所以，通常建議每個 class 都要需要 override `toString()`。一旦有了這個 annotation 後，預設情況下，Lombok 會將每個 field 按順序並以逗號分隔，大幅提高可讀性。

```
Student(id=A123, name=KaiSheng, address=Taiwan Taipei)
```

### **@EqualsAndHashCode**
有了這個標註了之後， Lombok 就會以所有 non-static 和 non-transient field 來幫我們實作 `equals()` 和 `hashCode()`.

另一種選擇是使用像 Hibernate 這樣的 ORM 工具，它使用 equals() 和 hashCode() 方法來分析對象並將它們保存在緩存中。如果這些方法沒有被覆蓋，造成 **Memory Leak** 的風險性是相當高的，因為 Hibernate 不能比較對象並且會用重複的對象填充它的緩存。

如果不 override 這兩個 method，一旦 model 放入 HashSet 或當作 HashMap 的 key 值時，容易引發 memory leak。

How to Prevent It?

- 根據經驗，在定義新實體時，始終覆蓋 equals() 和 hashCode() 方法。
- 僅僅覆蓋是不夠的，還必須以最佳方式覆蓋這些方法。


## **注意1. StackOverflowError**

與 `@ToString` 註解類似，如果兩個類都使用 `@EqualsAndHashCode`，類之間的雙向關係可能會導致java.lang.StackOverflowError：
調用toString方法會StackOverflowError的原因和解決方案

請注意，每個類都有對另一個的引用。當我們調用Employee的hashCode()時，它會給出 java.lang.StackOverflowError。因為每個類都調用其他類的hashCode()方法。


Java的方法參數(method parameters)物件參照或方法內的原始型別變數會存放在JVM的stack記憶體，當thread呼叫一個方法時stack會被建立，而該方法的參數及在方法內產生的本地原始型別變數會被存放在JVM的stack區，如果此時stack記憶體不足便會發生StackOverFlowError錯誤。

注意僅有原始型別資料(primitive type data)及物件參照(object reference address)會存在stack區，而方法內產生的物件則是存在heap區。當方法執行結束後stack變會被釋放。

而通常引起StackOverFlowError的原因是方法被

通過反編譯School類和Student類,我們發現它們的hashCode()方法存在循環引用。
看School類中的hashCode()方法，studentList是一個HashSet集合，HashSet集合的hashCode()計算方式會遍歷所有元素，累加求和每個元素的hashCode值。但是studentList裡面元素的類型是Student，Student類中的hashCode()又會依賴於School類的hashCode()方法，這樣就形成了循環依賴。

## **注意2: Jackson 反序列化**



## **@Builder, Setter" 
必要時可加 `@NonNull`，fail fast

## **注意3: 需要所有 getter?**
在我的經驗中，getter, setter 既然這麼方便就產生了，所以自然而然就拿來用，因此很多人就忽略了 **Tell, Don't Ask**.

Tell-Don't-Ask 是一個幫助人們記住面向對像是將數據與操作該數據的函數捆綁在一起的原則。它提醒我們，與其向對象索取數據並根據該數據採取行動，不如告訴對象該做什麼。這鼓勵將行為移動到對像中以與數據一起使用。

我看到的最多被违反的原则是“命令，不要去询问(Tell, Don’t Ask)”原则。这个原则讲的是，一个对象应该命令其它对象该做什么，而不是去查询其它对象的状态来决定做什么(查询其它对象的状态来决定做什么也被称作‘功能嫉妒（Feature Envy）’)。

封裝 (Encapsulation)  

必要時可加 `@Getter(AccessLevel.NONE)`


## **我常用的方式**
只要使用這兩個 @Annotation 就足以應付大部分的狀況了
- `@Data`
- `@Accessors(chain=true)`

```java
@Data
@Accessors(chain = true)
public class Student {
    private String id;
    private String name;
    private String address; 
}
```

```java
Student student = new Student()
    .setId("A123")
    .setName("KaiSheng")
    .setAddress("Taiwan Taipei");
```

## **@Accessors(chain=true)**
在 class 標註 `@Accessors(chain=true)` 後，就可將一連串的 setter 串接起來。可能有些人會問我：用 `@Builder` 不是更好嗎? 例如:

```java
Student student = Student.builder()
    .id("A123")
    .name("KaiSheng")
    .address("Taiwan Taipei ...")
    .build();
```
用 `@Builder` 確實能達到同樣目的，而且比較美觀。但我的經驗是，因為我有時會用 Factory 來建立物件，這時如果在工廠裡再用 Builder Pattern，會有點多此一舉。

- 而且用我的方式也不需要再呼叫 `builder`, `build`。
- 另外，我通常不會在 model/entity 使用繼承等複雜的操作，所以不考慮多型的狀況下，直接用 `new` 並不會有問題。
- 再來，Jackson 做 json 反序列化時，如果用 `@Builder` 就會因為沒有 setter 而導致失敗，需要另外加上其他 annotation 例如 `@Jacksonized`。但我的觀點是，不要過度依賴 library 的魔法，如果越能降低專案的學習門檻就越好。

這裡當然沒有標準答案，青菜蘿蔔各有所好。
雖然 Lombok 雖然很方便，但世上總是沒有十全十美的工具


## **後記**
Lombok有優點也有缺點，如果熟知優點缺點之後，在程式碼中運用，對程式人員來說也可以造成寫程式便利。
在沒有真正理解框架幹了什麼之前，不要對框架充分信任。我們要明白Lombok框架幹了什麼，不然出現一堆問題就懵逼了。
沒有完美的解法，但要懂得取捨

### **References**
- [Project Lombok](https://projectlombok.org/)
- [Improper equals() and hashCode() Implementations - Baeldung](https://www.baeldung.com/java-memory-leaks#3-improper-equals-and-hashcode-implementations)
- [Why do I need to override the equals and hashCode methods in Java? - StackOverflow](https://stackoverflow.com/a/2265637/5485454)

### **更多你可能會感興趣的文章**
- [常見的 Interface 錯誤用法](/articles/anti-pattern-of-java-interface-impl-style)
- [如何提高程式碼的可測試性 (Testability)](/articles/testability)