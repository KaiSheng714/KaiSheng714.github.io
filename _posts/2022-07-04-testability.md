---
layout: post
title: "如何提高程式碼的可測試性 (Testability)"
author: "Kai-Sheng"
permalink: /articles/testability
categories: [Design, Unit-testing]
image: /assets/image/site-image-small.png
--- 
 眾所皆知，寫單元測試有非常多好處，但有些主管會問，為什麼寫測試會讓工程師額外花這麼多時間？除了因為缺乏單元測試技術知識外，根本原因是產品程式碼的可測試性太低，導致工程師在撰寫測試時難以將精力放在正確的地方，甚至放棄撰寫測試。要寫出優秀的單元測試有一定的難度和門檻，關鍵在於如何提高程式碼的可測試性。

## **什麼是程式碼的可測試性?**

程式碼的可測試性的定義有多種說法，比較著名的是微軟的測試架構師 Dave Catlett 提出的 SOCK Model。雖然各家的定義有所不同，但概念大致相同，即：軟體在給定的測試環境下，能夠輕易進行測試的難易程度，或者說需要投入的測試成本多寡。

我們可以明顯地看出，程式碼的可測試性與設計品質高度相關。例如，低內聚高耦合、充滿 code smell 的程式通常難以測試。因此，提高程式碼的可測試性能讓工程師撰寫更有用、更有價值的測試案例，有助於在開發過程中快速發現並修正錯誤。

可測試性高的程式通常具備以下幾個關鍵特質：

### **設計簡潔**
如果待測程式專注於執行單一任務，它需要被測試的內容和情境也相對簡單。此外，可讀性和表達能力也會更好。因此，我們應該追求 **簡潔、簡單、不複雜（遵循 KISS 原則）** 的程式設計。相反的，複雜且複雜度過高的程式難以測試。例如，過長的方法往往過於複雜，我建議將方法保持短小，行數限制在30行以內。

### **容易初始化**
這意味著在單元測試中能夠輕鬆初始化待測程式，也就是能夠輕鬆地創建它的實例。這種情況下，待測程式的相依物件較少，或者能夠輕鬆地隔離外部相依性。如果初始化困難，這可能表示設計結構較差。容易初始化是撰寫單元測試的第一步，如果連這一點都難以實現，那麼就無法享受撰寫單元測試所帶來的好處。

### **易於控制輸入資料**
這意味著我們在單元測試中能夠輕鬆模擬不同的測試情境。舉個例子，某銀行系統在客戶提款超過1億元時會對銀行內發出警告通知。在單元測試中，我們可以控制客戶的提款金額，以模擬客戶大額提款的情境，而不必事先在銀行帳戶準備一筆鉅款。在 Java 的測試中，通常會使用 mocking 技術來模擬各種情境，以提高程式的可控程度。如果程式能夠被有效控制，就能夠進行重複執行測試，並將其納入 CI（持續整合）流程中，實現自動化測試的目標。
 
### **易於驗證輸出結果**
程式對於任何操作都應該產生預期的輸出。輸出的形式通常是可預期的回傳值、內部狀態或外部行為。無論以何種方式表達，都應該具有可驗證性，也就是有跡可循，這樣才容易進行驗證。如果能夠輕鬆驗證程式結果是否符合預期，就能降低測試的難度，例如僅使用 `assertEqual()`、`verify()` 等簡單的驗證方法就能得出測試結果。如果程式的輸出結果難以驗證，就表示它不容易進行測試。
 
## **程式碼可測試性的重要性**
軟體的可測試性高，通常意味著程式碼的品質也高，這樣製作出來的產品才會好用。

軟體的可測試性高，有助於早期發現和預防問題。如果能越早發現問題，解決問題的成本就越低，我們就能更快地交付產品給客戶，並快速獲得客戶的反饋，符合敏捷開發的原則。

如果軟體的可測試性低，工程師就需要花費更多時間寫測試，而這些測試可能會雜亂無章，效果不佳。**如果撰寫測試變成一種損失，工程師就不會願意寫測試，或者會拖延到很晚才開始寫，這違反了測試左移的原則和單元測試的初衷**。
## **如何提高程式碼的可測試性**

透過以下幾種方法，可以提高程式碼的可測試性，使得軟體的品質更高，並且更容易維護。

### **Single Responsibility Principle (SRP)**

單一職責意味著每個 class 應該有一個且只有一個職責（或被改變的理由），也意味著內聚力較高，這些都使得單元測試更容易。
 
若一個 class / method 同時實作了好幾個功能，這會大大提升測試的難度，因為功能多，依賴就會變多，而且需要驗證的結果變多，導致測試變得複雜。但實踐 SRP 並不如想像中的簡單，關鍵在於怎麼控制顆粒度，若切太細會創造出冗餘的 class；切得太大就變得不太單純。所以要怎麼適當的分配職責，還是得依實際情況而定。

