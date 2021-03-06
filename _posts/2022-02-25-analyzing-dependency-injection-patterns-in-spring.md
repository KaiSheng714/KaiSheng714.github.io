---
layout: post
title: "分析 Spring 的依賴注入模式"
tagline: ""
meta: java,spring,test,clean-code,dependency-injection
author: "Kai-Sheng"
permalink: /articles/analyzing-dependency-injection-patterns-in-spring
categories: [Design, Spring]
image: /assets/image/spring-di.png
---

依賴注入 (Dependency Injection, DI) 是 Spring 實現控制反轉（IoC）的重要手段。Spring 提供了數種 DI Patterns，其中最常用的是 **field based injection**，它是許多人第一次使用 Spring 時所使用的 pattern。雖然這方式簡單易用卻有不少缺點。
 

例如你會發現， IntelliJ 很貼心地告訴我們:


> 
> Field Injection is not recommended.
> 
> Spring Team recommends: "Always use constructor based dependency injection in tour beans. Always use assertions for mandatory dependencies".
> 

![Field Injection is not recommended](/assets/image/spring-di.png?style=center)

為何 **constructor based injection** 優於 **field based injection** 呢 ?

接下來我會解析這兩種 Dependency Injection Pattern.

## **1. Field Based Dependency Injection**

**這種注入方式顧名思義，就是直接在 field 加上 @Autowired**

```java
@Component
public class HelloBean {
  
   @Autowired
   private AnotherBean anotherBean;
  
   @Autowired
   private AnotherBean2 anotherBean2;
  
   // ...
```

### **優點**
1. 簡單方便易用，只要短短一行即可完成。

### **缺點**
1. 不易維護，**因為簡單方便，更容易產生 code smell 而不自知，例如** `God Object`
2. 不好測試，測試環境需要透過 DI container 並加上許多 @annotation 來初始化，看起來更像整合測試了。而且編譯執行時會多一些 overhead，也較不容易除錯。
3. 不好理解測試，以下範例程式為例

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

* `@InjectMocks` 做了什麼?
* 是否需要將待測物件 HelloBean 實體化呢 ?
* 如果有兩個 `AnotherBean`怎麼辦 ?

只有短短幾行就讓人產生諸多疑問，理解成本較高。若使用 Constructor Based Dependency Injection 則不易產生此問題。下面會詳述。

## **2. Constructor Based Dependency Injection**

**此方式最大的特點就是: Bean 的建立與依賴入是同時發生的**

```java
@Component
public class HelloBean {
 
   private final AnotherBean anotherBean;
   
   @Autowired
   public HelloBean(AnotherBean anotherBean) {
       this.anotherBean = anotherBean;
   }
   
   // ...
}
```

### **優點**
### **1. 容易發現 code smell**

假設我們需要注入 10 個 bean ，對比 Field 注入的方式，這種方式暴露了 constructor 中含有過多的參數，正常的開發者看到 10 個參數肯定是會頭痛的，這就表示我們需要想辦法重構它。

### **2. 容易測試**

它不需要任何 JUnit 以外的 @Annotation，這不僅讓程式是看起來更乾淨了，也降低了理解與維護成本。就算是不熟 Java 或 Mockito 的開發人員應該也能看得懂 70~80%。


```java
public class HelloBeanTest {
    
    private HelloBean helloBean;
    
    @Before
    public void setup() {    
        AnotherBean anotherBean = mock(AnotherBean.class);
        helloBean = new HelloBean(anotherBean);
    }
  
    // Test cases...
}
```

### **3. 不可變物件 (Immutable Object)**

意思是 Bean 在被創造之後，它的內部 state, field 等就無法被改變。不可變意味著唯讀，因而具備執行緒安全 (Thread-safety) 的特性。此外，相較於可變物件，不可變物件在一些場合下也較合理、易於了解，而且提供較高的安全性，是個良好的設計。因此，透過 constructor based DI，再把依賴宣都告成 **final**，就可以輕鬆建立 Immutable Object。

### **缺點**

### **1. 循環依賴問題 ([Circular dependency issues](https://en.wikipedia.org/wiki/Circular_dependency))**

只有在使用 Constructor DI 時才會造成此問題。

舉個簡單的例子，若依賴關係圖: Bean C → Bean B → Bean A → Bean C ，則會造成造成此問題， 程式在 Runtime 會拋出`BeanCurrentlyInCreationException` ，造成程式 crash。更白話一點，這就是**雞生蛋 / 蛋生雞的**問題。

![循環依賴問題 Circular dependency issues ](https://miro.medium.com/max/1044/1*vClDWHcM4nKPUz9uWksl-Q.png?style=center)

但是， [Circular dependency issues](https://en.wikipedia.org/wiki/Circular_dependency) 是一種 **Anti-Pattern**，所以如果能夠即時發現，提早讓開發人員意識到該問題重新設計此 bean，我個人認為這反而是個不錯的缺點。

## **總結**

本文介紹了兩種依賴注入模式。最常見的是 field based DI，很不幸的這種注入方式會造成程式的不良影響與 code smell，但依舊有許多人使用此方式。另外，Spring 官方團隊建議開發者使用 **constructor based DI**，雖然可能會有循環依賴的問題，但無論在開發、測試方面，總體而言都是利大於弊。

## **References**

- [Dependency injection patterns](https://kinbiko.com/java/dependency-injection-patterns/)  
- [What exactly is field injection and how to avoid it](https://stackoverflow.com/questions/39890849/what-exactly-is-field-injection-and-how-to-avoid-it/39891473)  
- [Circular dependencies in spring](https://www.baeldung.com/circular-dependencies-in-spring)
