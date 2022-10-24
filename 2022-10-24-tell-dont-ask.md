---
layout: post
title: "Tell, Don't Ask"
author: "Kai-Sheng"
permalink: /articles/tell-dont-ask
categories: [Java]
image: /assets/image/site-image-small.png
--- 
  
多年來 Java 流行分層設計，因此出現了許多 VO, DTO 這種只有資料的而沒有邏輯的 class，它們被設計成只允許擁有 getter, setter 方法。在我的經驗中，既然 Lombok 這麼方便就產生了 getter, setter，因此讓很多人忽略了 **Tell, Don't Ask** 原則的建議。例如有一段「限定北部會員才可購買」的邏輯：

```java
if (customer.getMembership().getExpiredDate() > today &&
    customer.getAddress().getCity().contains("Taipei") &&
    customer.getWallet().getBalance() > product.getPrice()) {
    orderService.create(customer, product);
}
```
像這樣詢問物件的狀態，導致它本不應該被暴露的內部狀態都洩漏出去了。

Tell, Don't Ask 原則提醒開發者：所謂 OOP 就是將資料與操作該資料的 method 綁在一起。與其在 service 層用了一堆 getter 然後進行邏輯運算，不如直接請它做：


```java
// OrderService
if (customer.canBuy(product)) {
    orderService.create(customer, product);
}    
```

根據我的經驗，面向對象的Tell, Don't Ask 原則 方式編寫，則更易於理解和維護。

在這種風格中，對象僅使用它們內部保存的信息或它們作為消息參數接收的信息來做出決策。他們不使用其他對象持有的信息做出決定。也就是說，對象通過相互發送命令來告訴對方要做什麼，它們不會互相詢問信息，然後根據這些查詢的結果做出決定。這種風格的最終結果是很容易將一個對象交換為另一個可以響應相同命令但


你應該努力告訴對像你想讓他們做什麼；不要問他們關於他們的狀態的問題，做出決定，然後告訴他們該怎麼做。

問題是，作為調用者，您不應該根據被調用對象的狀態做出決定，從而導致您更改對象的狀態。您正在實現的邏輯可能是被調用對象的職責，而不是您的職責。讓你在對象之外做出決定違反了它的封裝。

當然，你可能會說，這很明顯。我永遠不會寫那樣的代碼。儘管如此，仍然很容易陷入檢查某些引用對象然後根據結果調用不同方法的狀態。但這可能不是最好的方法。告訴對像你想要什麼。讓它弄清楚如何去做。以聲明方式而非程序方式思考！

如果您從根據職責設計類開始，則更容易擺脫這個陷阱；然後，您可以自然地指定類可以執行的命令，而不是通知您對象狀態的查詢。


我覺得這句話很好的將面向過程和麵向對象區分了出來。也就是說，對於對象來說，我應該直接告訴它你該做什麼，而不是先去問它的狀態，然後根據這個狀態再去告訴對像你需要做什麼什麼，怎麼怎麼做，至於具體如何去做，不應該是我考慮的事情，而是對象自己的事情，你只發出命令，然後對像做完後返回給你結果即可，你不需要知道拿到這個結果的具體過程是如何實現的。 
  
 其實，仔細想想，對象的狀態是什麼樣，該不該做，以及如何做都是對象自己該處理的，而不是我該操心的，這麼做只能破壞了對象的封裝，讓調用它的代碼和這個對象緊耦合起來。這樣也變得很好測試。
 
 
 

## **後記**
 
 
### **更多你可能會感興趣的文章**
- [常見的 Interface 錯誤用法](/articles/anti-pattern-of-java-interface-impl-style)
- [如何提高程式碼的可測試性 (Testability)](/articles/testability)