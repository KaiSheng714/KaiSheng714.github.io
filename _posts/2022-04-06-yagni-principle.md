---
layout: post
title: "軟體設計原則 YAGNI (You aren't gonna need it!)"
tagline: ""
author: "Kai-Sheng"
permalink: /articles/yagni-principle
categories: [Design]
image: /assets/image/yagni.png
--- 


**YAGNI (You aren't gonna need it!)**，是敏捷開發的核心設計原則之一。此原則指出，程式開發者應該在面臨確鑿的需求時，才實作相應的功能。換言之，就是不要實作那些現在用不到的東西，也不要因為未來可能需要的理由而事先實作。這樣做的目的是為了避免浪費時間和資源去開發最終可能永遠不會使用到的功能，同時也能使軟體的設計變得更加簡潔和易於維護。


![YAGNI](/assets/image/yagni.png?size=full)



以下是違反 YAGNI 原則的經典案例:

>
> 我先把這段程式註解起來，也許之後會用到。
>
> 我先將這段程式碼抽出成 Util 類，或許以後可以與其他人共用。
>
> 這個 function 先加入額外的參數，這樣以後可以更有彈性和易於重複使用。
>
> 我先將這個類別抽象化，這樣在替換實作時不需要修改太多程式碼，更具彈性。
>
> 我覺得這個功能將來可能會被要求，雖然目前還不需要，但我先預先做出來。
>

上述的預先設計，很有可能最終與實際情況背道而馳，或是達不到預期的結果。

作為開發者，我們並非算命仙，無法預測未來。在這個瞬息萬變的時代，計劃往往追不上變化。因此，我們應該更多地專注於解決當前的問題，專注於用戶目前的需求，儘快將產品交付給用戶，以便我們能夠更快地獲得反饋。我們不應該為了應對未來的各種變化而做出多餘的設計，造成更多問題。

也許有些人會質疑: 
> 這些設計雖然目前用不到，但保留著也沒有什麼壞處。

除非這個專案日後不再維護，否則每行程式碼、每個設計都是有成本的，包括：
- 開發成本
- 修改成本
- 測試成本
- 維護成本
- 閱讀、理解程式碼的成本
- 編譯、執行成本
- 其他有形和無形的成本

總之，違反 YAGNI 原則會增加程式的複雜度和修改難度，同時降低可測試性。因此，除了不要實作目前用不到的程式碼，我們也應該學會拋棄不需要的程式碼，保持整個專案的整潔。遵循此原則不僅有助於團隊加快開發速度，還能使程式碼保持簡單，更易於維護。

## **如何遵守 YAGNI 原則**
 
首先，要明確你要解決的問題是什麼。在滿足客戶需求的同時，以"簡單"為出發點，自我檢視：
 
**我要寫的這個 class/method/function，現在真的需要嗎? 可以真正的解決問題嗎?**

如果答案是否定的，那麼不要急著著手開發，而是停下來重新思考。

## **什麼情況下可以違反 YAGNI 原則**
如果你正在開發的專案是讓其他人使用的 API、library，那麼你需要花更多心思考慮未來可能發生的情況和可能需要的功能，因為這種情況下的變動往往造成牽一髮而動全身。以擁有千萬用戶的 library 為例，他們無法每次更新都要求 client 修改、編譯和重新上架。在這種情況下，預先的設計是必要的，此時開放封閉原則（Open-Closed Principle，OCP）在這裡就顯得重要。

## **後記**
The best code is no code. 不要實作目前用不到的程式碼，沒有什麼比**不寫程式碼**更好了。

### **References**
- [https://en.wikipedia.org/wiki/You_aren%27t_gonna_need_it](https://en.wikipedia.org/wiki/You_aren%27t_gonna_need_it)
- [https://enterprisecraftsmanship.com/posts/yagni-revisited/](https://enterprisecraftsmanship.com/posts/yagni-revisited/)

### **更多你可能會感興趣的文章**
- [如何寫出優秀的單元測試 (Best Practice)](/articles/good-unit-test)
- [常見的 Interface 錯誤用法](/articles/anti-pattern-of-java-interface-impl-style)