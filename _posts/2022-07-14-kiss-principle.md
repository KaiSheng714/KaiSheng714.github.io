---
layout: post
title: "軟體設計原則 KISS （Keep it simple, stupid）"
author: "Kai-Sheng"
permalink: /articles/kiss-principle
categories: [Design]
image: /assets/image/cover3.png
--- 

**KISS （Keep it simple, stupid）**，是敏捷開發的核心設計原則之一，意思是盡可能將程式、實作手法保持簡單、單純。KISS 原則也可以運用在很多地方，像是產品設計、學術理論、期貨交易...等等，是一個很重要且實用的原則。一言以蔽之就是「大道至簡」。

## **意義**
首先引用 Martin Fowler 的名言：
> Any fool can write code that a computer can understand. Good programmers write code that humans can understand.
> 任何白痴都能寫出電腦看得懂的程式。好的程式設計師則會寫出人類看得懂的程式。

對於大多數人而言，讀程式、寫程式的時間比例通常約 8:2，一位原本沒看過這段程式的人要來修 bug 或是加新功能時，這段程式應該要能被看得懂。而 KISS 原則的目的是讓的程式碼變得簡單、容易理解。

程式碼的可讀性、可維護性是衡量程式碼品質的標準。若程式碼夠簡單，也就意味著很容易讀懂，bug 比較難隱藏，即便出現 bug，修復起來也比較簡單、成本較低。能將其意圖清楚地傳達給讀者的程式碼，才能稱作好的程式碼。

但並不是每個專案都能夠很好被理解，例如要做一套繪圖軟體，開發團隊可能需要具備相關背景知識，如計算機圖學、線性代數、甚至人工智慧等等。而這種的複雜性是問題本質上的複雜，並不是 KISS 原則主要探討的。KISS 原則探討的是在**開發系統的過程中，問題的解決方案應保持簡單**，意即「將功能與規格單純化」，以及「將實作手法單純化」，例如系統的商業邏輯、流程、機制、快取策略等。

KISS 原則說起來簡單，做起來卻很難，接下來我將介紹一些實際的作法。

## **一些 KISS 原則的實際做法**
- 如果需要在兩種解決方案之間進行選擇，選擇最簡單的那個
- 盡量將程式碼寫得平鋪直敘，寫出看起來不怎樣的程式碼
- 盡量不要使用同事可能不懂的寫法、技術，開發時也要考慮到**人的因素**
- 不要自己造輪子，要善於使用已經有的 library
- 先不要急著過度優化效能，除非真的遇到效能瓶頸 (Premature Optimization Is the Root of All Evil)
- 將過於複雜的 if 條件抽成一個命名良好的 method
- 盡可能減少 method 的循環複雜度（Cyclomatic complexity）
- 減少物件的內部狀態，過多狀態往往會帶來更多複雜性
- 少用繼承，多用組合 （Composition over Inheritance）
- 避免過多的抽象層
- 時常去簡化專案 code base（重構、單元測試）
- code review。如果在 code review 時，同事對你的程式碼有很多疑問，那就說明你的程式碼可能不夠簡單

## **後記**
- 用複雜、高深技巧解決問題的開發者，往往不是最厲害的；越是能用簡單的方法解決複雜的問題，越能體現一個人的能力。
- 人類的腦容量有限，無法同時處理太多問題。如果在專案中加入不必要的複雜性，這將導致程式碼可讀性下降，從而降低其他同事的理解、推理能力；更重要的是，這種負面影響不是線性的，而是指數性成長的。因此避免增加不必要的複雜性，遵循 KISS 原則是很重要的。簡單永遠是個值得的投資。

### **References**
- [KISS原則](https://zh.wikipedia.org/zh-tw/KISS%E5%8E%9F%E5%88%99)
- [kiss-revisited]（https://enterprisecraftsmanship.com/posts/kiss-revisited/）
- [[翻譯]為何資深開發者寫的程式看起來不怎樣，又如何從千里之外認出菜鳥]（https://medium.com/@CQD/%E7%82%BA%E4%BD%95%E8%B3%87%E6%B7%B1%E9%96%8B%E7%99%BC%E8%80%85%E5%AF%AB%E7%9A%84%E7%A8%8B%E5%BC%8F%E7%9C%8B%E8%B5%B7%E4%BE%86%E4%B8%8D%E6%80%8E%E6%A8%A3-%E5%8F%88%E5%A6%82%E4%BD%95%E5%BE%9E%E5%8D%83%E9%87%8C%E4%B9%8B%E5%A4%96%E8%AA%8D%E5%87%BA%E8%8F%9C%E9%B3%A5-c1afa754c5e4）
 