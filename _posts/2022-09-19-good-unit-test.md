---
layout: post
title: "如何寫出優秀的單元測試"
author: "Kai-Sheng"
permalink: /articles/good-unit-test
categories: [Design]
image: /assets/image/cover4.png
--- 

我看過許多新進同仁寫的單元測試看起來好像有那麼一回事，但實際上好像又沒測到重點。如果這樣的測試持續增長，單元測試不僅沒有帶來好處，反而還讓專案變得更不可靠。雖然寫出優秀的單元測試是一件不容易的事，優秀的開發者不可不知。

 
想像一下這樣的場景：

> 新人小明的第一個任務是新增一個簡單的功能，也許只需要10幾行程式。
> 但是當他完成後，自動測試系統卻回報了大量的測試失敗。
> 本應很快完成的工作卻花了小明好幾天。扼殺了他的工作效率，也挫傷了他的士氣。

前人寫的這些單元測試產生了反效果
- 不準確：這次的更改並沒有加入真正的 bug，但是測試卻失敗了。
- 不清楚：小明很難確定哪出了問題、如何修復，以及這些測試最初應該做什麽。



## **Best Practice**

0. 一個測試案例只驗證一個行為
0. 第1種驗證方式(回傳值)所佔的比例應要最高
0. 測試案例之間無相依性
0. 測試案例的命名盡量清楚、口語化
0. 測試案例不具備邏輯
0. 驗證時，不過度指定  (over specification)
0. 不過度依賴 mock framework
0. 進行互動驗證時，只驗證會改變外部的行為



## **後記**
 

### **References**
- [Software Engineering at Google: Lessons Learned from Programming Over Time](https://www.amazon.com/Software-Engineering-Google-Lessons-Programming/dp/1492082791)