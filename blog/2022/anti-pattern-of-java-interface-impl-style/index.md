

·8 min read

常見的 Interface 錯誤用法
==================

![常見的 Interface 錯誤用法](https://miro.medium.com/max/1400/0*_o1uujfIgv4JMHpJ)Photo by [Glenn Carstens-Peters](https://unsplash.com/@glenncarstenspeters?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)

**TL;DR 如果 class 不需要或不具備抽象化的條件，就不做抽象化。**

在 java 專案中，相信很多人曾看過或寫過`只有一個實作(implementation) 的介面 (interface)`，並且以 **Interface-Impl** 的風格命名，如圖:

![常見的 Interface 錯誤用法](https://miro.medium.com/max/896/1*AHXZgS9Zc8Ly73Tk3jYo4Q.jpeg)java interface-impl style

上圖中的 FooService, BarService 只有一個實作，而且它們都放在同一個 package 或 module 裡，通常只給專案或產品內部使用，並不會當作 library 提供給其它專案參考。不曉得大家有沒有思考過為什麼要這樣寫 ?

據我觀察，可能是有些人將 C++ 的標頭檔 .h 和 實作檔 .cpp 結構的習慣帶進來 ；也有可能是拜 naming convention 或常見的 practice 所賜，習慣成自然；又或者是考量到未來的擴充性，所以就事先定義好 interface；又或者是許多文章提到：物件之間應該盡量依賴於抽象而不是實作。又或者是：”大家都這樣寫，但我不知道為什麼，所以就照樣寫”。

但是，像這樣**只有一個實作的介面** (本文簡稱為 **interface-impl**) ，其實會對程式品質與開發帶來負面影響。

Anti Pattern
============

我認為 **interface-impl** 這樣的設計是個 Anti Pattern。它會產生幾個問題:

1\. 違反 YAGNI 原則
---------------

人性使然，人們在面對未知事物時傾向於未雨綢繆，因為能為自己帶來確定感和安心感，但在設計程式時也許要有不同思維。

如果 interface-impl 成對出現時，表示已經違反 [YAGNI 原則](https://en.wikipedia.org/wiki/You_aren%27t_gonna_need_it)，原因是，設計程式時不需為了**未來有可能**使用的理由就事先建立 interface，因為它反而很有可能不會有第二個實作。因此只要程式設計得剛好、不會難以維護，不需要考慮太遙遠的未來，就是個好的設計。

如果 interface 沒有第二個實作，換言之，**實作並沒有被替換的可能**，那這種 interface 在用法上、在依賴上與 concrete class 是沒有差異的，**表面**上是 interface，本質上是個 duplicated type，並不是 interface 該提供的價值，**沒有抽象概念，更沒有解耦**，也失去了使用 interface 的初衷與目的。

2\. 違反 DRY 原則
-------------

當你定義出 interface-impl 時，當其中一者發生改變時，無論是重構或是任何程式修改，都迫使你需要花費額外的成本去同步另一者，否則 compile 會失敗，但我們不應該將同樣的事情再重複做一次 [(DRY 原則)](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself)。如果事先建立 interface 的理由是”以後”會用到，則開發者會不斷地面臨這個問題 (cost of carry)

此外，以 java 的 List 以及常見的兩個實作 ArrayList, LinkedList 為例:

![常見的 Interface 錯誤用法](https://miro.medium.com/max/836/0*iKFqT32yPICHOy01.png)java list interface

這兩個 List 具有不同的實作與特性，透過 interface，我們可以用很少的改動成本，再根據應用情境（例如效能、時間空間複雜度等因素)決定要使用 ArrayList 或 LinkedList ，如此也增加了程式的彈性，一個好的 interface 就應如此設計。

但 JDK 中肯定**沒有 ListImpl、ArrayListImpl、LinkedListImpl**，對吧 ? 因此，我認為 interface-impl 這種設計，不僅沒有講到重點，反而只是換個方式再將 interface 重複講一遍而已 (tautology)。

想像一下，如果一個 interface-impl 真的有第二個 implementation 出現時，你該如何為它命名呢? -impl2 ? 再者，要如何在 java doc 裡清楚地描述 interface 與 -impl，或是只能重複地描述呢?

3\. 不必要的干擾
----------

想像一下，專案中有超過 500 個檔案，如果其中包含許多 interface-impl，當你需要一層一層 trace code 時，IDE 會使你無法很流暢地進行，在如此龐大的系統裡遇到這種事，有害開發者的工作效率與心理。

此外，當 interface-impl 出現時，表示檔案的數量是比原本多一倍的，這不僅讓專案更龐大，也隱含的增加了專案的複雜度，因為多了這層非必要的 abstract layer，讓系統變得更不直觀，也會讓開發者永遠好奇眼前的 interface 是否有其他的 implementation ?

How To Fix It ?
===============

我認為這種 **interface-impl 不應存在**，反而使用一般的 concrete class 即可，開發程式不需要過度包裝與設計，Keep it simple, straightforword。可能有些人會認為專案中即使有一些 **interface-impl** 也無傷大雅，**但我認為大問題往往是從小問題引起的**，一旦病入膏肓，就算想改也改不動了。因此，稱職的 clean coder 應盡量維持專案的乾淨與健康。

如果是為了寫單元測試，也就是說在 production 裡有一對 interface 與 implementation，而在 test 裡會有另一個 implementation 時，我建議可以使用 mock library，如 java 的 [Mocktio](https://site.mockito.org/)，如此就不必特地為了單元測試而建立新的實作，以致於可以刪除這個 interface，使專案保持簡潔。

因此，若開發者當下不確定是否需要一個 interface 時，我的建議是：**暫時不要**。因為仰賴於現代 IDE 的強大，若等到有明確需要一個 interface 時再進行 extract interface，只要滑鼠點幾下就可以達成，幾乎無成本，隨時都可以 extract interface，下圖是 IntelliJ 的 extract interface：

![常見的 Interface 錯誤用法](https://miro.medium.com/max/1400/0*Pf8uNoMzyWkN6buL.png)extract interface by intelliJ

可能有些人會問:

_很多文章或書上介紹 pattern 或 architecture時，常會看到它的 class diagram 有出現 interface 和單一的 impl 阿。_

但我認為它的表達方式也許只是**教學、範例**用途而已，並不是真的要你也一起 impl，另外也要考量自身需求，不需硬套用某個 pattern 或 architecture，造成過度設計。

如果你遇到 **“大家都這樣寫，但我不知道為什麼，所以我就照樣寫”** 的情況或團隊，也許可以試著打破墨守成規的思想，你可以將想法提出來和大家溝通交流，經過評估與分析，也許能改掉這種不好的寫作風格。

**關於 interface 的正確用法**，在我簡單舉例前，先引用一段話:

> _Programming to an interface, not an implementation_

意思是，設計時應專注於**程式能提供什麼功能，而不是如何辦到的。**

因此，首先描述你的 interface 能提供`什麼`功能，例如你有一個提供檔案存取服務的 interface 命名為 **FileService** ，那它的 implementation 應該要描述`如何`存取檔案，例如可能有 DiskService, FtpService, MyMagicService …，而不應該是 FileServiceImpl。

結語
--

本文描述了許多人對於 java interface 的誤用，導致這種`只有一個實作的介面 (interface-impl)` 成對出現在許多專案中，這並沒有利用 interface 的優點，若開發者只為了盲目遵循錯誤的設計原則，而沒有理解 interface 實際的涵義與價值，將導致許多負面影響。**並不是只要有 interface 就等於抽象、解耦**，誤用比未用更糟糕。

因此，當你有個 interface 時，它應該會提供給許多 client 實作，並且能清楚描述實作的細節，開發者也可以輕易的替換不同實作；反之，則表示開發者可能誤用了 interface，或者是根本不需要 interface。也許你可以檢視你的專案是否有類似的情況，試著讓專案更乾淨、簡潔、明確，提高可讀性。

**Ref**

[Martin Fowler- InterfaceImplementationPair](https://martinfowler.com/bliki/InterfaceImplementationPair.html)

[Adam Bien — Service s = new ServiceImpl() — Why You Are Doing That?](http://adambien.blog/roller/abien/entry/service_s_new_serviceimpl_why)

[Ted Kaminski — Why an interface with only one implementation?](https://www.tedinski.com/2018/07/31/interfaces-cutting-dependencies.html)

[Jérôme Loisel — IMPL CLASSES ARE EVIL](https://octoperf.com/blog/2016/10/27/impl-classes-are-evil/)

[Do I need to use an interface when only one class will ever implement it?](https://softwareengineering.stackexchange.com/questions/159813/do-i-need-to-use-an-interface-when-only-one-class-will-ever-implement-it/159815#159815)
