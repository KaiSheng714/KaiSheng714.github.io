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

寫單元測試時常會使用 mocking framework，因為它能幫助我們輕鬆建立 mocked object，不必再為了單元測試而寫假物件，更容易對待測物件隔絕外部相依，進而降低寫單元測試的負擔。目前有許多主流 mocking framework，如最受歡迎的 Mockito，以及本篇文章的主角 — PowerMock。

![powermock](/assets/image/powermock.png?style=center)

[PowerMock](https://github.com/powermock/powermock) 是基於 Mockito 並擴充了許多實用測試方法。PowerMock 讓開發者可以在測試中輕易的模擬 private method, static, final class 甚至是 constructor 等。簡而言之， Mockito 不能做到的事，PowerMock 幾乎都能一手包辦！不過，在 PowerMock 官方的 README 中說了一段耐人尋味的話:
>
> Please note that PowerMock is mainly intended for people with expert knowledge in unit testing. 
> Putting it in the hands of junior developers may cause more harm than good.
>  

既然 PowerMock 這麼強大，為何作者做出此評論呢？ 接著我將加以分析與探討：
 
## **PowerMock 的優點**
- 強大的 mock 功能: 能因應各種難以撰寫測試的情況，尤其是針對棘手的 legacy code，可以在不改 production code 的條件下加入測試。
- 用法與 Mockito 類似: 對於熟悉 Mockito 的廣大使用者來說能快速上手。

## **PowerMock 的缺點**
### **1. 相同的 API**
因為 PowerMock 與 Mockito 有許多 API 的用法是一模一樣的，但兩者間的支援度與行為卻不同。所以如果在 IDE 沒有特別指出，寫出來的程式都會是一模一樣，因此容易被誤用，更難以 debug，而且開發者不該花時間 debug 測試程式碼。

### **2. 複雜的用法**
PowerMock 提供了很多的神奇的 API，例如 `@PrepareForTest`, `@SuppressStaticInitializationFor`, `Whitebox` 等等，研究它們 + 寫測試的時間可能比開發更久，不僅如此，可能三個月後就看不懂，其他同事讀起來也困難，造成不好維護。此外，若升級 Powermock 的版本，也可能會讓舊的寫法失效，甚至導致意想不到的問題。

### **3. Overhead**
優秀的單元測試速度要快，不過 PowerMock 的初始化時間比 Mockito 更久，如果測試數量不多，也許還可以忍受；但隨著專案漸漸龐大，累積了上百上千的測試案例，此時就容易讓人下 skip test 指令，那就失去了寫測試的意義了。

### **4. 容易衝突其他 library**
PowerMock 容易與其他 library 產生衝突，例如某些版本的 mockito, javassist。尤其是大型專案中常會用到很多 library，萬一有衝突時往往會消耗大量成本，在 stackoverflow 上已有許多苦主：

- [java.lang.NoSuchMethodError...](https://stackoverflow.com/a/71977415/5485454)
- [PowerMock throws NoSuchMethodError](https://stackoverflow.com/a/40371375/5485454)

### **5. 可能寫出不好的單元測試**
對於 PowerMock 來說， production code 的實作細節一覽無遺，所以開發者很可能會寫很多 arrange、一大堆的 mock/stub。導致每當程式碼有小改動或重構時，就容易造成測試失敗，讓 test case 難以維護。

### **6. 容易忽略 code design**
因為 PowerMock 如此 powerful，容易使開發者過於依賴與濫用，原因很簡單，**因為無論 production code 再怎麼雜亂無章都能夠寫出單元測試**，久而久之讓人容易忽略 code design。這是我認為它最大的缺點。


如果你對於上面提的幾點有感，覺得 PowerMock 弊大於利，或覺得現階段不適合使用，因而決定棄用，那可以參考以下的方法 — **重構**。
 
## **以重構取代 PowerMock**
重構是為了提高程式碼的可測試性(Testability)，如果可測試性高，可維護(Maintainability)、可讀(Readability)、可理解(Understandability) 性自然而然就提高了，這對專案的健康是有幫助的。但重構已經寫好的程式是有一定風險的，重構前最好是搭配 code review、整合測試、end-to-end 測試等方案來防止重構時意外產生的 bug。

以下是幾個 PowerMock 常見的 use case，我將提供幾個重構方法與思路來取代 PowerMock：

### **Mock Static class/method** 
PowerMock 可以輕易的 mock static，我相信這應該是 PowerMock 受歡迎的理由。

雖然 static 使用方便、效能較快，但也因此常被濫用，造成物件隱含相依、維護困難、不易測試等問題，因此在使用 static 之前應以更嚴苛的標準來檢視。以 `getProperty()` 為例：

```java
// bad, it may cause error when server is shutdown.
public static getProperty(String key) {
  return server.getProperty(key);
}
```
從程式的角度看起來似乎沒問題，但實際上在 Server 尚未啟動時可能產生錯誤或是沒有回傳值。因為這個 method 相依了 Server 環境，所以不適合作為 static method，應該改成 instance method。

良好的 static method 不應該有外部環境依賴，自然就不需要被 mock，因此我不建議 mock static。雖然如此，實務上仍可能有不得不 mock static 的情形。好消息是 Mockito 3.4.0 發布了 `mockStatic()` 功能來模擬 static method，因此就可不需使用 PowerMock。

### **WhiteBox** 
PowerMock.Whitebox 是利用 Reflection 來繞過物件封裝的特性，允許 test case 直接存取物件的 private field, method，例如

```java
String foo = Whitebox.getInternalState(Foo.class, "FIELD_NAME");
String bar = Whitebox.invokeMethod(MyUtil.class, "getStringFromArray", (Object)arrayOfStrings);
```

一般而言，在測試中存取 private 並不是好的做法，但如果非測不可的話，還是有其他出路的。

例如在測試中存取 private field，可以考慮在 production code 加上 package-private getter/setter，專門給單元測試使用；如果想要在 test case 中單獨測試或呼叫 private method，這就表示該 method 也許應該屬於另一個 class，此時可以考慮使用 `Delegate Method` 委派另一個類別。或者更簡單粗暴的方式是使用 Reflection。

### **doNothing**
例如你想要驗證 `getData` 的回傳值，卻不想執行與測試不相干的 private method `processA` 時，可以使用 PowerMock 的 `doNothing()`

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

之所以會想要使用 `doNothing()`，通常是因為程式做了太多事情。可以思考 `getData()` 是否違反 Single Responsibility Principle？ 此時可以考慮使用 `Delegate Method` 委派另一個類別，權責分明，測試自然就好寫。

### **Mock System Class**
假設有一 method `isLate()` 相依了系統時間，所以每次執行測試時可能會有不同的結果，因此需要 mock System.class 來模擬系統時間。

```java
// bad design. hard to test.
public boolean isLate() {
  long now = System.currentTimeMillis();
  return now > 1500000L ? true : false;
}
```

而比較好的做法是：不讓 method 自己去請求現在的時間，而是由 caller 藉由參數傳遞進去，有點像依賴注入 (Dependency Injection, DI) 的觀念，透過 DI 能夠使我們更容易控制輸入端的資料。經過重構後的程式碼，甚至連 mocking framework 都不需要了，如果能不依賴於 framework，通常會是個更好的 practice：

```java
// better
public boolean isLate(long now) {
  return now > 1500000L ? true : false;
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
若寫單元測試時，欲將 `new` 回傳成不同的實作的 instance，這時可以使用 PowerMock 提供的 `whenNew()`：

```java
// In MyClass (SUT)
public void doSomething() {
  Dependency dependency = new Dependency();
  dependency.doSomething();
  // ...
}

@Test
public void whenNew_example() {
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

但其實有更好的替代方案：就是用依賴注入。我們先產生 mocked object，做好初始設定後，再透過 constructor 的參數的方式傳入待測物件。如此一來不僅程式增加了彈性，也可以達到的測試目的。

```java
// better
public MyClass(Dependency dependency) {
  this.dependency = dependency;
}

@Test
public void alternatives_of_whenNew_example() {
 // arrange    
 Dependency mockedDependency = mock(Dependency.class);  
 MyClass sut = new MyClass(mockedDependency);

 // act
 sut.doSomething();
 ...
}
```

## **何時該用 PowerMock**
講了這麼多 PowerMock 的壞處，但 PowerMock 並非一無是處，存在即合理。我認為最適當的應用場景就是重構與測試 legacy code。一個沒有單元測試保護的 legacy code 需要被重構時，通常開發者會先寫一個大範圍的整合測試，接著進行重構，讓 production code 有了基本的可測試性後，再寫單元測試。

但實際上 legacy code 裡面什麼鬼故事都有，例如程式相依時間或外部 API 等難以寫整合測試的場合，或是可能需要用到 test double 時，這時 PowerMock 就派上用場了。承前面所說的：**對於 PowerMock 來說，production code 的實作細節一覽無遺**。因此開發者可以透過上述 PowerMock 的各種 API 去控制待測物件的行為，接著寫好 test case，讓 legacy code 有了基本保護與驗證方法後，就能讓開發者更有信心、大膽的重構。

## **結語**
PowerMock 是個功能強大的單元測試工具，但也不可否認的，若使用不當，容易使開發人員忽略程式碼品質，導致日後花費更多開發與維護成本；若是讓對於測試不熟悉的人使用 PowerMock，反而會使他們不知該如何寫出優秀的測試與程式。

如果測試中充斥著 PowerMock，則表示 production code 的可測試性並不好。因此，考慮到專案未來的發展，我建議開發者在開發時遵循良好的設計原則，避免寫出 bad smell。撰寫單元測試時，若有必要，使用 Mockito 即可。

總之，我認為 PowerMock 只適合用於 legacy code，不應該用於開發中的系統。
 
### **References**
[Modern Mocking Tools and Black Magic](https://martinfowler.com/articles/modernMockingTools.html)

### **更多你可能會感興趣的文章**
- [如何寫出優秀的單元測試 (Best Practice)](/articles/good-unit-test)
- [分析 Spring 的依賴注入模式](/articles/analyzing-dependency-injection-patterns-in-spring)
- [常見的 Interface 錯誤用法](/articles/anti-pattern-of-java-interface-impl-style)