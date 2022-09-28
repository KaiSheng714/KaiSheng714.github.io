---
layout: post
title: "如何提高程式碼的可測試性 (Testability)"
author: "Kai-Sheng"
permalink: /articles/testability
categories: [Design, Unit-testing]
image: /assets/image/cover4.png
--- 
 
眾所皆知，寫單元測試有非常多好處，但有些主管會問，為什麼寫測試會讓工程師額外花這麼多時間？除了本身對單元測試技術不熟悉以外，追根究柢是因為產品程式碼的可測試性太低，導致工程師寫測試時很難將精力投入在對的地方，甚至導致放棄寫單元測試。要寫出優秀的單元測試有一定的難度與門檻，關鍵就在於工程師要思考如何提高程式碼的可測試性，進而讓寫單元測試變得簡單。本文將介紹提高 Java 單元測試可測試性的教學。


## **什麼是程式碼的可測試性?**

程式碼的可測試性的定義有多種說法，比較著名的如微軟的測試架構師 Dave Catlett 提出的 SOCK Model。雖然各家定義不同，但概念幾乎都圍繞在同一件事，就是**指一個軟體在一給定的測試環境下，能夠被測試的難易程度，或是投入測試成本的多寡**。

我們不難察覺到**程式的可測試性與設計品質是高度相關的**，例如低內聚高耦合、充滿臭味（code smell）的程式通常都難以測試，因此若能提高可測試性，讓工程師們能撰寫更多有用、高價值的 test case，就容易在開發過程中快速發現並消除 bug。

可測試性高的程式通常具備幾個關鍵特質：

### **設計單純**
若待測程式只專注做一件事，則它需要被測試的事情、情境也相對單純，且可讀性、表達力也會較佳。因此將程式設計得越**單純、簡單、沒有不必要的複雜（KISS 原則）**越好。反之，越是複雜、不夠單純的程式就越難測試。複雜的程式例如過於冗長的 method，我的建議是 method 不可以過長，行數應限制在 30 行以內才算合格。

### **容易初始化**
意即在單元測試中可以很容易的初始化待測程式，換句話說，能輕易在測試中 `new` 出來。這種情況下，通常待測程式的相依物件不多，或是能很輕易的隔絕外部依賴。若難以初始化，則表示它可能是個 **God Object**。容易初始化是撰寫單元測試的第一步，如果連這點都很難做到，那就得不到寫單元測試帶來的好處了。

### **輸入資料容易被控制**
這意味著我們在單元測試中能夠輕易的模擬出不同的測試情境。舉個例子：某銀行系統在客戶提款超過1億元時會對銀行內發出警告通知。這時我們可以在單元測試中控制顧客的提款金額，以便模擬顧客大額提款的情境，而不必真的事先在銀行帳戶準備一筆鉅款。在 Java 的測試中，實務上會透過 mock 技術來模擬各種情境以提高程式的可控制程度。如果程式能有效被控制住，就能被不斷的重複執行測試，接著加入 CI （Continuous Integration） 的流程中，就能達到自動化測試的境界。
 
### **輸出結果容易被驗證**
程式對於任何一項操作都要能產生預期的輸出，而輸出的表現形式通常是可預期的回傳值、內部狀態、外在行為，不管是以什麼方式表現，至少都要**有跡可循**，才容易被驗證。若能容易的驗證程式結果是否符合預期，就能降低測試的難度，例如只用 `assertEqual()`, `verify()` 等簡單驗證方法就能得出測試結果。如果程式的輸出結果不容易被驗證，就代表不容易被測試。

----- 

## **程式碼可測試性的重要**
軟體可測試性高，通常就意味著程式碼的品質也高，做出來的產品才會好用。

軟體可測試性高，就容易測試左移。測試左移本質上是要儘早發現、預防問題。若能越早發現問題，解決問題的成本就越便宜，我們就能更快速的交付產品給客戶、更快速得到客戶的 feedback，更符合敏捷開發的精神。

若軟體可測試性低，工程師就要花很多時間寫測試，通常在這種情況所寫的單元測試也會是一團亂，不僅測試效果不佳，也不容易測到關鍵、容易 test fail，導致測試品質低落，例如在測試中 mock 過多依賴、stubbing，會讓測試變得很複雜，而測試程式也是需要注重品質的。**若寫測試像是一門賠本生意，就會讓工程師不願意寫測試，或者拖到很晚才開始寫測試，這些都違背了測試左移的原則以及單元測試的初衷**。

----- 

## **如何提高程式碼的可測試性**

### **Single Responsibility Principle (SRP)**

單一職責意味著每個 class 應該有一個且只有一個職責（或被改變的理由），也意味著內聚力較高，這些都使得單元測試更容易。
 
若一個 class / method 同時實作了好幾個功能，這會大大提升測試的難度，因為功能多，依賴就會變多，而且需要驗證的結果變多，導致測試變得複雜。但實踐 SRP 並不如想像中的簡單，關鍵在於怎麼控制顆粒度，若切太細會創造出冗餘的 class；切得太大就變得不太單純。所以要怎麼適當的分配職責，還是得依實際情況而定。

以剛才某銀行系統的例子，提款功能應屬於 `WithdrawService` 的職責，發出內部警示功能應屬於 `NotifyService` 的職責，兩者各司其職、權責分明、彼此獨立。

### **依賴注入 (Dependency Injection)**

