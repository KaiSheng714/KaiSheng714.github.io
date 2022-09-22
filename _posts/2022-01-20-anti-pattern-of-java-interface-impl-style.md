---
layout: post
title: "常見的 Interface 錯誤用法"
tagline: ""
author: "Kai-Sheng"
permalink: /articles/anti-pattern-of-java-interface-impl-style
categories: [Java, Design]
image: /assets/image/cover0.png
--- 

在 Java 專案中，應該不少人看過或寫過只有一個實作(implementation) 的介面 (interface)，並且以 **Interface-Impl** 的風格普遍存在，如下圖的 FooImpl, BarImpl, ServiceImpl，
而且它們通常都放在同一個 package 或 module 裡，也只給內部人員使用，並不會當作 library 提供給其它專案參考。我認為 interface-impl 的設計是個 **anti-pattern**。它會產生幾個問題，本文將探討此寫法的負面影響以及如何改善。 

![常見的 Interface 錯誤用法](/assets/image/interface-impl-dir.png?margin=vertical-medium)

### **1. 違反 YAGNI 原則**
很多人會這樣寫，是基於以下理由

>
> 先寫出 interface，之後可以替換不同實作，較有彈性。
>

我們無法預測未來，設計程式時不需為了**未來有可能**使用的理由就事先建立 interface，因為它反而很有可能不會有第二個實作。如果 interface 沒有第二個實作，換言之，當前的實作並沒有被替換的可能，那這種 interface 在用法上、在依賴上與 concrete class 是沒有差異的，表面上是 interface，本質上是個 duplicated type，並不是 interface 該提供的價值，**沒有抽象概念，也沒有解耦**，也失去了使用 interface 的初衷與目的，成為一種 over design。

延伸閱讀: [軟體設計原則 YAGNI (You aren't gonna need it)](/articles/yagni-principle)

### **2. 違反 DRY 原則**

當你寫出 interface-impl，其中一者發生改變時，無論是重構或是任何程式修改，都迫使你需要花費額外的成本去同步、維護另一者，但我們不應該將同樣的事情再重複做一次。這不僅是程式的重複，也是知識上的重複，違反了 DRY 原則。

延伸閱讀: [軟體設計原則 DRY (Don't repeat yourself)](/articles/dry-principle) 

### **3. 不必要的干擾**

想像一下，專案中有超過 500 個檔案，如果其中包含許多 interface-impl，當你需要一層一層 trace code 時，IDE 會使你無法很流暢地進行，若在較大的專案裡遇到這種事，將會打擊開發者的工作效率與心理感受。

此外，當 interface-impl 出現時，表示檔案的數量是比原本多一倍的，這不僅讓專案更龐大，也隱含的增加了專案的複雜度，因為多了這層非必要的 abstract layer，讓系統變得更不直觀，也會讓開發者永遠好奇眼前的 interface 是否有其他的 implementation ?

------

## **How To Fix It ?**

我認為這種 **interface-impl 不應存在**，反而使用一般的 concrete class 即可，開發程式不需要過度包裝與設計，保持簡單直觀是最重要的。可能有些人會認為專案中即使有一些 interface-impl 也無傷大雅，但我認為大問題往往是從小問題引起的，一旦病入膏肓，就算想改也改不動了。因此，稱職的 clean coder 應盡量維持專案的乾淨與健康。

如果是為了寫單元測試，在 test 裡會有唯一的 implementation 時，我建議可以使用 mock library，如 [Mocktio](https://site.mockito.org/)，或是利用繼承與 @Override 或 faking 技術在測試中替換實作，如此就不必特地為了單元測試而寫 interface，使專案保持簡潔。

因此，若開發者當下不確定是否需要一個 interface 時，我的建議是：**暫時不要**。因為仰賴於現代 IDE 的強大，若等到有明確需要 interface 時再利用工具進行 extract interface 即可，這幾乎無成本，很容易就能生出一個 interface。換言之，避免此問題的方法其實很簡單: **等待**、**延遲決定**。
 

## **關於 interface 的正確用法**:

先引用一段話:

> 
> Programming to an interface, not an implementation
> 

意思是，設計時應專注於**程式能提供什麼功能，而不是如何辦到的。**

因此，首先描述你的 interface 能為 client 提供**什麼功能**，例如你有一個提供檔案存取服務的 interface 命名為 **FileService** ，那它的 implementation 應該要描述**如何**存取檔案，例如可能有 DiskService, FtpService, MyMagicService …，而不應該是 FileServiceImpl。

再者，如果你的專案並沒有開放給團隊外使用，例如企業級應用程式，在實務上大部分的情況下是不需要 interface 的。反觀，若你開發的是例如 library, SDK 給外部 client 開發使用，此時就很適合利用 interface 定義出系統邊界，讓外部 client 透過 interface 界接你的作品，並由他們自行開發具體細節。

## **結語**

本文描述了許多人對於 java interface 的誤用，導致這種只有一個實作的介面 (interface-impl) 成對出現在許多專案中，這並沒有利用 interface 的優點，若開發者盲目遵循錯誤的設計原則，而沒有理解 interface 實際的涵義與價值，將導致許多負面影響。**並不是只要有 interface 就等於抽象、解耦**，誤用比未用更糟糕。
 
也許你可以檢視你的專案是否有類似的情況，試著讓專案更乾淨、直觀、明確，降低維護成本。

### **Reference**

- [Martin Fowler- InterfaceImplementationPair](https://martinfowler.com/bliki/InterfaceImplementationPair.html)
- [Adam Bien — Service s = new ServiceImpl() — Why You Are Doing That?](http://adambien.blog/roller/abien/entry/service_s_new_serviceimpl_why)
- [Do I need to use an interface when only one class will ever implement it?](https://softwareengineering.stackexchange.com/questions/159813/do-i-need-to-use-an-interface-when-only-one-class-will-ever-implement-it/159815#159815)
