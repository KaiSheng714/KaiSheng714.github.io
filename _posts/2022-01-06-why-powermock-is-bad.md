---
layout: post
title: "不建議使用 PowerMock 的理由"
tagline: ""
meta: java,unit-test,powermock,mockito,design,interface
author: "Kai-Sheng"
permalink: /articles/drawback-of-powermock
categories: [Design, Unit-testing]
image: /assets/image/site-image-small.png
--- 

我們在寫 Java 的單元測試時常會使用 mocking framework，因為它能幫助我們輕鬆建立 mocked object，不必再為了單元測試而寫假物件，也更容易對目標物件進行獨立測試，隔絕外部相依，而降低寫單元測試的負擔。目前有許多主流 mocking framework，如最受歡迎的 Mockito，以及本篇文章的主角 — PowerMock。

![powermock](/assets/image/powermock.png?style=center)

[PowerMock](https://github.com/powermock/powermock) 是基於 Mockito 並擴充了許多實用的測試方法。PowerMock 讓開發者可以輕鬆地 mock private method, static, final class 甚至是 constructor 等。簡而言之， Mockito 不能做到的事，PowerMock 幾乎都能一手包辦！不過，在 PowerMock 官方的 README 中說了一段耐人尋味的話:
>
> Please note that PowerMock is mainly intended for people with expert knowledge in unit testing. 
> Putting it in the hands of junior developers may cause more harm than good.
>  

既然 PowerMock 這麼強大，為什麼作者會做出此評論呢? 接著我將加以分析與探討：
 
## **PowerMock 的優點**
- 強大的 mock 功能: 能因應各式難以撰寫測試的情況，尤其是欲在 legacy code 中加入測試時非常實用。
- 用法與 Mockito 類似: 對於熟悉 Mockito 的廣大使用者來說能快速上手。

## **PowerMock 的缺點（不建議使用的理由）**
### **1. 相同的 API**
因為 PowerMock 與 Mockito 有許多 method 的用法是一模一樣的，但兩者間的支援度與行為卻不同。所以如果在 IDE 沒有特別指出，寫出來的程式都會是一模一樣，因此容易被誤用，更難以 debug，而且**你不該花時間 debug 測試程式碼。**

### **2. 同一種 Annotation 卻有不同用法**
例如 `@PrepareForTest` 是 PowerMock 特有的 Annotation，其旨在於告訴 PowerMock 要 mock 測試目標物件的 static, final 等。例如

```java
@PrepareForTest(MyUtils.class)
```

而當你要 mock 的對象是 java 提供的物件(例如 System)時，需要帶入的參數卻是 ”使用 System 的類別”

```java
// wrong
@PrepareForTest(System.class)

// correct
@PrepareForTest(TheClassUseSystem.class)
```

### **3. Overhead**
良好的測試程式大多遵循 **F.I.R.S.T** 原則，不過 PowerMock 的初始化時間比 Mockito 更久，如果測試數量不多，也許還可以忍受；但隨著專案日漸龐大，累積了上百上千的測試案例，此時就容易讓人下 skip test 指令，那就失去了寫測試的意義了。

### **4. 容易忽略 code design**
這也是我認為最大的缺點。正因為 PowerMock 如此 powerful，容易使開發者過於依賴與濫用，原因很簡單，**因為無論 production code 再怎麼雜亂無章都能夠寫出單元測試**，久而久之讓人容易忽略 code design。

### **5. 可能寫出不好的單元測試**
對於 PowerMock 來說， production code 的實作細節一覽無遺，所以開發者很可能會寫很多 arrange、mock 一堆依賴。導致每當程式碼有小改動或重構時，就容易造成測試失敗，讓 test case 難以維護。


如果你對於上面提的幾點有感，覺得 PowerMock 弊大於利，或覺得現階段不適合使用，因而決定棄用，那可以參考以下的方法 — **重構**。
 
## **重構(Refactoring)**
為了從專案移除 PowerMock，最終目標就是**只用或不用 Mockito 也能完成單元測試**，為了達成這個目標，首先我們必需重構程式碼，目的是提高程式碼的可測試性(Testability)，如果可測試性高，可維護(Maintainability)、可讀(Readability)、可理解(Understandability)性自然而然提高了，這對專案的健康是有幫助的。後來才加入的新同事也會很感謝你，因為這樣也能減少他們上手的成本。

以下是幾個簡單的 PowerMock 常見的使用案例，並提供重構方法與思路：

### **測試 static class / method** 
我相信這應該是 PowerMock 受歡迎的最大理由，雖然 static 使用方便、效能較快，但也因此常被濫用，造成物件隱含相依、維護困難、不易測試等問題。因此在使用 static 之前應以更嚴苛的標準來檢視。

舉例來說，下面的 getProperty() 函式，從程式的角度看起來沒問題，但實際上在 Server 尚未啟動時可能產生錯誤或是沒有回傳值。因為這個 method 相依了 Server 的狀態，所以不適合作為 static method，應該改成 instance method。

```java
// bad, it may cause error when server is shutdown.
public static getProperty(String key) {
  return server.getProperty(key);
}
```

理論上，一個良好的 static method 是不需要 mock 的，就讓它執行該做的事吧！如下例子：

```java
// Let it run, don't mock StringUtils.
if (StringUtils.isNullOrEmpty(str)) {
   doSomething();
} else {
   doAnother(); 
}
```

### **doNothing()**
例如你想要驗證 `getData`的回傳值，卻不想執行與測試不相干的 private method `processA` 時，可以使用 PowerMock 的 `doNothing()`

```java
public Data getData(String key) {
  Cache cahce = getCache(); 
  init(cache);
  if (flag) {
    processA();
  } else {
    processB();
    processC();
  }
  calculate();
  return cache.get(key);
}

@Test
public void data_should_be_blabla() {
  // arrange
  PowerMockito.doNothing().when(myClass, method(MyClass.class, "processA")) ...

  // act
  Data result = myClass.getData(key);
  ...
}
```

從上可以看到 getData 做了許多事，乍看之下程式碼篇幅雖然不多，但廣義上也能算是個 `Long Method`。可以思考的是 getData 是否違反 Single Responsibility? 此時可以考慮使用 `Delegate Method` 委派另一個類別，權責分明，測試自然就好寫。

### **Mock System Class**
假設有一函式 `isLate` 用來檢查現在是否超過某個時間，但因 return value 是根據系統當下時間，所以每次執行測試可能會有不同的結果。因此我們需要 mock System.class，如下

```java
// bad design. hard to test.
public boolean isLate() {
  long now = System.currentTimeMillis();
  if (now > 1500000L) {
    return true;
  } else {
    return false;
  } 
}

@Runwith(PowermockRunner.class)
@PrepareforTest(MyClass.class)
public class ExampleTest {

  // ...
  @Test
  public void exceed_some_time_is_late() {
    Powermockito.mockStatic(System.class);
    when(System.currentTimeMillis()).thenReturn(1500001L);
  }

}

```

而比較好的做法是：不讓 method 自己去請求 System 提供現在時間，而是由 caller 傳遞進去，有點像依賴注入 (Dependency Injection, DI) 的觀念，透過 DI 能夠使我們更容易控制輸入端的資料。經過重構後的程式碼，甚至連 mocking framework 都不需要了，如果能不依賴於 framework，通常會是個更好的 practice：


```java
// better
public boolean isLate(long now) {
  return now > 1500000L? true : false;
}

@Test
public void exceed_some_time_is_late() {
  // arrange 
  long now = 1600000L;
  MyClass myClass = new MyClass();

  // act
  boolean result = myClass.isLate(now); 

  // assert
  assertTrue(result);
}
```

### **Mock Constructor**
若寫單元測試時，在 `new` 的地方替換成 mocked object，這時可以使用 PowerMock 提供的 `whenNew()`：

```java
// In MyClass (SUT)
public void doSomething() {
  Dependency dependency = new Dependency();
  dependency.doSomething();
  // ...
}

@Test
public void when_new_example() {
  // arrange
  Dependency mocked = mock(Dependency.class);  
  PowerMockito.whenNew(Dependency.class)
              .withNoArguments()
              .thenReturn(mockedDependency); // bad

  // act
  sut.doSomething();
  ...
}
```

但其實有更好的替代方案：就是用依賴注入。我們先產生 mocked object，做好初始設定後，再透過 constructor 的參數的方式傳入待測函式。如此一來不僅程式增加了彈性，也可以達到的測試目的。

```java
// better
public MyClass(Dependency dependency) {
  this.dependency = dependency;
}

@Test
public void when_new_example() {
 // arrange    
 Dependency mockedDependency = mock(Dependency.class);  
 MyClass sut = new MyClass(mockedDependency);

 // act
 sut.doSomething();
 ...
}
```

## **什麼時候該用？**
講了這麼多 Powermock 的壞處，但存在即合理，一定有它的應用場景，我認為最適當的應用場景就是 legacy code。一個沒有單元測試保護的 legacy code 需要被重構時，通常開發者會先寫一個大範圍的整合測試，有一個基本保護網再去重構，能一定程度的減少改動程式產生 bug 風險，重構完讓 production code，接著再寫單元測試。

除了寫大範圍的整合測試，另一個方案就是使用 Powermock。承上面所說的：對於 PowerMock 來說，production code 的實作細節一覽無遺。因此開發者可以透過 Powermock 的各種 API 去控制待測類別的行為，先寫一個 test case，讓 legacy code 有保護與基本驗證方法後，就能讓開發者更有信心、大膽的重構。

## **結語**
PowerMock 是個功能強大的單元測試工具，但也不可否認的，若使用不當，容易使讓開發人員忽略程式碼品質，導致後續消耗更多開發與維護成本；若是讓對於測試不熟悉的人使用 PowerMock，反而會使他們不知該如何寫出優秀的測試與程式。

如果測試中充斥著 PowerMock，則表示 production code 的可測試性並不好。因此，考慮到專案未來的發展，我建議藉由重構或 TDD，在開發時遵循良好的設計原則，避免寫出 anti-pattern 與 bad smell。撰寫單元測試時，若有必要，使用 Mockito 即可。

我認為 Powermock 只適合用於測試 legacy code。
 
### **References**
[Modern Mocking Tools and Black Magic](https://martinfowler.com/articles/modernMockingTools.html)

### **更多你可能會感興趣的文章**
- [如何寫出優秀的單元測試 (Best Practice)](/articles/good-unit-test)
- [分析 Spring 的依賴注入模式](/articles/analyzing-dependency-injection-patterns-in-spring)
- [常見的 Interface 錯誤用法](/articles/anti-pattern-of-java-interface-impl-style)