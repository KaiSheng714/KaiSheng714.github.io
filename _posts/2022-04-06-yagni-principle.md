---
layout: post
title: "談談 YAGNI 設計原則"
tagline: "設計得剛好就好"
author: "Kai-Sheng"
permalink: /blog/yagni-principle.html
categories: [Design]
--- 


**YAGNI (You aren't gonna need it!)**，是一個在敏捷開發中常見的設計原則。此原則指出，程式開發者應該在面臨確鑿的需求時，才實作相應的功能。換言之，就是不要實作那些現在用不到的東西，也不要因為未來可能需要的理由而事先實作。

以下是違反 YAGNI 原則的經典案例:

>
> 這程式我先註解起來，之後可能會再用到。
>
> 這段程式我先抽出成 util，之後可以與別人共用。
>
> 這個 function 先多開幾個參數，之後比較有彈性、容易 reuse。
>
> 這個 class 我先把它抽象化，之後可以替換實作時可以不用改，較有彈性。
>
> 我覺得這個功能，user 之後可能會提出來，雖然現在還用不到，所以我事先做出來。
>

 -----

![YAGNI](/assets/image/yagni.png)

-----

上述為了未來而預先做的設計，有很大的機率事與願違，或是畫虎不成反類犬。

我們不是算命仙，不能預測未來，在這個瞬息萬變的時代，計劃往往趕不上變化。所以，我們更應該多花精力去解決當前的問題，專注於 user 目前的需求，盡快的交付產品給 user，讓我們更快的得到 feedback；而不是為了應對未來的各種變化，做出畫蛇添足的設計。

也許有些人會質疑: 
> 這些設計雖然現在沒用到，但留著也沒什麼不好。

除非此專案日後不再作任何維護，否則 **每行程式碼、每個設計都是有成本的**，包含:
- 開發成本
- 修改成本
- 測試成本
- 維護成本
- 增加理解程式碼的成本
- 編譯、執行成本
- ...等有形與無形的成本

因此，除了不要實作那些現在用不到的程式碼，我們也要學習斷捨離的精神，盡量刪除那些用不到的程式碼，保持整個專案的乾淨 (Clean Code)。

## **如何不違反 YAGNI 原則**

如何不違反 YAGNI 原則，換言之，就是如何做到剛剛好就可以、如何不過度設計。
首先要解決眼前的問題、設計要足夠符合 user 的需求，並套用 KISS 原則，之後就可以做簡單的自我檢視 :

**這種設計能不能輕易的被改變 ?**

如果可以，就表示該設計能夠輕易地應付未來的不確定性，那就是個剛好的設計，符合敏捷開發的精神。


## **什麼時候可以違反 YAGNI 原則**

如果你開發的專案是要給其他人使用的 API、Library，或是 DB schema 時，就需要多花一點心力考慮未來可能會發生的狀況、未來可能會用到的功能，因為這種情況很容易牽一髮而動全身。以一個擁有千萬用戶的 library 為例，他們無法每次都為了更新 library 而讓他們的 APP 也得重新修改、compile、上架。這種情況下違反 YAGNI 原則是必須的，而 SOLID 之一的開放封閉原則 **Open-Closed Principle (OCP)** 就顯得特別重要了。
 

### **References**
- [https://en.wikipedia.org/wiki/You_aren%27t_gonna_need_it](https://en.wikipedia.org/wiki/You_aren%27t_gonna_need_it)
- [https://enterprisecraftsmanship.com/posts/yagni-revisited/](https://enterprisecraftsmanship.com/posts/yagni-revisited/)
