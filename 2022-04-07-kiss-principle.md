---
layout: post
title: "軟體設計原則 KISS (Keep it simple, stupid)"
author: "Kai-Sheng"
permalink: /articles/kiss-principle
categories: [Design]
--- 

**KISS (Keep it simple, stupid)**，是敏捷開發的核心設計原則， KISS 原則算是一個萬金油類型的設計原則，可以應用在很多場景中。它不僅經常用來指導軟體開發，還經常用來指導更加廣泛的系統設計、產品設計等，比如，冰箱、建築、iPhone 手機的設計等等。不過，咱們的專欄是講代碼設計的，所以，接下來，我還是重點講解如何在編碼開發中應用這條原則。
意思是「保持盡可能簡單，簡單到笨蛋也能理解」。KISS 原則可以運用在很多地方，像是產品設計、簡報設計、學術理論、期貨交易...等等。

![kiss-principle](/assets/image/kiss-principle.png)


簡單設計是敏捷開發中非常重要的一項實踐，但是這條原則說起來簡單卻做起來難。因為每個程式設計師其實都是一個有完美主義的藝術家，所做軟體其實都是一件自己的藝術品，同時受到許多關於設計方面的資料的影響，所以在做設計的時候會情不自禁的加上許多「優雅特性」和「靈活性」

此原則的原意是應將設計單純化，以簡化設備維護。將此涵義套用於軟體開發時，則變為 “將功能與規格單純化”，以及 “將實作手法單純化”。尤其是關於實作手法，須注意避免將目的與手段搞錯。所謂程式開發原則和規範、程式碼的美觀與一致性、設計方式等，應該是提升程式碼可讀性與強健性用的手段，但將其本身作為目的時，反而會傷害程式碼的可讀性與強健性。例如：在 reactive programming 設計中，可將包含常數值的所有數值設為 observable，但這種作法雖然在一致性的觀點上 “美化”程式碼，但相對地卻降低了程式碼的可讀性與提高了除錯的難度。
 
 
Keep it short and simple
Keep it simple and straightforward

本身就復雜的問題，用復雜的方法解決，並不違背 KISS 原則.
 
https://www.yinxiang.com/everhub/note/3f7637f0-5d0b-4188-b398-b0b7e13c2b9b
https://time.geekbang.org/column/article/179607

https://enterprisecraftsmanship.com/posts/kiss-revisited/