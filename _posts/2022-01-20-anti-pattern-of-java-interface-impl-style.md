---
layout: post
title: "常見的 Java Interface 錯誤用法"
tagline: ""
author: "Kai-Sheng"
permalink: /articles/anti-pattern-of-java-interface-impl-style
categories: [Java, Design]
image: /assets/image/interface-impl-dir.png
--- 

在 Java 專案中，應該不少人看過或寫過只有一個實作（implementation）的介面（interface），並且以 **interface-impl** 的風格成對出現，如下圖的 FooImpl, BarImpl, ServiceImpl：

![常見的 Interface 錯誤用法](/assets/image/interface-impl-dir.png?margin=vertical-medium)
雖然此寫法可以允許用另一個實作來替換原本的實作，提高程式碼的彈性，但我認為，如果目前不存在多個實作類別，此寫法反而會增加額外的工作量，是一種過度設計。

## **問題**
很多人會這麼寫的原因是根據經典的設計原則，認為程式應依賴抽象介面，於是他們事先寫出許多 interface、抽象化，以便於未來替換不同實作方式。然而，現實世界中充滿了變數，專案隨著時間推移，許多過早預想、太彈性的設計，最終可能與當初想的不一樣。（YAGNI 原則）

再者，每當 interface-impl 之一發生改變時，都需要額外的工作量來同步另一方。在開發中，我們應該減少重複的工作（DRY 原則）

此外，在較大的專案中，如果存在許多 interface-impl，IDE 可能會妨礙進行 trace code，這也會降低開發者的工作效率與心理感受，而成對出現的 interface-impl，表示檔案數量是比原本多一倍的，讓專案虛胖，變得不那麼直觀。
 
## **如何改善？**
我認為這種情況下，直接使用具體類別是合理的。有些人可能會覺得這樣寫也無傷大雅，但大問題通常源於小問題，最終很可能成為令眾人束手無策的**歷史共業**。

如果需要為單元測試而使用 interface，我建議可以使用模擬（mocking）函式庫，如 Mockito，或者利用繼承、@Override等技術在測試中替換具體實作，這樣就不需要為了增加可測試性而特別建立 interface。

因此，我建議開發者在不確定是否需要 interface 時，先別急著建。因為在現代 IDE 的幫助下，可以隨時進行「extract interface」，幾乎沒有額外的成本。因此，改善這個問題的方法很簡單：延遲決定，並通過反饋和迭代，調整和改進設計。**好的設計總是重構來的**。

## **interface 的建議用法**
一般而言，使用 interface 的目的是實現多型，其中一種常見的用法是透過屬性檔例如 application.yaml，實現在 runtime 時根據不同環境選擇使用不同的 implementation。

對於開發函式庫、SDK 等需要提供給外部專案使用的情況，也很適合運用 interface 去定義**系統邊界**，讓外部 client 透過 interface 來整合與使用。說白了，就是寫給別人實作，而不是自己實作自己的 interface。

另一方面，在實務上，若專案程式碼不對外開放（例如企業應用程式），或者不使用必須寫 interface 的框架（如微服務、ORM等），則需要使用 interface 的情況可能較少。

## **結語**
雖然 interface 可以提高程式碼的靈活性和可維護性，如果程式在設計時就有很具體的行為，而且目前也不會有不同實作，那其實可不必寫 interface。

雖然經典的設計原則鼓勵程式之間應依賴抽象介面，但依賴具體類別也並不是個錯，因此我建議開發者應根據專案的情況，權衡是否需要 interface。

### **Reference**
- 本文是 ["常見的 Interface 錯誤用法 - 叡揚資訊"](https://www.gss.com.tw/blog/interface) 的修訂新版，原文作者為我本人
- [Martin Fowler- InterfaceImplementationPair](https://martinfowler.com/bliki/InterfaceImplementationPair.html)
- [Adam Bien — Service s = new ServiceImpl() — Why You Are Doing That?](http://adambien.blog/roller/abien/entry/service_s_new_serviceimpl_why)
- [Do I need to use an interface when only one class will ever implement it?](https://softwareengineering.stackexchange.com/questions/159813/do-i-need-to-use-an-interface-when-only-one-class-will-ever-implement-it/159815#159815)

### **更多你可能會感興趣的文章**
- [軟體設計原則 YAGNI (You aren't gonna need it!)](/articles/yagni-principle)
- [軟體設計原則 DRY (Don't repeat yourself)](/articles/dry-principle)
- [如何提高程式碼的可測試性 (Testability)](/articles/testability)
