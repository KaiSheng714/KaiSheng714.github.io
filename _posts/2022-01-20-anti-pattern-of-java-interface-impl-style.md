---
layout: post
title: "常見的 Java Interface 錯誤用法"
tagline: ""
author: "Kai-Sheng"
permalink: /articles/anti-pattern-of-java-interface-impl-style
categories: [Java, Design]
image: /assets/image/interface-impl-dir.png
--- 

在 Java 專案中，應該不少人看過或寫過只有一個實作(implementation) 的介面 (interface)，並且以 **interface-impl** 的風格成對存在，如下圖的 FooImpl, BarImpl, ServiceImpl：

![常見的 Interface 錯誤用法](/assets/image/interface-impl-dir.png?margin=vertical-medium)

雖然此寫法可以在任何時候用另一個實作來替換原本的實作，提高程式碼的彈性，但我認為如果當前沒有多個實作，這反而會增加額外的工作量，是一種 **anti-pattern**。

### **問題1. 違反 YAGNI 原則**
很多人會這樣寫，可能是認為

>
> 先寫出 interface，先抽象化，以後就可以替換不同實作，較有彈性。
>

但經驗法則告訴我們，計畫通常都趕不上變化，我們無法預測未來。因此，設計程式時不需為了**未來有可能**使用的理由就事先建立 interface，因為它反而很可能和我們當初所想的不一樣。

如果當前的實作並沒有被替換的可能，那這種 interface 在用法上、在依賴上與 concrete class 是幾乎沒有差異的，**不抽象，也沒有解耦，更沒有多型**，反而成為一種 over design，往往造成後續接手的人更難維護。


延伸閱讀: [軟體設計原則 YAGNI (You aren't gonna need it)](/articles/yagni-principle)

### **問題2. 違反 DRY 原則**
任何人都不喜歡做重複的事、維護重複出現的程式碼，但每當 interface-impl 的其中一者發生改變時，無論是重構或是任何修改，都迫使開發者需要花費額外的成本去同步、維護另一者。

延伸閱讀: [軟體設計原則 DRY (Don't repeat yourself)](/articles/dry-principle) 

### **問題3. 不必要的干擾**
想像一下，專案中有超過 500 個檔案，如果其中包含許多 interface-impl，IDE 會使你無法很流暢地進行 trace code，因為要一層層經過那些不必要的 interface-impl，在龐大的專案裡遇到這種事，將會打擊開發者的工作效率與心理感受。

此外，這也表示檔案的數量是比原本多一倍的，這不僅讓專案**虛胖**，也連帶增加複雜度，因為多了這層非必要的 abstract layer，不僅更不直觀，也可能會隱藏其他的 implementation。
 
## **如何改善它？**
我認為在這種情況下，直接使用 concrete class 即可。開發程式保持簡單直觀是很重要的。可能有些人會認為這樣寫無傷大雅，但我認為大問題往往是從小問題引起的，一旦病入膏肓，就算想改也改不動了。

如果是為了寫**測試**，在 test 裡會有唯一的 implementation 時，我建議可以使用 mocking library 如 [Mocktio](https://site.mockito.org/)，或是利用繼承與 @Override 或 faking 技術在測試中替換實作，如此就不必特地為了單元測試而寫 interface，使專案保持簡潔。

因此，若開發者當下不確定是否需要一個 interface 時，我的建議是：**先不要**。因為仰賴於現代 IDE 的強大，若等到有明確需要時再利用進行 **extract interface** 即可，幾乎無成本。換言之，避免此問題的方法其實很簡單: **延遲決定、需要時再建**。 
 

## **interface 的正確用法**:
先引用一段話:

> 
> Programming to an interface, not an implementation.
> 

意思是，設計時應專注於**程式能提供什麼功能，而不是如何辦到的。**

因此，首先描述你的 interface 能為 client 提供**什麼功能**，例如你有一個提供檔案存取服務的 interface 命名為 **FileService** ，那它的 implementation 應該要描述**如何**存取檔案，例如可能有 DiskService, FtpService, MyMagicService …，而不應該是 FileServiceImpl。

再以 Java 的 `ArrayList`, `LinkedList` 為例，它們都實作了 `List` 介面，各有各的實作細節。透過 interface，我們可以根據應用情境（例如時空複雜度)，用很少的改動成本決定要使用哪個 List，這可以增加程式的彈性，`List` 就是個好的 interface。

在實務上，如果你正在開發的產品或專案並無開放給團隊外部引用，也沒有使用遠端服務(RPC/微服務)，大部分的情況是不需要自行定義 interface 的。

反之，若你開發的是例如 library, SDK 發布給外部，此時就很適合利用 interface 定義出系統邊界，讓外部 client 透過 interface 界接你的專案，並由他們自行開發具體細節。

## **結語**
**並不是寫了 interface 就等於抽象、解耦**，有時候誤用比未用更糟糕，應該根據具體的情況去權衡是否需要 interface。也許你可以檢視你的專案是否有類似的情況，並試著改善它，降低維護成本，提升軟體品質。
### **Reference**
- 本文是 ["常見的 Interface 錯誤用法 - 叡揚資訊"](https://www.gss.com.tw/blog/interface) 的修訂新版，原文作者為我本人
- [Martin Fowler- InterfaceImplementationPair](https://martinfowler.com/bliki/InterfaceImplementationPair.html)
- [Adam Bien — Service s = new ServiceImpl() — Why You Are Doing That?](http://adambien.blog/roller/abien/entry/service_s_new_serviceimpl_why)
- [Do I need to use an interface when only one class will ever implement it?](https://softwareengineering.stackexchange.com/questions/159813/do-i-need-to-use-an-interface-when-only-one-class-will-ever-implement-it/159815#159815)


### **更多你可能會感興趣的文章**
- [軟體設計原則 YAGNI (You aren't gonna need it!)](/articles/yagni-principle)
- [軟體設計原則 DRY (Don't repeat yourself)](/articles/dry-principle)
- [如何提高程式碼的可測試性 (Testability)](/articles/testability)
