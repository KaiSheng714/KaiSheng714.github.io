---
layout: post
title: "Tell, Don't Ask"
author: "Kai-Sheng"
permalink: /articles/tell-dont-ask
categories: [Design]
image: /assets/image/site-image-small.png
--- 

Web MVC 三層架構已盛行多年，因此在許多專案中經常可看到 VO, DTO 只有資料而沒有邏輯的物件，也稱為貧血模型(Anemic Domain Model)。然而這種設計並不符合物件導向的設計理念，因為它破壞了物件的封裝特性，使物件的內部資訊容易被外部濫用，提高與外部物件的耦合程度。Tell, Don't Ask 軟體設計原則意旨在提醒開發者應盡量避免類似情形。

## **問題描述**

舉例來說，Customer class 只存放顧客的基本資料，和一些 getter, setter，沒有任何邏輯：

```java
public class Customer {

    private int age;
    private CreditCard creditCard;
    private LocalDate registerDate;

    // getter, setter...
}
```

若有一需求：

- 程式需判斷顧客是否有 VIP 的會員資格：年齡滿18歲、信用額度大於100萬、註冊滿一年的會員
- Service 根據會員資格，並做出相應業務邏輯。

因此 Service 需多次呼叫 Customer 的 getter 來取得資料，程式如下：

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

從這簡短的程式碼就可以發現幾個議題：

- `doBusiness` 多次詢問 Customer，導致 Customer 本不應該被暴露的內部資料都洩漏出去，也造成了雙方緊密耦合。這是一種壞味道 -- **Feature Envy**，意思是對另一個物件的興趣高於自己。
- 若專案中其他地方需要使用相同邏輯時，同事可能會不小心再寫一遍，實作重複的知識 **(DRY原則)**。
- Service 和 Customer 內部的 CreditCard 耦合，違反了最小知識原則 **(Law of Demeter)**。
- 因此，當 Customer 的內部資料操控在別人手上時，就容易被濫用，導致**降低物件內聚力，提高與外部的耦合程度**。
- 以單元測試的角度來說，會需要建立許多測試資料或 mocked object，因此測試可讀性、維護性變得比較差。Service 的單元測試如下：

```java
// ServiceTest.java
@Test
public void do_for_vip() {
    CreditCard creditCard = new CreditCard().setLimit(5_000_000);
    Customer customer = new Customer()
                            .setAge(20)
                            .setCreditCard(creditCard)
                            .setRegisterDate(LocalDate.of(2010, 1, 1));

    service.doBusiness(customer);

    verify(service).doForVip();
}

@Test
public void do_for_other() {
    CreditCard creditCard = new CreditCard().setLimit(5_000_000);
    Customer customer = new Customer()
                            .setAge(17)
                            .setCreditCard(creditCard)
                            .setRegisterDate(LocalDate.of(2010, 1, 1));

    service.doBusiness(customer);

    verify(service).doForOther();
}

```

這裡不難發現一個問題：**測試中暴露過多 Customer 的內部細節，導致每個 Customer setter 都可能影響 Service 的測試結果**。所以在 Service 的單元測試中，開發者可能會寫出較敏感的 test case。這種現象表明程式碼需要被重構。

## **應用 Tell, Don't Ask 原則**

Tell, Don't Ask 原則提醒開發者：與其一直詢問物件的內部資料然後執行運算，還不如直接命令它執行任務。

因此 Service 應該直接命令 Customer 做事，做完後回傳結果即可，也不必了解 Customer 內部具體是怎麼實作的。此時可用 **Move Method** 的重構手法，將這段邏輯搬進 Customer 之中，讓它處在對的地方，萬一日後有異動發生時，**可能會需要同時被修改的程式碼都放在一起**，這也是一種讓物件更具備內聚力的手法。

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

經過重構後，Service 不再耦合判斷 vip 實作細節，而是直接呼叫 `customer.isVip()`。對比重構前的例子，這樣不只降低物件之間的耦合性，也提高 Customer 的內聚力和資訊隱藏的封裝性。

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

### **ServiceTest**

撰寫單元測試時，我們不必再準備各種 Customer 資料來測試不同行為，現在只要控制 mocked customer 就很容易 verify，測試意圖變得更明確：

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

### **CustomerTest**
同場加映，重構後當然也可以很容易的測試 Customer 內部的 `isVip()` 邏輯，寫出更多不同的 test case：

```java
// CustomerTest.java
@Test
public void customer_age_under_18_is_not_vip() {
    Customer customer = new Customer().setAge(17);
    assertFalse(customer.isVip());
}

@Test
public void customer_credit_limit_under_1M_is_not_vip() {
    CreditCard card = new CreditCard().setLimit(999_999);
    Customer customer = new Customer()
            .setAge(18)
            .setCreditCard(card);
    assertFalse(customer.isVip());
}

@Test
public void customer_register_less_than_1_year_is_not_vip() {
    CreditCard card = new CreditCard().setLimit(1_000_000);
    Customer customer = new Customer()
            .setAge(18)
            .setCreditCard(card)
            .setRegisterDate(LocalDate.of(2021, 11, 9));
    givenTodayIs(2022, 11, 8);
    assertFalse(customer.isVip());
}

// a lot of test cases...
```

## **結論**
封裝，是物件導向設計的重要特性之一，Tell, Don't Ask 原則建議我們盡量不要破壞封裝，應該直接命令物件去完成任務，而不是暴露物件內部資訊。若是能妥善運用此原則，就能更容易設計出好理解、好維護、好測試、高內聚、低耦合的程式。

當然，此原則並不是鐵律，VO,DTO 仍然有其存在的價值，因此開發者需要衡量與評估專案的情況，再決定出最適合的作法。

### **References**
- [TellDontAsk - Martin Fowler](https://martinfowler.com/bliki/TellDontAsk.html)
- [這裡貧血、那裡充血，到底資料模型要怎麼設計？](https://dotblogs.com.tw/regionbbs/2021/05/29/anemicdomainmodel)

### **更多你可能會感興趣的文章**
- [如何寫出優秀的單元測試 (Best Practice)](/articles/good-unit-test)
- [如何提高程式碼的可測試性 (Testability)](/articles/testability)