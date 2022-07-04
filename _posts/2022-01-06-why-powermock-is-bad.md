---
layout: post
title: "不建議使用 PowerMock 的理由"
tagline: ""
meta: java,unit-test,powermock,mockito,design,interface
author: "Kai-Sheng"
permalink: /articles/drawback-of-powermock
categories: [Design, Unit-testing]
image: /assets/image/powermock.png
--- 

我們在寫 Java 的單元測試時常會使用 mock framework，因為它能幫助我們輕鬆建立 mock object，不必再為了單元測試而寫假物件，也更容易對目標物件進行獨立測試，隔絕外部相依，而降低寫單元測試的負擔。目前有許多主流 mock framework，如最受歡迎的 Mockito，以及本篇文章的主角 — PowerMock。

![powermock](/assets/image/powermock.png?style=center)

## **PowerMock**
[PowerMock](https://github.com/powermock/powermock) 是基於 Mockito 並擴充了許多實用的測試方法。PowerMock 實現了 mock private method、static final class 甚至是 constructor 等。簡而言之 Mockito 不能做到的事，PowerMock 都能一手包辦！不過，在 PowerMock 的 readme 中說了一段耐人尋味的話:
>
> Please note that PowerMock is mainly intended for people with expert knowledge in unit testing. 
> Putting it in the hands of junior developers may cause more harm than good.
>  

既然 PowerMock 這麼強大，為什麼作者會做出此評論呢? 請讓我以優缺點分析作為出發點並探討：

-----

## **PowerMock 的優點**
1. 強大的 mock 功能，能因應各式難以撰寫測試的情況。尤其是欲在 legacy code 中加入測試時非常實用。
2. 對於熟悉 Mockito 的廣大使用者來說能快速上手。

## **PowerMock 的缺點 / 不建議使用的理由**
### **1. 相同的 API**

因為 PowerMock 與 Mockito 有許多 method 的用法是一模一樣的，但兩者間的支援度與行為卻不同。所以如果在 IDE 沒有特別指出，寫出來的程式都會是一模一樣，因此容易被誤用，更難以 debug，而且**你不會也不該花時間 debug test code。**

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

這也是我認為最大的缺點。正因為 PowerMock 如此 powerful，容易使開發者過於依賴與濫用，原因很簡單，**因為無論 production code 再怎麼雜亂無章都能夠寫出單元測試** (而通常在這種情況所寫的單元測試也會是一團亂)，久而久之讓人容易忽略 code design 。

如果你對於上面提的幾點有感，覺得 PowerMock 弊大於利，或覺得現階段不適合使用，因而決定棄用，那可以參考以下的方法 — **重構**。


-----


## **重構(Refactoring)**

為了從專案移除 PowerMock，最終目標就是**只用或不用 Mockito 也能完成單元測試**，為了達成這個目標，首先我們必需重構程式碼，目的是提高程式碼的可測試性(Testability)，如果可測試性高，可維護(Maintainability)、可讀(Readability)、可理解(Understandability)性自然而然提高了，這對專案的健康是有幫助的。後來才加入的新同事也會很感謝你，因為這樣也能減少他們上手的成本。

以下是幾個簡單的 PowerMock 常見的使用案例，並提供重構方法與思路：


### **1. Static class / method**
 
我相信這應該是 PowerMock 受歡迎的最大理由，static 確實會讓寫測試變得很棘手。雖然 static 使用方便、效能較快，但也因此常被濫用，造成物件隱含相依、維護困難、不易測試等問題。因此在使用 static 之前應以更嚴苛的標準來檢視。

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

### **2. Private method**

例如你想要驗證 `getData`的回傳值，卻不想執行與測試不相干的 private method `processA` 時，可以使用 PowerMock 的 `doNothing()`

```java
// bad, long method, and too complex
private Data getData(String key) {
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
  doNothing().when(myClass, method(MyClass.class, "processA")) ...

  // act
  Data result = myClass.getData(key);
  ...
}
```

從上可以看到 getData 做了許多事，乍看之下程式碼篇幅雖然不多，但廣義上也能算是個 `Long Method`。可以思考的是 getData 為何需要做 `processA`與 `processB` 和其他操作呢 ? 是否違反 Single Responsibility ? 此時可以考慮使用 `move method`搬到另一個類別，權責分明，測試自然就好寫，反之，testability 就會大幅降低。而不是試圖從測試程式改變 getData 原有的行為。

回到正題，測試 private method，應該由 public method 作為入口去測試即可。

### **3. System Class**

假設有一函式 `isLate` 用來檢查現在是否超過某個時間，但因 return value 是根據系統當下時間，所以每次執行測試可能會有不同的結果。因此我們需要 mock System，如下

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

而比較好的做法是：不讓 method 自己去請求 System 提供現在時間，而是由 caller 傳遞進去，有點像 Dependency Injection (DI) 的觀念，透過 DI 能夠使我們更容易建立 mock object。經過重構後的程式碼如下(甚至連 mock framework 都不需要了，如果能不依賴於 framework，會是個更好的 practice)


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

### **4. Constructor**

如下例所示，寫測試時，如果想在程式執行 `new A()` 時替換成我們自訂的 mockedA，可以使用 PowerMock 提供的 `whenNew()`：

```java
// bad
public int process() {
  a = new A();
  return a.process();
}

@Test
public void execute_some_example() {
  // arrange
  A mockedA = mock(A.class); 
  PowerMockito.whenNew(A.class).withNoArguments().thenReturn(mockedA);

  // act
  myClass.example();
  ...
}
```

但其實有更好的替代方案，方法與上一個例子的概念很類似，我們先產生 mocked object ，做好初始設定後，再透過參數的方式傳入待測函式。如此一來不僅程式增加了彈性，也可以達到的測試目的。(除此之外，也可以使用 factory pattern 來處理物件的建立)

```java

// better
public MyClass(InterfaceA a) {
  this.a = a;
}

public int process() {
  return a.process();
}

@Test
public void execute_some_example() {
 // arrange    
 InterfaceA mockedA = mock(InterfaceA.class);
 MyClass myClass = new MyClass(mockedA);

 // act
 int result = myClass.process();
 ...
}
```

## **什麼時候該用 Powermock ?**
已經沒有招式能用、無法重構 production code、對眼前的糟糕程式投降的時候。

-----

## **結語**

沒有工具是使用上毫無代價的、萬能的，使用前請停下来想一想。

PowerMock 是個功能強大、非常實用的單元測試工具，但也不可否認的，若使用不當，容易使讓開發人員忽略程式碼品質，導致後續消耗更多開發與維護成本；若是讓對於測試不熟悉的人使用 PowerMock，反而會使他們不知該如何寫出優秀的測試與程式。

如果你的測試充斥著 PowerMock，表示你的 production code 可能已經有許多 bad smell 。因此，考慮到專案未來的發展，我建議你不要使用 PowerMock ，而是藉由重構或 TDD，在開發時遵循良好的設計原則，避免寫出 anti-pattern 與 bad smell。撰寫單元測試時，若有必要，使用一般的 mock framework (如 Mockito) 即可。

更重要的是，在撰寫程式時，稱職的 **clean coder 們應時常思考什麼才是好的設計。**
 
### References

[https://martinfowler.com/articles/modernMockingTools.html](https://martinfowler.com/articles/modernMockingTools.html)