---
layout: post
title: "Tell, Don't Ask"
author: "Kai-Sheng"
permalink: /articles/tell-dont-ask
categories: [Java]
image: /assets/image/site-image-small.png
--- 

Web MVC 三層架構盛行多年，因此許多專案中都出現了 VO, DTO 只有資料而沒有邏輯的資料類別，也稱為貧血模型(Anemic Domain Model)，這種設計並不符合物件導向的概念，破壞了物件的封裝特性，容易讓開發者寫出偏向程序導向的設計，使得物件之間的耦合度更高。Tell, Don't Ask 原則意旨在提醒開發者，應避免類似情形發生。

## **問題**

例如有個 Customer 類別，只存放顧客基本資料，沒有任何邏輯：

```java
public class Customer {

    private int age;
    private CreditCard creditCard;
    private LocalDate registerDate;

    // getter, setter
}
```

若有一需求：

- 判斷 VIP 的資格：年齡滿18歲、信用額度大於100萬、註冊滿一年的會員
- Service 根據 VIP 資格，並做出相應業務。

因此 Service 需要呼叫 Customer 的 getter 來取得資料，程式如下：

```java
// In Service.java
public void doBusiness(Customer customer) {
    if (customer.getAge() >= 18 &&
        customer.getCreditCard().getLimit() >= 1_000_000 &&
        DAYS.between(customer.getRegisterDate(), today()) >= 365) {
        doForVip();
    } else {
        doForOther();
    }    
}
```

像這樣收集許多物件的資訊以後再做出決定，就是**程序導向**的設計。

範例程式中的 `doBusiness` 中詢問 Customer 的多個內部資料，導致 Customer 本不應該被暴露的內部狀態都洩漏出去了，其實這也是一種壞味道-- **Feature Envy**，意思是對另一個 class 的興趣高於自己本身的 class，例如常常取得另一個 class 的多個成員變數，或是呼叫另一個 class 的多個 method。這會導致 Customer 的細節一旦有變，service 就不得不跟著變，造成了雙方緊密耦合。

Service 單元測試如下：

```java
// ServiceTest.java
@Test
public void do_for_vip() {
    Customer customer = new Customer();
    customer.setAge(35);        
    customer.setRegisterDate(LocalDate.of(2010, 1, 1)); 
    CreditCard creditCard = new CreditCard();
    creditCard.setLimit(5_000_000);
    customer.setCreditCard(creditCard);       // 暴露過多 Customer 的細節

    service.doBusiness(Customer);

    verify(service).doForVip();
}

@Test
public void do_for_other() {
    Customer customer = new Customer();
    customer.setAge(17);    
    customer.setRegisterDate(LocalDate.of(2010, 1, 1));  
    CreditCard creditCard = new CreditCard();
    customer.setCreditCard(creditCard);       // 暴露過多 Customer 的細節

    service.doBusiness(customer);

    verify(service).doForOther();
}

```

在這種程序導向設計風格中，一個物件與鄰居、鄰居的鄰居的結構耦合，這種情況不容易用 Mock Objects 進行測試。您會發現自己創建了許多假資料，這些假資料只是為了讓待測物件 (SUT) 到達它實際使用的對象而存在。

因為 vip 的具體實作細節也跟著暴露，

因此在 Service 的單元測試中，可能會寫出敏感的 test case，意思是只要 Customer 內部有一點改動，就可能導致 Service 的測試失敗，這是不合理的現象。這是程式碼需要重構的一個跡象


## **重構**
此時可用 **Move Method** 的重構手法，將這段邏輯移動到 Customer 之中

```java
public class Customer {

    private int age;
    private CreditCard creditCard;
    private LocalDate registerDate;

    // Move from Service class
    public boolean isVip() {
        return this.age >= 18 &&
                this.creditCard.getLimit() >= 1_000_000 &&
                DAYS.between(this.registerDate, today()) >= 365;
    }

    // setter
}
```

對於 Service 來說，不應該不斷詢問 Customer 的內部資訊，然後再根據這些資訊去做業務邏輯。較佳的作法是：Service 應該直接命令 Customer 該做什麼，Customer 做完後回傳結果即可，Service 不必了解 Customer 內部具體是怎麼實作的。


```java
// Service.java
public void doBusiness(Customer customer) {
    if (customer.isVip()) {
        doForVip();
    } else {
        doForOther();
    }
}
```

經過重構後，Service 不再耦合 vip 的實作細節，而是直接呼叫 `isVip()` 。從可測試性的角度來看，這也變得很好測試。我們不必再準備各種 Customer 資料來測試不同行為，也可避免更動 vip 的判斷條件。現在只要控制 mocked customer 與驗證 service 即可，測試意圖變得更明確：

```java
// ServiceTest.java
@Test
public void do_for_vip() {
    Customer customer = mock(Customer.class);
    when(customer.isVip()).thenReturn(true);

    service.doBusiness(customer);

    verify(service).doForVip();
}

@Test
public void do_for_other() {
    Customer customer = mock(Customer.class);
    when(customer.isVip()).thenReturn(false);

    service.doBusiness(customer);

    verify(service).doForOther();
}

```

根據經驗，Tell, Don't Ask 原則方式編寫，則更易於理解和維護。

在這種風格中，對象僅使用它們內部保存的信息或它們作為消息參數接收的信息來做出決策。他們不使用其他對象持有的信息做出決定。也就是說，對象通過相互發送命令來告訴對方要做什麼，它們不會互相詢問信息，然後根據這些查詢的結果做出決定。這種風格的最終結果是很容易將一個對象交換為另一個可以響應相同命令但

你應該努力告訴對像你想讓他們做什麼；不要問他們關於他們的狀態的問題，做出決定，然後告訴他們該怎麼做。

問題是，作為調用者，您不應該根據被調用對象的狀態做出決定，從而導致您更改對象的狀態。您正在實現的邏輯可能是被調用對象的職責，而不是您的職責。讓你在對象之外做出決定違反了它的封裝。


## **結論**
物件導向設計的核心思想之一是封裝狀態，Tell, Don’t Ask 建議我們直接要求物件達成目標，而不要外露物件內部狀態和使用內部狀態的邏輯。如此一來，可降低物件之間的耦合性，提供日後改變實作的彈性。

一個好的經驗法則是，一起改變的事物應該在一起。讓設計更為內聚的作法：「把每次異動發生時，總會需要同時被修改到的程式碼放在一起。」

### **更多你可能會感興趣的文章**
- [常見的 Interface 錯誤用法](/articles/anti-pattern-of-java-interface-impl-style)
- [如何提高程式碼的可測試性 (Testability)](/articles/testability)