以剛才某銀行系統的例子，提款功能應屬於 `WithdrawService` 的職責，發出內部警示功能應屬於 `NotifyService` 的職責，兩者各司其職、權責分明、彼此獨立。

### **依賴注入 (Dependency Injection)**

定義: **由外部提供待測物件所需的依賴，而待測物件不必自己建立它們。**

這是一種可以大幅提升可測試性的重要手段，降低了物件之間的耦合度，也避免 `new` 關鍵字與重要的業務邏輯混雜在一起。藉由外部注入相依性物件，提升了待測程式的可控制性，我們就可以輕鬆的建立測試替身並注入到待測程式中。

一般而言，有3種 dependency injection pattern，而我建議使用 constructor 注入依賴模式。延伸閱讀: [分析 Spring 的依賴注入模式](https://kaisheng714.github.io/articles/analyzing-dependency-injection-patterns-in-spring)


### **移除 Bad Smell**

容易影響可測試性的 bad smell 有： **Long Parameter List, Divergent Change, Long Method, Large Class...**

解決方式之一就是團隊要時常 code review，而且團隊成員需要熟悉**重構手法**。如果不重構，除了難以測試外，久而久之就會變成技術債。
 
### **YAGNI 原則**
工程師應該在面臨確鑿的需求時，才實作相應的功能。例如某團隊成員在 method 中增加了 **現在用不到、未來有可能用到** 的 if 分支，請問我們現在該不該測它呢？

延伸閱讀: [軟體設計原則 YAGNI (You aren't gonna need it!)](/articles/yagni-principle)

### **Constructor 中不包含任何邏輯**
Constructor 應該只專注於初始化，而不會有任何邏輯。若 constructor 不僅初始化、if-else，還呼叫 API、查詢 DB，這就使得初始化成本提高，難以隔絕外部依賴，可測試性就降低了。

此外，在單元測試中想要改寫或是 mock constructor 是不容易的，雖然現在有些測試框架可以替換 constructor 的行為，但通常不建議使用。延伸閱讀: [不建議使用 PowerMock 的理由](https://ithelp.ithome.com.tw/articles/10284923) 

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

這個觀點有個但書，如果是 `Strings.isBlank()`, `Math.abs()` 這種簡單、內部沒有共享狀態、沒有與外部環境相依的 static method，我認為並不會造成難以測試的問題。

但有時若是真的不得已，Mockito 3.4 版也提供了 `Mockito.mockStatic`，讓我們可以在單元測試中替換 static 的行為，代價就是測試程式會變得比較複雜、執行測試的時間也會更久。


### **Test-Driven Development (TDD)**

TDD 是一種**先從使用者角度寫測試，再回頭撰寫產品程式碼的開發手法**。因為 TDD 讓開發者換位思考，從使用者的角度出發，換個位子就換了腦袋，就更容易了解到該怎麼設計才能讓 class / API 更易用。為了先寫出測試，開發者就必須先去思考如何進行測試，他不僅了解需求，還要逐步解構需求成一個個單純、小的 test case。若熟練 TDD 技術，就能大幅提高軟體可測試性。

更重要的是，使用 TDD ，能讓開發者更專注於產品的使用行為，而非程式的實作細節 (因為根本就還沒有細節)，因此更容易設計出強健的單元測試。一旦有強健的單元測試，產品隨著時間不斷演進，開發者將會更勇於重構，進而重構出更好的設計，**好的設計都是不斷重構而來的**
 

## **結論**
- 本文解釋了程式碼的可測試性及其重要性，並介紹了許多實務上可以提高可測試性的方法。
- 若工程師覺得單元測試很難寫，原因通常不是不會寫測試，而是產品程式的可測試性太低。
- 若整個團隊的觀念、基本功如果沒有到位，也不懂如何提高程式的可測試性，則要在組織內推動單元測試是很困難的。
- 提高產品程式的可測試性，較容易寫出優秀的測試程式，接著才能享受自動化測試帶來的甜美果實。
 
### **References**
- [Patterns in Practice: Design For Testability - Microsoft Docs](https://docs.microsoft.com/zh-tw/archive/msdn-magazine/2008/december/patterns-in-practice-design-for-testability)
- [如何提升軟體的可測試性-搞笑談軟工](https://teddy-chen-tw.blogspot.com/2013/04/blog-post_4.html)
- [程式碼的可測性](https://www.ithome.com.tw/voice/88062)
- [Effective Unit Testing by Eliotte Rusty Harold](https://www.youtube.com/watch?v=fr1E9aVnBxw)
- [The Clean Code Talks -- Unit Testing](https://www.youtube.com/watch?v=wEhu57pih5w)

### **更多你可能會感興趣的文章**
- [如何寫出優秀的單元測試 (Best Practice)](/articles/good-unit-test)
- [常見的 Interface 錯誤用法](/articles/anti-pattern-of-java-interface-impl-style)