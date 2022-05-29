---
layout: post
title: "軟體設計原則 KISS (Keep it simple, stupid)"
author: "Kai-Sheng"
permalink: /articles/kiss-principle
categories: [Design]
--- 

**KISS (Keep it simple, stupid)**，是敏捷開發的核心設計原則之一， KISS 原則算是一個萬金油類型的設計原則，可以應用在很多場景中。它不僅經常用來指導軟體開發，還經常用來指導更加廣泛的系統設計、產品設計等，比如，冰箱、建築、iPhone 手機的設計等等。不過，咱們的專欄是講代碼設計的，所以，接下來，我還是重點講解如何在編碼開發中應用這條原則。
意思是「保持盡可能簡單，簡單到笨蛋也能理解」。KISS 原則可以運用在很多地方，像是產品設計、簡報設計、學術理論、期貨交易...等等。



![kiss-principle](/assets/image/kiss-principle.png?size=full)



Quote by Martin Fowler:

Any fool can write code that a computer can understand. Good programmers write code that humans can understand.

任何白痴都能寫出電腦看得懂的程式。好的程式設計師則會寫出人類看得懂的程式。


簡單設計是敏捷開發中非常重要的一項實踐，但是這條原則說起來簡單卻做起來難。因為每個程式設計師其實都是一個有完美主義的藝術家，所做軟體其實都是一件自己的藝術品，同時受到許多關於設計方面的資料的影響，所以在做設計的時候會情不自禁的加上許多「優雅特性」和「靈活性」

此原則的原意是應將設計單純化，以簡化設備維護。將此涵義套用於軟體開發時，則變為 “將功能與規格單純化”，以及 “將實作手法單純化”。尤其是關於實作手法，須注意避免將目的與手段搞錯。所謂程式開發原則和規範、程式碼的美觀與一致性、設計方式等，應該是提升程式碼可讀性與強健性用的手段，但將其本身作為目的時，反而會傷害程式碼的可讀性與強健性。例如：在 reactive programming 設計中，可將包含常數值的所有數值設為 observable，但這種作法雖然在一致性的觀點上 “美化”程式碼，但相對地卻降低了程式碼的可讀性與提高了除錯的難度。
 
  

本身就復雜的問題，用復雜的方法解決，並不違背 KISS 原則.
 
 
https://www.yinxiang.com/everhub/note/3f7637f0-5d0b-4188-b398-b0b7e13c2b9b
https://time.geekbang.org/column/article/179607

https://enterprisecraftsmanship.com/posts/kiss-revisited/

https://medium.com/@CQD/%E7%82%BA%E4%BD%95%E8%B3%87%E6%B7%B1%E9%96%8B%E7%99%BC%E8%80%85%E5%AF%AB%E7%9A%84%E7%A8%8B%E5%BC%8F%E7%9C%8B%E8%B5%B7%E4%BE%86%E4%B8%8D%E6%80%8E%E6%A8%A3-%E5%8F%88%E5%A6%82%E4%BD%95%E5%BE%9E%E5%8D%83%E9%87%8C%E4%B9%8B%E5%A4%96%E8%AA%8D%E5%87%BA%E8%8F%9C%E9%B3%A5-c1afa754c5e4