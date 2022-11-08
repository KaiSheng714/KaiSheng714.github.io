---
layout: post
title: "Tell, Don't Ask"
author: "Kai-Sheng"
permalink: /articles/tell-dont-ask
categories: [Java]
image: /assets/image/site-image-small.png
--- 
  
多年來 Web 分層架構盛行，因此出現了許多 VO, DTO 這種只有資料的而沒有邏輯的、只允許擁有 getter, setter 的類別。
目前大部分業務系統都是基於 MVC 架構來開發的，而這種架構實際上是一種基於貧血模型的 MVC 三層架構開發模式。

雖然這種開發模式已經成為標準的 Web 項目的開發模式，但它卻違反了物件導向 OOP 風格，是一種類似程序導向(procedure-oriented)風格， 

其實，仔細想想，對象的狀態是什麼樣，該不該做，以及如何做都是對象自己該處理的，而不是我該操心的，這麼做只能破壞了對象的封裝，容易寫出**程序式導向**的設計，讓調用它的代碼和這個對象緊耦合起來。例如有一段「限定北部會員才可購買」的邏輯：

```java
// Service
if (today().isBefore(customer.getMembership().getExpiredDate()) &&
    customer.getAddress().contains("Taipei") &&
    customer.getWallet().getBalance() > product.getPrice()) {
    orderService.create(customer, product);
}
```
像這樣詢問物件的狀態，導致它本不應該被暴露的內部狀態都洩漏出去了。

關於 Feature Envy 的定義，Martin Fowler 在 Refactoring 書中指出：「函式對於某個 class 的興趣高過對自己所處之 host class 的興趣。」
Martin Fowler 直指，這種迷戀最常發生在「資料」上。今天如果 A 類裡的某個方法，老是喜歡存取 B 類的資料來運算，這會導致 B 的細節一旦有變，A 就不得不跟著變。或是每當 B 想要改變自己身上資料的存取方式時，還得看 A 的臉色。這就造成了 A 與 B 緊密耦合，而我們並不樂見此事。

Tell, Don't Ask 原則提醒開發者：所謂 OOP 就是將資料與操作該資料的 method 綁在一起。與其在 service 層用了一堆 getter 然後進行邏輯運算，不如直接請它做：


```java
// OrderService
if (customer.canBuy(product)) {
    orderService.create(customer, product);
}    
```

物件設計的重點在於將資料與這些資料的操作行為封裝在一起，一個典型的味道是某函式對於另一個class的興趣高於自己本身的class，例如函式常常需要取得另一個class的多個成員變數，或是呼叫另一個class的多個成員函式。則這個函式也許就不該屬於這個class，此時，可以用Move Method將該函式移到另一個class。 

注意，有時 Message Chain 是可以接受的，但大多數是可以避免的。


這樣也變得很好測試。


## 可測試性
在編碼風格中，一個對象的實現與其鄰居及其鄰居的鄰居的結構耦合，這種風格的代碼很難用 Mock Objects 進行測試。您會發現自己創建了許多模擬對象，這些模擬對像只是為了讓被測對像到達它實際使用的對象而存在。這是代碼需要重構的一個強烈跡象：您可以通過在被測對象的直接鄰居中引入新方法來簡化代碼。


根據我的經驗，面向對象的Tell, Don't Ask 原則 方式編寫，則更易於理解和維護。

在這種風格中，對象僅使用它們內部保存的信息或它們作為消息參數接收的信息來做出決策。他們不使用其他對象持有的信息做出決定。也就是說，對象通過相互發送命令來告訴對方要做什麼，它們不會互相詢問信息，然後根據這些查詢的結果做出決定。這種風格的最終結果是很容易將一個對象交換為另一個可以響應相同命令但


你應該努力告訴對像你想讓他們做什麼；不要問他們關於他們的狀態的問題，做出決定，然後告訴他們該怎麼做。

問題是，作為調用者，您不應該根據被調用對象的狀態做出決定，從而導致您更改對象的狀態。您正在實現的邏輯可能是被調用對象的職責，而不是您的職責。讓你在對象之外做出決定違反了它的封裝。

 

## 結論
物件導向設計的核心思想之一是封裝狀態，Tell, Don’t Ask 建議我們直接要求物件達成目標，而不要外露物件內部狀態和使用內部狀態的邏輯。如此一來，可降低物件之間的耦合性，提供日後改變實作的彈性。

從測試的角度來看，這也簡化了替換實作的麻煩。以上面的例子來說，我們不用費心生出各種 X 來測試不同行為。也免去更動 X 格式後，會需要修改許多測試碼的成本。套用 IU 後，控制假實作之間的互動即可。

## **後記**
一個好的經驗法則是，一起改變的事物應該在一起。
讓設計更為內聚的作法：「把每次異動發生時，總會需要同時被修改到的程式碼放在一起。」

### **更多你可能會感興趣的文章**
- [常見的 Interface 錯誤用法](/articles/anti-pattern-of-java-interface-impl-style)
- [如何提高程式碼的可測試性 (Testability)](/articles/testability)



  private static class UserService {

        public void addUser(Activity activity, User user) {
            if (user.getAge() >= 18 &&
                user.getPoint() >= 1000 &&
                DAYS.between(user.getRegisterDate(), today()) >= 365) {
                doSomething();
            } else {
                doAnother();
            }
        }

        private LocalDate today() {
            return LocalDate.now();
        }
    }