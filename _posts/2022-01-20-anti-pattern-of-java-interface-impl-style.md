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
很多人會寫 interface-impl 的原因是根據 SOLID 原則之一的依賴反轉原則 (Dependency Inversion Principle)，認為程式應依賴抽象介面，所以他們事先寫出許多 interface-impl、提早抽象化，以後就可以替換不同實作，較有彈性。但如果當前的實作並沒有被替換的可能，那這種 interface 在用法上、在依賴上與具體類別是幾乎沒有差異的，像這種 interface 反而成為了一種過度設計，往往造成後續接手的人更難維護、寫更多程式。

### **問題2. 違反 DRY 原則**
任何人都不喜歡做重複的事、維護重複出現的程式碼，但每當 interface-impl 的其中一者發生改變時，無論是重構、改名或是任何修改，都迫使開發者需要花費額外的成本去同步、維護另一者。

### **問題3. 不必要的干擾**
想像一下，專案中有超過 500 個檔案，如果其中包含許多 interface-impl，IDE 會使你無法很流暢地進行 trace code，在龐大的專案裡遇到這種事，將會打擊開發者的工作效率與心理感受。

此外，這也表示檔案的數量是比原本多一倍的，這不僅讓專案**虛胖**，也連帶增加複雜度、變得更不直觀。
 
## **如何改善？**
我認為在這種情況下，**直接使用 concrete class 即可**。開發程式保持簡單直觀是很重要的，可能有些人會認為這樣寫無傷大雅，但我認為大問題往往是從小問題引起的，一旦病入膏肓，就算想改也改不動，最終成為歷史共業。

如果是為了寫測試，我建議可以使用 mocking library，或是利用繼承與 @Override 或 faking 技術在測試中替換實作，如此就不必特地寫 interface，使專案保持簡潔。

因此，若開發者當下不確定是否需要一個 interface 時，我的建議是：**先不要**。仰賴於現代的 IDE 之強大，若等到有明確需要時再做 **extract interface** 即可，幾乎無成本。換言之，改善此問題的方法其實很簡單: **延遲決定、需要時再建**。 

## **interface 的建議用法**
開發者應該根據 client 的需求進行設計 interface，接著再讓 client 自行實作出功能。白話文就是： 我完全不知道、也不在乎 client 怎麼實作，但只要求 client 能符合我所設計的規格(interface)即可。

例如開發 library, SDK, framework 此類會發布給外部的專案，就很適合利用 interface 定義出**系統邊界**，讓外部 client 透過 interface 整合該專案。

意思是，設計時應專注於**程式能提供什麼功能，而不是如何辦到的。**

因此，首先描述你的 interface 能為 client 提供**什麼功能**，例如你有一個提供檔案存取服務的 interface 命名為 **FileService** ，那它的 implementation 應該要描述**如何**存取檔案，例如可能有 DiskService, FtpService, MyMagicService …，而不應該是 FileServiceImpl。

再以 Java 的 `ArrayList`, `LinkedList` 為例，它們都實作了 `List` 介面，各有各的實作細節。透過 interface，我們可以根據應用情境（例如效能、時間複雜度等)，用很少的改動成本決定要使用哪個 List，這可以增加程式的彈性。好的 interface 就應如此。

再者，如果你的專案並無開放給團隊外引用，或者不是為了界接外部服務，通常是不需要自行定義 interface 的。反之，若你開發的是例如 library, SDK 發布給外部 client 開發使用，此時就很適合利用 interface 定義出系統邊界，讓外部 client 透過 interface 界接你的專案，並由他們自行開發具體細節。

## **結語**
如果程式有很具體的行為，且不會有多個不同實作，那就不要寫 interface。雖然經典的設計原則鼓勵依賴抽象介面，但依賴具體類別也並不是錯。開發者應該權衡是否需要 interface，不要過度設計。

### **Reference**
- 本文是 ["常見的 Interface 錯誤用法 - 叡揚資訊"](https://www.gss.com.tw/blog/interface) 的修訂新版，原文作者為我本人
- [Martin Fowler- InterfaceImplementationPair](https://martinfowler.com/bliki/InterfaceImplementationPair.html)
- [Adam Bien — Service s = new ServiceImpl() — Why You Are Doing That?](http://adambien.blog/roller/abien/entry/service_s_new_serviceimpl_why)
- [Do I need to use an interface when only one class will ever implement it?](https://softwareengineering.stackexchange.com/questions/159813/do-i-need-to-use-an-interface-when-only-one-class-will-ever-implement-it/159815#159815)

### **更多你可能會感興趣的文章**
- [軟體設計原則 YAGNI (You aren't gonna need it!)](/articles/yagni-principle)
- [軟體設計原則 DRY (Don't repeat yourself)](/articles/dry-principle)
- [如何提高程式碼的可測試性 (Testability)](/articles/testability)
