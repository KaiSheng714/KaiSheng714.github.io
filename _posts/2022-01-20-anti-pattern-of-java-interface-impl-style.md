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
很多人會寫 interface-impl 的原因是根據經典的設計原則，認為程式應依賴抽象介面，所以他們事先寫出許多 interface-impl、提早抽象化，以便於未來替換不同實作，增加程式彈性。

但事實上，如果當前的實作並沒有被替換的可能，那這種 interface 在用法上、在依賴上與具體類別是幾乎沒有差異的，這個 interface 反而成為了一種過度設計，往往造成後續接手的人更難維護、寫更多程式碼。

### **問題2. 違反 DRY 原則**
每當 interface-impl 的其中一者發生改變時，無論是重構、改名或是任何修改，都迫使開發者需要花費額外的成本去同步、維護另一者。我相信任何人都不喜歡做重複的事、維護重複出現的程式碼。

### **問題3. 不必要的干擾**
在較大的專案中如果出現許多 interface-impl，IDE 會使你無法很流暢地進行 trace code，因為 IDE 會不斷詢問要導向 interface 或 implementation，這將會打擊開發者的工作效率與心理感受。

此外，成對出現的 interface-impl 表示檔案數量是比原本多一倍的，這不僅讓專案虛胖，也連帶增加複雜度、變得更不直觀。
 
## **如何改善？**
我認為在這種情況下，直接使用具體類別即可，開發程式保持簡單直觀是很重要的。可能有些人會認為這樣寫無傷大雅，但我認為大問題往往是從小問題引起的，一旦病入膏肓，就算想改也改不動，最終成為**歷史共業**。

如果是為了寫測試，我建議可以使用 mocking library，或是利用繼承與 @Override 或 faking 技術在測試中替換實作，如此就不必特地寫 interface，使專案保持簡潔。

因此，若開發者當下不確定是否需要一個 interface 時，我的建議是：先不要。仰賴於現代強大的 IDE，若等到有明確需要時再做 **extract interface** 即可，幾乎無成本。換言之，改善此問題的方法其實很簡單: **延遲決定**。 

## **interface 的建議用法**
開發者應該根據 client 的需求進行設計 interface，接著再讓 client 自行實作出功能。白話文就是：我完全不知道、也不在乎 client 怎麼實作，但只要求 client 能符合我所設計的規格(interface)即可。

例如開發 library, SDK, framework 此類會發布給外部的專案，就很適合利用 interface 定義出**系統邊界**，讓外部 client 透過 interface 整合該專案。

另一方面，在實務上，如果專案並無開放給團隊外部引用（例如企業應用程式），或是不使用微服務、ORM 等等一定要寫 interface 的 framework，則真正需要 interface 的情況其實並不多。

## **結語**
如果程式在設計時就有很具體的行為，而且目前也不會有多個不同實作，那就可以不必寫 interface 了。雖然經典的設計原則鼓勵程式之間應依賴抽象介面，但依賴具體類別也並不是錯，因此開發者應該權衡是否需要 interface，並且不要過度設計。

### **Reference**
- 本文是 ["常見的 Interface 錯誤用法 - 叡揚資訊"](https://www.gss.com.tw/blog/interface) 的修訂新版，原文作者為我本人
- [Martin Fowler- InterfaceImplementationPair](https://martinfowler.com/bliki/InterfaceImplementationPair.html)
- [Adam Bien — Service s = new ServiceImpl() — Why You Are Doing That?](http://adambien.blog/roller/abien/entry/service_s_new_serviceimpl_why)
- [Do I need to use an interface when only one class will ever implement it?](https://softwareengineering.stackexchange.com/questions/159813/do-i-need-to-use-an-interface-when-only-one-class-will-ever-implement-it/159815#159815)

### **更多你可能會感興趣的文章**
- [軟體設計原則 YAGNI (You aren't gonna need it!)](/articles/yagni-principle)
- [軟體設計原則 DRY (Don't repeat yourself)](/articles/dry-principle)
- [如何提高程式碼的可測試性 (Testability)](/articles/testability)
