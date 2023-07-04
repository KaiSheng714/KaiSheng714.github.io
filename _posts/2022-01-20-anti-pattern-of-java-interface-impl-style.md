---
layout: post
title: "常見的 Java Interface 錯誤用法"
tagline: ""
author: "Kai-Sheng"
permalink: /articles/anti-pattern-of-java-interface-impl-style
categories: [Java, Design]
image: /assets/image/interface-impl-dir.png
--- 

在 Java 專案中，應該不少人看過或寫過只有一個實作(implementation) 的介面 (interface)，並且以 **interface-impl** 的風格成對出現，如下圖的 FooImpl, BarImpl, ServiceImpl：

![常見的 Interface 錯誤用法](/assets/image/interface-impl-dir.png?margin=vertical-medium)
雖然此寫法可以在任何時候用另一個實作來替換原本的實作，提高程式碼的彈性。但我認為如果當前沒有多個實作，這反而會增加額外的工作量，是一種過度設計，可能導致下列幾個問題:

### **問題1. 違反 YAGNI 原則**
很多人會寫 interface-impl 的原因是根據經典的設計原則或設計模式，認為程式應依賴抽象介面，所以他們事先寫出許多 interface-impl、提早抽象化，以便於未來替換不同實作，增加程式彈性。

但事實上，如果當前的實作並沒有被替換的可能，此時這個 interface 在用法上、在依賴上與具體類別是幾乎沒有差異的，反而成為了一種過度設計，往往造成後續接手的人更難維護、寫更多程式碼。

### **問題2. 違反 DRY 原則**
每當 interface-impl 之一發生改變時，無論是新增功能、重構、改變名稱或其他修改，都需要額外的工作量來同步另一方。在開發中，我們應該減少重複的工作，特別是重複維護程式碼。

### **問題3. 不必要的干擾**
在較大的專案中如果出現許多 interface-impl，IDE 會使你無法很流暢地進行 trace code，因為 IDE 會不斷詢問要導向至何者，這也會降低開發者的工作效率與心理感受。

此外，成對出現的 interface-impl 表示檔案數量是比原本多一倍的，這不僅讓專案虛胖，也連帶增加複雜度、變得更不直觀。
 
## **如何改善？**
我認為這種情況下，直接使用具體類別是合理的，因為保持程式碼的簡單直觀非常重要。有些人可能會覺得這樣寫也無傷大雅，但大問題通常源於小問題，最終很可能成為**歷史共業**。

如果需要為單元測試而使用 interface，我建議可以使用模擬（mocking）函式庫，或者利用繼承、@Override或模擬（faking）技術在測試中替換具體實作，這樣就不需要特別寫 interface，同時也能保持專案的簡潔性。

因此，我建議開發者在不確定是否需要 interface 時，可以先暫時不要。在現代強大的整合開發環境（IDE）的幫助下，可以在確定需要時隨時進行「extract interface」，幾乎沒有額外的成本。因此，改善這個問題的方法很簡單：延遲決定，並通過持續反饋和迭代，及時調整和改進設計。

## **interface 的建議用法**
一般而言，使用 interface 的目的是實現多型，以提高程式碼的靈活和可維護性。其中一種常見的用法是透過 property，在 runtime 時根據不同環境動態選擇使用哪種 implementation。

對於開發函式庫、SDK 等需要提供給外部專案使用的情況，也很適合運用 interface 定義**系統邊界**，這個方式可以讓外部 client 透過 interface 來整合與使用。只要在設計時專注於提供規格，且不必在乎 client 如何實作，只要求他們符合 interface 的規範即可。

另一方面，在實務上，若專案程式碼不對外開放（例如企業應用程式），或者不使用必須寫 interface 的框架（如微服務、ORM等），則需要使用 interface 的情況可能較少。

## **結語**
雖然 interface 可以提高程式碼的靈活性和可維護性，如果程式在設計時就有很具體的行為，而且目前也不會有不同實作，那其實可不必寫 interface。

雖然經典的設計原則鼓勵程式之間應依賴抽象介面，但依賴具體類別也並不是錯，因此我建議開發者應根據專需求，權衡是否需要 interface，並且遵循最佳實踐。

### **Reference**
- 本文是 ["常見的 Interface 錯誤用法 - 叡揚資訊"](https://www.gss.com.tw/blog/interface) 的修訂新版，原文作者為我本人
- [Martin Fowler- InterfaceImplementationPair](https://martinfowler.com/bliki/InterfaceImplementationPair.html)
- [Adam Bien — Service s = new ServiceImpl() — Why You Are Doing That?](http://adambien.blog/roller/abien/entry/service_s_new_serviceimpl_why)
- [Do I need to use an interface when only one class will ever implement it?](https://softwareengineering.stackexchange.com/questions/159813/do-i-need-to-use-an-interface-when-only-one-class-will-ever-implement-it/159815#159815)

### **更多你可能會感興趣的文章**
- [軟體設計原則 YAGNI (You aren't gonna need it!)](/articles/yagni-principle)
- [軟體設計原則 DRY (Don't repeat yourself)](/articles/dry-principle)
- [如何提高程式碼的可測試性 (Testability)](/articles/testability)
