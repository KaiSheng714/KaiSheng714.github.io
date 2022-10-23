---
layout: post
title: "Java Lombok @Data 教學與注意事項"
author: "Kai-Sheng"
permalink: /articles/lombok
categories: [Java]
image: /assets/image/lombok-cover.png
--- 

Project Lombok 是很實用且被廣泛使用的語法糖 library，它可以減少許多樣板程式，例如 getter, setter 等，我們只需幾個簡單的 annotation，例如 @Data，就能讓 class 變得五臟齊全，幫助我們省下大量重複的工作，讓開發者更專注於關鍵的邏輯，進而提高開發效率。另外，我認為過度使用 Lombok 可能會導致問題，使程式更加複雜。


![lombok](/assets/image/lombok-title.png)


Lombok 的 annotation 都是在編譯時才被轉換成 java code，因此早期使用 Lombok 時，開發者除了需要引入 dependency 以外，還要在 IDE 安裝套件才能正常使用 Lombok。時至今日，一切已經變得非常容易，它甚至直接被整合進了 IntelliJ IDEA，無痛使用；而若是用 Eclipse 的開發者就要比較麻煩一點了，可以參考官方的[安裝教學](https://projectlombok.org/setup/eclipse)

## **Lombok @Data**
Lombok 中的 `@Data` 應該是最常被使用的 annotation，根據官方的文件，其成分如下：

### **@Getter**
為每一個 field 產生一個 getter。

### **@Setter**
為每一個 non-final field 產生一個 setter。

### **@RequiredArgsConstructor**
建立一個 constructor，其參數為 class 中所有 `@NotNull`, `final` field。

### **@ToString()**
將每個 field 按順序並以逗號分隔，以 name=value 的形式組成字串，大幅提高可讀性。

### **@EqualsAndHashCode**
有了這個標註了之後， Lombok 就會以所有 non-static 和 non-transient field 來實作 `equals()` 和 `hashCode()`。如果不能夠良好的 override 這兩個 method，一旦 entity/model 放入 HashSet 或當作 HashMap 的 key 值時，[可能會引發 memory leak](https://www.baeldung.com/java-memory-leaks#3-improper-equals-and-hashcode-implementations)。
 
DTO 是最簡單的一種 class，它只保存資料，沒有邏輯，如果是 DTO，基本上可以無腦使用 `@Data` 也不會產生太多問題。如果是 domain model 則應該關心以下幾個議題：

## **注意事項**

### **Delombok**
知己知彼，百戰百勝。用 Lombok 真的很方便，不過更重要的是要了解**它實際產生了什麼程式、它隱藏的細節？**，否則有時容易踩了坑還找不出原因。想要查看 Lombok 反編譯後的樣子，就可以透過 **Delombok** 這面照妖鏡來查看，步驟為在 IntelliJ 編輯區按右鍵 → refactor → Delombok。

### **StackOverflowError**
因為 `@Data` 中包含了 `@ToString`，`@EqualsAndHashCode`，如果兩個之間的有雙向依賴關係，就會導致 `java.lang.StackOverflowError`，例如 Student 類別依賴 Teacher 類別， Teacher 類別也依賴 Student，並且這兩個類別都標記了 `@ToString` 或 `@EqualsAndHashCode`。
透過 Delombok，可以發現它們的 `hashCode()`, `toString()` 都存在循環引用，造成了無窮迴圈，最後因為記憶體不足拋出例外。最好的解決方法就是藉由重構來消除雙向依賴的關係。

### **@Builder, @Setter**
```java
Student student = Student.builder()
    .id("A123")
    .name("KaiSheng")
    .address("Taiwan Taipei ...")
    .build();
```
用 `@Builder` 確實能達到同樣目的，而且比較美觀。通常域對像很小（或應該很小）並且@Builder具有非常危險的行為：它允許在無效狀態下創建對象，有時候難以發現的 bug。
一旦我們用 註釋這個類 `@Builder`，沒有什麼能阻止我這樣做：

必要時可加 `@NonNull`，fail fast

## **@Builder 與 Jackson **
Jackson 是 Java 中應用非常廣泛的序列化、反序列化的 library，它可以幫助我們簡單、快速將 Java 物件與 json 之間作轉換，就連 Spring 將 Jackson 的 ObjectMapper 作為預設使用。

如果我們使用 `@Builder`，無參數的 constructor 會被設成 package-private，這時若我們反序列化 json 字串，就會導致 Jackson 無法找到 constructor 並拋出 `InvalidDefinitionException`。

```
Error on Jackson Deserialization
com.fasterxml.jackson.databind.exc.InvalidDefinitionException: 
Cannot construct instance of `...  cannot deserialize from Object value 
(no delegate- or property-based Creator)
```

解決辦法是在 class 上多加一個 `@Jacksonized` 即可。不過，我並不喜歡這個作法，因為它還是實驗性質而已，有可能隱含未知的風險，且要讓團隊花額外時間去了解、學習它的意義，太累了，如果越能降低專案的學習門檻就越好。順帶一提，Google Gson 能夠直接序列化與反序列化 `@Builder` class，無需額外處理。

## **需要所有 getter ？**
在我的經驗中，getter, setter 既然這麼方便就產生了，所以自然而然就拿來用，因此讓很多人忽略了 **Tell, Don't Ask** 原則。

Tell, Don't Ask 是一個幫助人們記住面向對像是將數據與操作該數據的函數捆綁在一起的原則。它提醒我們，與其向對象索取數據並根據該數據採取行動，不如告訴對象該做什麼。這鼓勵將行為移動到對像中以與數據一起使用。

我看到的最多被违反的原则是“命令，不要去询问(Tell, Don’t Ask)”原则。这个原则讲的是，一个对象应该命令其它对象该做什么，而不是去查询其它对象的状态来决定做什么(查询其它对象的状态来决定做什么也被称作‘功能嫉妒（Feature Envy）’)。

封裝 (Encapsulation)  

必要時可加 `@Getter(AccessLevel.NONE)`，或是在指定的 field 個別標記 `@Setter` 和 `@Getter`，而不是標記在 class level 。
 

## **我常用的方式**
只要使用這三個 @Annotation 就足以應付大部分的狀況了
- `@Data`
- `@Accessors(chain=true)`
- `@NotNull`

```java
@Data
@Accessors(chain = true)
public class Student {
    @NotNull
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
在 class 標註 `@Accessors(chain=true)` 後，就可將一連串的 setter 串接起來。可能有些人會問我：用 `@Builder` 不是更好嗎? 
但我的經驗是，因為我有時會用 Factory 來建立物件，這時如果在工廠裡再用 Builder Pattern，會有點多此一舉。而且用我的方式也不需要再呼叫 `builder`, `build`。
 
 
這裡當然沒有標準答案，青菜蘿蔔各有所好。
雖然 Lombok 雖然很方便，但世上總是沒有十全十美的工具

## **後記**
Lombok有優點也有缺點，如果熟知優點缺點之後，在程式碼中運用，對程式人員來說也可以造成寫程式便利。
在沒有真正理解框架幹了什麼之前，不要對框架充分信任。我們要明白Lombok框架幹了什麼，不然出現一堆問題就懵逼了。
沒有完美的解法，但要懂得取捨，要看使用場合

### **References**
- [Project Lombok](https://projectlombok.org/)

### **更多你可能會感興趣的文章**
- [常見的 Interface 錯誤用法](/articles/anti-pattern-of-java-interface-impl-style)
- [如何提高程式碼的可測試性 (Testability)](/articles/testability)