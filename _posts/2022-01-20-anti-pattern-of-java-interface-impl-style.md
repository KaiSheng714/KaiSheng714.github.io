---
layout: post
title: "常見的 Java Interface 錯誤用法"
tagline: ""
author: "Kai-Sheng"
permalink: /articles/anti-pattern-of-java-interface-impl-style
categories: [Java, Design]
image: /assets/image/interface-impl-dir.png
--- 

在 Java 專案中，應該不少人看過或寫過只有一個實作(implementation) 的介面 (interface)，並且以 **Interface-Impl** 的風格普遍存在，如下圖的 FooImpl, BarImpl, ServiceImpl，而且通常它們只給團隊內部使用，也不會界接外部服務，我認為這樣設計是個 **anti-pattern**。它會產生幾個問題，本文將探討此寫法的負面影響以及如何改善。 

![常見的 Interface 錯誤用法](/assets/image/interface-impl-dir.png?margin=vertical-medium)

### **問題1. 違反 YAGNI 原則**
很多人會這樣寫，是基於以下理由

>
> 先寫出 interface，先抽象化，以後就可以替換不同實作，較有彈性。
>

經驗法則告訴我們，計畫通常都趕不上變化，我們無法預測未來，導致最終結果可能和當初想的不一樣。因此，設計程式時不需為了**未來有可能**使用的理由就事先建立 interface，因為它反而很有可能不會有第二個實作，等到有明確需求時再實行抽象化也不遲。

如果 interface 沒有第二個實作，換言之，當前的實作並沒有被替換的可能，那這種 interface 在用法上、在依賴上與 concrete class 是沒有差異的，表面上是 interface，本質上是個 duplicated type，**沒有抽象概念，也沒有解耦，更沒有多型**，失去了使用 interface 的初衷與目的，成為一種 over design，往往造成後續接手的人難以維護。

延伸閱讀: [軟體設計原則 YAGNI (You aren't gonna need it)](/articles/yagni-principle)

### **問題2. 違反 DRY 原則**
當你寫出 interface-impl，其中一者發生改變時，無論是重構或是任何程式修改，都迫使你需要花費額外的成本去同步、維護另一者，但我們不應該將同樣的事情再重複做一次，我相信任何人都不喜歡維護重複出現的程式碼。

延伸閱讀: [軟體設計原則 DRY (Don't repeat yourself)](/articles/dry-principle) 

### **問題3. 不必要的干擾**
想像一下，專案中有超過 500 個檔案，如果其中包含許多 interface-impl，當你需要一層一層 trace code 時，IDE 會使你無法很流暢地進行，若在較大的專案裡遇到這種事，將會打擊開發者的工作效率與心理感受。

此外，當 interface-impl 出現時，表示檔案的數量是比原本多一倍的，這不僅讓專案更龐大，也隱含的增加了專案的複雜度，因為多了這層非必要的 abstract layer，讓系統變得更不直觀，也會讓開發者永遠好奇眼前的 interface 是否有其他的 implementation ?
 
## **如何解決？**
我認為這種 interface-impl 不應存在，反而直接了當使用 concrete class 即可。開發程式不需要過度包裝與設計，保持簡單直觀是最重要的。可能有些人會認為專案中即使有一些 interface-impl 也無傷大雅，但我認為大問題往往是從小問題引起的，一旦病入膏肓，就算想改也改不動了。因此，優秀的 clean coder 應盡量維持專案的乾淨與健康。

如果是為了寫**測試**，在 test 裡會有唯一的 implementation 時，我建議可以使用 mocking library 如 [Mocktio](https://site.mockito.org/)，或是利用繼承與 @Override 或 faking 技術在測試中替換實作，如此就不必特地為了單元測試而寫 interface，使專案保持簡潔。

因此，若開發者當下不確定是否需要一個 interface 時，我的建議是：**暫時不要**。因為仰賴於現代 IDE 的強大，若等到有明確需要 interface 時再利用工具進行 extract interface 即可，這幾乎無成本，很容易就能產生一個 interface。換言之，避免此問題的方法其實很簡單: **等待**、**延遲決定**。
 

## **interface 的正確用法**:
先引用一段話:

> 
> Programming to an interface, not an implementation.
> 

意思是，設計時應專注於**程式能提供什麼功能，而不是如何辦到的。**

因此，首先描述你的 interface 能為 client 提供**什麼功能**，例如你有一個提供檔案存取服務的 interface 命名為 **FileService** ，那它的 implementation 應該要描述**如何**存取檔案，例如可能有 DiskService, FtpService, MyMagicService …，而不應該是 FileServiceImpl。

再以 Java 的 `ArrayList`, `LinkedList` 為例，它們都實作了 `List` 介面，各有各的實作細節。透過 interface，我們可以根據應用情境（例如效能、時間複雜度等)，用很少的改動成本決定要使用哪個 List，這可以增加程式的彈性。好的 interface 就應如此。

再者，如果你的專案並無開放給團隊外引用，或者不是為了界接外部服務，通常是不需要自行定義 interface 的。反之，若你開發的是例如 library, SDK 發布給外部 client 開發使用，此時就很適合利用 interface 定義出系統邊界，讓外部 client 透過 interface 界接你的專案，並由他們自行開發具體細節。

## **結語**
本文描述了許多人對於 java interface 的誤用，導致這種只有一個實作的介面 (interface-impl) 成對出現在許多專案中，**並不是寫了 interface 就等於抽象、解耦**，誤用比未用更糟糕，**如果不需要，就不應該盲目的寫。**
 
也許你可以檢視你的專案是否有類似的情況，並試著讓專案更乾淨、直觀、明確，降低維護成本，提升軟體品質。

### **Reference**
- 本文是 ["常見的 Interface 錯誤用法 - 叡揚資訊"](https://www.gss.com.tw/blog/interface) 的修訂新版，原文作者為我本人
- [Martin Fowler- InterfaceImplementationPair](https://martinfowler.com/bliki/InterfaceImplementationPair.html)
- [Adam Bien — Service s = new ServiceImpl() — Why You Are Doing That?](http://adambien.blog/roller/abien/entry/service_s_new_serviceimpl_why)
- [Do I need to use an interface when only one class will ever implement it?](https://softwareengineering.stackexchange.com/questions/159813/do-i-need-to-use-an-interface-when-only-one-class-will-ever-implement-it/159815#159815)


### **更多你可能會感興趣的文章**
- [軟體設計原則 YAGNI (You aren't gonna need it!)](/articles/yagni-principle)
- [軟體設計原則 DRY (Don't repeat yourself)](/articles/dry-principle)
- [如何提高程式碼的可測試性 (Testability)](/articles/testability)
