---
layout: post
title: "分析 Spring 的依賴注入模式"
tagline: ""
meta: java,spring,test,clean-code,dependency-injection
author: "Kai-Sheng"
permalink: /articles/analyzing-dependency-injection-patterns-in-spring
categories: [Design, Spring]
image: /assets/image/site-image-small.png
---

依賴注入 (Dependency Injection, DI) 是 Spring 實現控制反轉概念的重要手段。Spring 提供了數種 DI patterns，其中最方便、最常用的是 **field injection**，它應該是許多人第一次寫 Spring 專案時所使用的 pattern，雖然這方式簡單易用，卻有不少缺點。
 

例如你會發現， IntelliJ IDEA 會很貼心地告訴我們:

> 
> Field Injection is not recommended.
> 
> Spring Team recommends: "Always use constructor based dependency injection in tour beans. Always use assertions for mandatory dependencies".
> 

![Field Injection is not recommended](/assets/image/spring-di.png?style=center)

為何 constructor injection 優於 field injection 呢？接下來我會解析這兩種 pattern. （雖然 Spring 還有其他種注入方式，但我比較不常用，所以就不在此介紹了)

## **Field Injection**

這種注入方式顧名思義，就是直接在 field 加上 `@Autowired`

```java
@Component
public class HelloBean {
  
   @Autowired private AnotherBean anotherBean;
  
   @Autowired private AnotherBean2 anotherBean2;
  
   // ...
```

### **優點**
- 簡單方便易用，只要短短一行即可完成。
- 程式碼最少，讀起來真舒服

### **缺點**
- 不易維護，**因為簡單方便，更容易產生 code smell 而不自知**，例如 **God Object**
- 不好寫單元測試，測試環境需要透過 DI container 並加上許多 @Annotation 來初始化，看起來更像整合測試了。而且編譯、執行時會多一些 overhead。
- 不好理解測試，以下程式為例

```java
@RunWith(MockitoJUnitRunner.class)
public class HelloBeanTest {

    @Mock
    private AnotherBean anotherBean;
    
    @Mock
    private AnotherBean2 anotherBean2;
    
    ...
    
    @Mock
    private AnotherBean10 anotherBean10;
    
    @InjectMocks
    private HelloBean helloBean;
    
    @Before
    public void setup() {
        ...
    }
    
    // Test cases...
}
```

這是相當常見的 Mockito+Junit 單元測試寫法，但容易造成疑問：

- `@RunWith(MockitoJUnitRunner.class)` 是什麼意思 ?
- `@InjectMocks` 做了什麼 ?
- 是否需要將待測物件 `HelloBean` 實體化呢 ?
- 如果有兩個 `AnotherBean` 類型的依賴怎麼辦 ?

只有短短幾行就讓人產生諸多疑問，因此理解成本較高。雖然這種注入方式很簡單方便，但**寫單元測試時就得還債了**。若使用 constructor injection 則不易產生此問題，我們接著看下去：

## **Constructor Injection**
此方式最大的特點是: Bean 的建立與依賴的注入是同時發生的

```java
@Component
public class HelloBean {
 
   private final AnotherBean anotherBean;
   private final AnotherBean2 anotherBean2;
   // ...
   
   @Autowired
   public HelloBean(AnotherBean anotherBean, AnotherBean2 anotherBean2, ...) {
       this.anotherBean = anotherBean;
       this.anotherBean2 = anotherBean2;
       // ...
   }
   
   // ...
}
```
### **優點1. 容易發現 code smell**
假設我們需要注入十幾個 dependecies，對比 field injection 的方式，這種方式暴露了 constructor 中含有過多的參數 (Long Parameter List)，這是個很好的**臭味偵測器**，正常的開發者看到這麼多參數肯定是會頭痛的，這就表示我們需要想辦法重構它，盡可能使它符合單一職責原則 (Single Responsibility Principle)。

### **優點2. 容易釐清依賴關係**
一看到 constructor 就可以讓開發者釐清這個物件所需要的 dependency，且缺一不可，進而縮小該物件在專案中的使用範圍，事物的範圍越窄，就越容易理解與維護。另外，我們也可以透過 constructor 注入假的依賴，進而容易寫單元測試。

### **優點3. 容易寫單元測試**

一個簡單的範例：

```java
public class HelloBeanTest {
    
    private HelloBean helloBean;
    
    @Before
    public void setup() {
        AnotherBean anotherBean = mock(AnotherBean.class);
        AnotherBean2 anotherBean2 = mock(AnotherBean2.class);
        // ...
        helloBean = new HelloBean(anotherBean, anotherBean2, ...);
    }
  
    // Test cases...
}
```

相較前面的例子，這種注入方式不需要太多 @Annotation，讓測試程式碼看起來更乾淨了，我們也能輕鬆的用 `new` 來實體化待測物件、注入假依賴，整體而言看起來更 **清楚、好理解**，就算是不熟 Java 或 Mockito 的開發人員應該也能看得懂七八成，對於新人也比較好上手，而且也比較不會有誤用 @Annotation 所產生額外成本，**[優秀的單元測試](/articles/good-unit-test)**就應該如此。

### **優點4. Immutable Object**
意思是 Bean 在被創造之後，它的內部 state, field 就無法被改變了。不可變意味著唯讀，因而具備執行緒安全 (Thread-safety) 的特性。此外，相較於可變物件，不可變物件在一些場合下也較合理、易於了解，而且提供較高的安全性，是個良好的設計。因此，透過 constructor injection，再把依賴宣都告成 **final**，就可以輕鬆建立 Immutable Object。


### **缺點：循環依賴**
只有在使用 constructor injection 時才會造成此問題。

舉個簡單的例子，若依賴關係圖: Bean C → Bean B → Bean A → Bean C ，則會造成造成此問題，程式在 Runtime 會拋出`BeanCurrentlyInCreationException`，更白話來說，這就是**雞生蛋 / 蛋生雞的**問題，而 Spring 容器初始化時無法解決這樣的窘境，因此拋出例外並中斷程式。

![循環依賴問題 Circular dependency issues](https://miro.medium.com/max/1044/1*vClDWHcM4nKPUz9uWksl-Q.png?style=center)

但是，[Circular dependency](https://en.wikipedia.org/wiki/Circular_dependency) 其實算是一種 **Anti-Pattern**，所以如果能夠即時發現它，提早讓開發人員意識到該問題重新設計此 bean，我個人認為這點反而蠻好的。

## **總結**
本文介紹了兩種依賴注入模式，它們各有好壞，也都能達到同樣的目的，而比較常見的是 field injection，但不幸的這種方式較可能會寫出 code smell。另外，Spring 官方團隊建議開發者使用 **constructor injection**，雖然可能會有循環依賴異常，但無論在開發、測試方面，總體而言都是利大於弊，我也一直遵循這個模式。

### **References**
- [Dependency injection patterns](https://kinbiko.com/java/dependency-injection-patterns/)  
- [What exactly is field injection and how to avoid it](https://stackoverflow.com/questions/39890849/what-exactly-is-field-injection-and-how-to-avoid-it/39891473)  
- [Circular dependencies in spring](https://www.baeldung.com/circular-dependencies-in-spring)

### **更多你可能會感興趣的文章**
- [如何提高程式碼的可測試性 (Testability)](/articles/testability)
- [Spring + Maven + IntelliJ 多環境 (Profile) 整合技巧](/articles/spring-profile-with-maven-package)
- [常見的 Interface 錯誤用法](/articles/anti-pattern-of-java-interface-impl-style)