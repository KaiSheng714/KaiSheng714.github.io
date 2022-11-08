---
layout: post
title: "Tell, Don't Ask"
author: "Kai-Sheng"
permalink: /articles/tell-dont-ask
categories: [Java]
image: /assets/image/site-image-small.png
--- 

Web MVC 三層架構已盛行多年，因此在許多專案中經常可看到 VO, DTO 這種只有資料而沒有邏輯的類別，也稱為貧血模型(Anemic Domain Model)。這種設計並不符合物件導向的理念，容易破壞物件的封裝特性，讓開發者寫出程序導向風格的設計，使得物件之間的耦合度更高。Tell, Don't Ask 原則意旨在提醒開發者應盡量避免類似情形。

## **問題**

例如有個 Customer 類別，只存放顧客基本資料和 getter, setter，沒有任何邏輯：

```java
public class Customer {

    private int age;
    private CreditCard creditCard;
    private LocalDate registerDate;

    // getter, setter...
}
```

若有一需求：

- 程式需判斷顧客是否有 VIP 的資格：年齡滿18歲、信用額度大於100萬、註冊滿一年的會員
- Service 根據顧客是否擁有 VIP 資格，並做出相應業務邏輯。

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

Service 中的 `doBusiness` 中首先詢問 Customer 的多個內部資料，導致 Customer 本不應該被暴露的內部資料都洩漏出去，造成了雙方緊密耦合。其實這也是一種壞味道 -- **Feature Envy**，意思是對另一個 class 的興趣高於自己本身，例如常常存取另一個 class 的多個 field 或 method。

此外，Service 不僅與 Customer 耦合，也和 Customer 的內部 CreditCard 耦合，這種情況下不易單純只用一個 mock 進行測試，而是需要建立許多測試資料或 mock，因此測試可讀性也較差。Service 的單元測試如下：

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

因為判斷 vip 的具體實作細節暴露在外，所以在 Service 的單元測試中，可能會寫出敏感的 test case：只要 Customer 內部有一點改動，就可能導致 Service 的測試失敗，這是程式碼需要重構的一個跡象。

## **重構**

可用 **Move Method** 的重構手法，將這段邏輯搬到 Customer 之中。這也是一種讓物件更具備內聚力的作法 -- 異動發生時，把可能會需要同時被修改到的程式碼放在一起：

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

    // setter...
}
```

因此 Service 不應該一直詢問 Customer 的內部資訊，而是應該直接命令 Customer 該做什麼，Customer 做完後回傳結果即可，Service 不必了解 Customer 內部具體是怎麼實作的。經過重構後，Service 不再耦合判斷 vip 實作細節，而是直接呼叫 `customer.isVip()`。這樣不只降低物件之間的耦合性，也提高 Customer 的內聚力和資訊隱藏的封裝性。

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

這也提高了可測試性。我們不必再準備各種 Customer 資料來測試不同行為，也可避免更動 vip 的判斷條件。現在只要控制 mocked customer 與驗證 service 即可，測試意圖變得更明確：

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

## **結論**
封裝是物件導向設計的理念之一，Tell, Don't Ask 原則建議我們應該直接命令物件去完成目標，而不是暴露物件內部狀態。套用此原則，就能更容易設計出好理解、好維護、高內聚、低耦合的程式。

### **更多你可能會感興趣的文章**
- [常見的 Interface 錯誤用法](/articles/anti-pattern-of-java-interface-impl-style)
- [如何提高程式碼的可測試性 (Testability)](/articles/testability)