---
layout: post
title: "Java Lombok-比@Builder更好的選擇"
author: "Kai-Sheng"
permalink: /articles/lombok-builder
categories: [Java]
image: /assets/image/lombok-cover.png
--- 


## **問題描述**




因此應盡量按規格在必填的 field 加上 Lombok 的 `@NonNull`：如果沒有賦值，則立即拋出 NullPointerException，fail fast。又或者可以透過重構手段，**讓 class 變小一點**，減少開發者忘記賦值的機率。

備註: `@NonNull` 僅適用於非 primitive type。


### **References**
- [Be Careful With Lombok](https://levelup.gitconnected.com/be-careful-with-lombok-2e2edfc01110)
- [5 Tips For Using Lombok In Production](https://dzone.com/articles/5-tips-for-using-lombok-in-production)

### **更多你可能會感興趣的文章**
- [軟體設計原則 Tell, Don't Ask](/) 
- [Java Jackson ObjectMapper 教學與注意事項](/articles/object-mapper)
- [分析 Spring 的依賴注入模式](https://kaisheng714.github.io/articles/analyzing-dependency-injection-patterns-in-spring)