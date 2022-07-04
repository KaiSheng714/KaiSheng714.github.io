---
layout: post
title: "Private void static 要怎麼測"
 
meta: design,clean-code
author: "Kai-Sheng"
permalink: /articles/ 
categories: [test]
--- 

 

 不需要特別測
private method 屬於 public method 內容的其中一部分。
單元測試是用來模擬外部如何使用待測物件，驗證其行為是否符合預期，代表著外部使用者不需要了解測試目標物件非 public 的行為。因此只測 public 即可一起測到 private method 。
