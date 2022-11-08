---
layout: post
title: "Tell, Don't Ask"
author: "Kai-Sheng"
permalink: /articles/tell-dont-ask
categories: [Java]
image: /assets/image/site-image-small.png
--- 

Tell, Don't Ask 原則提醒開發者：所謂物件導向設計，就是應將資料與操作該資料的 method 綁在一起。與其請求了一堆物件的內部資料再進行邏輯運算，不如直接請它做完，並回傳結果。

Web MVC 三層架構盛行多年，因此許多專案中都出現了 VO, DTO 只有資料而沒有邏輯的資料類別，也稱為貧血模型(Anemic Domain Model)，這種設計並不符合物件導向的概念，破壞了物件的封裝特性，容易讓開發者寫出偏向程序導向的設計，使得物件之間的耦合度更高。

## **問題範例**

例如有個 User 類別

```java
public class User {

    private int age;
    private int point;
    private LocalDate registerDate;

    // getter, setter
}
```

若有一需求為「判斷 User 是否擁有 VIP 資格並做出對應業務」，VIP 的資格是「user 年齡滿18歲、紅利點數大於1000點、註冊滿一年的會員」。

因此需要呼叫 user 的 getter 來取得資料，在 Service 中會像這樣寫：

```java
// In Service.java
public void doBusiness(User user) {
    if (user.getAge() >= 18 &&
        user.getPoint() >= 1000 &&
        DAYS.between(user.getRegisterDate(), today()) >= 365) {
        doSomethingForVip();
    } else {
        doAnother();
    }    
}
```

像這樣收集許多物件的資訊以後再做出決定，就是**程序導向**的設計。

**doBusiness** 中詢問多個 user 的內部資料，導致 user 本不應該被暴露的內部狀態都洩漏出去了，其實這也是一種壞味道 **Feature Envy**，意思是對另一個 class 的興趣高於自己本身的 class，例如常常取得另一個 class 的多個成員變數，或是呼叫另一個 class 的多個 method。這會導致 user 的細節一旦有變，service 就不得不跟著變，造成了雙方緊密耦合。Service 單元測試就會像這樣：

```java
// ServiceTest.java
@Test
public void do_something_for_vip() {
    User user = new User();
    user.setAge(20);
    user.setPoint(1000);
    user.setRegisterDate(LocalDate.of(2010, 1, 1)); // 暴露過多 user 的細節

    service.doBusiness(user);

    verify(service).doSomethingForVip();
}

@Test
public void do_another() {
    User user = new User();
    user.setAge(17);
    user.setPoint(1000);
    user.setRegisterDate(LocalDate.of(2010, 1, 1));  // 暴露過多 user 的細節

    service.doBusiness(user);

    verify(service).doAnother();
}

```

在編碼風格中，一個對象的實現與其鄰居及其鄰居的鄰居的結構耦合，這種風格的代碼很難用 Mock Objects 進行測試。您會發現自己創建了許多模擬對象，這些模擬對像只是為了讓被測對像到達它實際使用的對象而存在。這是代碼需要重構的一個強烈跡象：您可以通過在被測對象的直接鄰居中引入新方法來簡化代碼。

因為 vip 的具體實作細節也跟著暴露，因此在 Service 的單元測試中，可能會寫出敏感的 test case，意思是只要 user class 有一點風吹草動，就可能導致 Service 的測試失敗。


## **改善**
此時可用 **Move Method** 的重構手法，將這段邏輯移動到 User class 之中

```java
public class User {

    private int age;
    private int point;
    private LocalDate registerDate;

    // Move from Service class
    public boolean isVip() {
        return this.age >= 18 &&
                this.point >= 1000 &&
                DAYS.between(this.registerDate, today()) >= 365;
    }

    // setter
}
```

對於對象來說，我應該直接告訴它你該做什麼，而不是先去問它的狀態，然後根據這個狀態再去告訴對像你需要做什麼什麼，怎麼怎麼做，至於具體如何去做，不應該是我考慮的事情，而是對象自己的事情，你只發出命令，然後對像做完後返回給你結果即可，你不需要知道拿到這個結果的具體過程是如何實現的。

```java
// In Service
public void doBusiness(User user) {
    if (user.isVip()) {
        doSomethingForVip();
    } else {
        doAnother();
    }
}
```


Service 不再耦合 vip 的具體實作細節，從測試的角度來看，這樣也變得很好測試，這也簡化了替換實作的麻煩。以上面的例子來說，我們不用費心生出各種 User 來測試不同行為。也避免更動 User 判斷條件後，會需要修改許多測試碼的成本。現在只要控制 mocked user 與驗證 service 即可，意圖變得更明確：

```java

@Test
public void do_something_for_vip() {
    User user = mock(User.class);
    when(user.isVip()).thenReturn(true);

    service.doBusiness(user);

    verify(service).doSomethingForVip();
}

@Test
public void doAnother() {
    User user = mock(User.class);
    when(user.isVip()).thenReturn(false);

    service.doBusiness(user);

    verify(service).doAnother();
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