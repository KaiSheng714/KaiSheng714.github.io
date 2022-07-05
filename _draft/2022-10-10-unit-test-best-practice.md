---
layout: post
title: "單元測試 Best Practice"
author: "Kai-Sheng"
permalink: /articles/unit-test-best-practice
categories: [Unit-testing]
image: /assets/image/unit-test-best-practice.png
--- 
  
許多工程師專注於寫出優秀的產品程式碼，卻忽略了單元測試也同樣需要注重品質，才能將單元測試的優點發揮到最大。其實有許多技巧可以達到。
 
----- 



## **從哪裡開始加入單元測試 ?**
-  新功能、新 code
Legacy code 
    -   評估邏輯複雜度
    -   評估依賴數量多寡
    -   評估優先級 (money, bug, importance)

顧名思義很複雜，例如程式有一堆巢狀的if, switch, for等等結構。目前有現成的工具可以查這段 code 的複雜程度。
通常依賴越多，也會較難測，因為需要逐一隔離依賴
優先級:
和錢有關係的
時常壞掉
-   它在產品裡面很重要，萬一有 bug 會發生嚴重的後果
或者是以前在正式環境出現過的 bug。這個是價值最高的 test case

最終可以得出哪些 class 是好測/難測、優先順序

 那什麼東西的投資報酬率最高？
要修正的BUG（BUG越晚發現修正成本就更高）
實務上常跑到的scenario
最主要的情境
和錢有關的
和人命有關的（EX：自動駕駛系統）
最常改到的CODE，可以避免被改錯

我都說寫測試就像買每年都要續約跟調漲的保險，買保險是要成本的，怎麼選擇好的測試 ROI 策略是很重要的。



----- 

## **Best Practice of Unit Test**
 

 - 3A Pattern
    - Arrange
    - Act 
    - Assert



-  一個測試案例只驗證一件事
-  測試案例不與外部（檔案、DB、網路、物件）直接相依
-  測試案例不具備邏輯
-  測試案例之間無相依性
-  測試案例的命名不過於技術細節 



- 單元測試中的 3 種驗證方式
    - 待測物件的回傳值  (計算1 + 2 結果是否等於 3 ?)
    - 待測物件的內部狀態  (Stack 執行 pop/push 後, length 是多少?)
    - 待測物件與相依物件的互動 (DB若有值，待測物件是否未呼叫 API ?)
- 一個測試案例只會有一個驗證方式
 

 
測試的品質
測試的程式一定要重構
測試的語意一定要清楚明白，這是為了讓測試更容易理解，可以很簡單的的從測試的程式碼由語意就能理解這個測試要做什麼
寫測試的難度會反應程式的好壞


-----

## **結論**
 
### **References**
 
- [unit-testing-best-practices](https://docs.microsoft.com/zh-tw/dotnet/core/testing/unit-testing-best-practices)
- [You are naming your tests wrong!](https://enterprisecraftsmanship.com/posts/you-naming-tests-wrong/)