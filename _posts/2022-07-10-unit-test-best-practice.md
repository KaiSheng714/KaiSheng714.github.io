---
layout: post
title: "單元測試 Best Practice"
author: "Kai-Sheng"
permalink: /articles/unit-test-best-practice
categories: [Unit-testing]
image: /assets/image/unit-test-best-practice.png
--- 
  
 
 
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
 
-----

## **結論**
 
### **References**
 
- [unit-testing-best-practices](https://docs.microsoft.com/zh-tw/dotnet/core/testing/unit-testing-best-practices)
- [You are naming your tests wrong!](https://enterprisecraftsmanship.com/posts/you-naming-tests-wrong/)