定義: **由外部提供待測物件所需的依賴，而待測物件不必自己建立它們。**

這是一種可以大幅提升可測試性的重要手段，降低了物件之間的耦合度，也避免 `new` 關鍵字與重要的業務邏輯混雜在一起。藉由外部注入相依性物件，提升了待測程式的可控制性，我們就可以輕鬆的建立測試替身並注入到待測程式中。

一般而言，有3種 dependency injection pattern，而我建議使用透過 constructor 注入依賴的模式。延伸閱讀: [分析 Spring 的依賴注入模式](/articles/analyzing-dependency-injection-patterns-in-spring)


### **移除 Bad Smell**

容易影響可測試性的 bad smell 有： **Long Parameter List, Divergent Change, Long Method, Large Class...**

解決方式之一就是團隊要時常 code review，而且團隊成員需要熟悉**重構手法**。如果不重構，除了難以測試外，久而久之就會變成技術債。
 
### **YAGNI 原則**
工程師應該在面臨確鑿的需求時，才實作相應的功能。例如某團隊成員在 method 中增加了**現在用不到、未來有可能用到**的 if 分支，請問我們現在該不該測它呢？

延伸閱讀: [軟體設計原則 YAGNI (You aren't gonna need it!)](https://kaisheng714.github.io/articles/yagni-principle)

### **Constructor 中不包含任何邏輯**
Constructor 應該只專注於初始化，而不會有任何邏輯。若 constructor 不僅初始化、if-else，還呼叫 API、查詢 DB，這就使得初始化成本提高，難以隔絕外部依賴，可測試性就降低了。

此外，在單元測試中想要改寫或是 mock constructor 是不容易的，雖然現在有些測試框架可以替換 constructor 的行為，但通常不建議使用。延伸閱讀: [不建議使用 PowerMock 的理由](/articles/drawback-of-powermock) 

一個好的 constructor 應該是 **沒有任何邏輯，只做依賴注入與狀態初始化**:

```java
public BankService(WithdrawService withdrawService, NotifyService notifyServic, ...) {
  this.withdrawService = withdrawService;
  this.notifyService = notifyService;
  ...
}
```
 
### **減少使用 Singleton / static**
要在測試中替換一個 static method 是困難的。此外，濫用 Singleton Pattern 容易產生難以維護的 global state。例如:

```java
DbManager.getConnection().doSomething();
Calendar.getInstance();
```

雖然 Singleton / static 很方便，但它無形之中也帶來了提高耦合度的問題，這兩者都是造成不好測的原因，它有可能讓我們難以立即發現問題，畢竟單元測試的本質就是**探討如何隔離外部相依**。有時候要完全避免使用 static 方法可能蠻難的，如果可以，那就盡量減少使用頻率。

這個觀點有個例外，如果是 `Strings.isBlank()`, `Math.abs()` 這種簡單、沒有與外部環境相依的 static method，我認為並不會造成難以測試的問題。

但有時若是真的不得已，Mockito 3.4 版也提供了 `Mockito.mockStatic`，讓我們可以在單元測試中替換 static 的行為，代價就是測試程式會變得比較複雜、執行測試的時間也會提高。

### **Test-Driven Development (TDD)**

TDD 是一種**先從使用者角度寫測試，再回頭撰寫產品程式碼的開發手法**。因為 TDD 讓開發者換位思考，從使用者的角度出發，換個位子就換了腦袋，就更容易了解到該怎麼設計才能讓 class / API 更易用。為了先寫出測試，開發者就必須先去思考如何進行測試，他不僅了解需求，還要逐步解構需求成一個個單純、小的 test case。若熟練 TDD 技術，就能大幅提高軟體可測試性。

上手 TDD 並不容易，其實以上討論的議題都包含在 TDD 的領域之中，之後我會再寫一篇文章專門討論 TDD。

-----

## **結論**
- 本文解釋了程式碼的可測試性及其重要性，並介紹了許多實務上可以提高可測試性的方法。
- 若工程師覺得單元測試很難寫，原因通常不是不會寫測試，而是產品程式的可測試性太低。
- 若整個團隊的觀念、基本功如果沒有到位，也不懂如何提高程式的可測試性，則要在組織內推動單元測試是很困難的。
- 提高產品程式的可測試性，較容易寫出優秀的測試程式，接著才能享受自動化測試帶來的甜美果實。

### **References**
- [Software testability](https://en.wikipedia.org/wiki/Software_testability)
- [Patterns in Practice: Design For Testability - Microsoft Docs](https://docs.microsoft.com/zh-tw/archive/msdn-magazine/2008/december/patterns-in-practice-design-for-testability)
- [如何提升軟體的可測試性-搞笑談軟工](https://teddy-chen-tw.blogspot.com/2013/04/blog-post_4.html)
- [程式碼的可測性](https://www.ithome.com.tw/voice/88062)
- [Effective Unit Testing by Eliotte Rusty Harold](https://www.youtube.com/watch?v=fr1E9aVnBxw)
- [The Clean Code Talks -- Unit Testing](https://www.youtube.com/watch?v=wEhu57pih5w)


## **更多你可能會感興趣的文章**
- [如何寫出優秀的單元測試 (Best Practice)](/articles/good-unit-test)
- [常見的 Interface 錯誤用法](/articles/anti-pattern-of-java-interface-impl-style)