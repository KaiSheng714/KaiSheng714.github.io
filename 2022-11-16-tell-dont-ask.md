---
layout: post
title: "Tell, Don't Ask"
author: "Kai-Sheng"
permalink: /articles/tell-dont-ask
categories: [Java]
image: /assets/image/site-image-small.png
--- 
  
多年來 Web MVC 分層架構盛行，因此許多專案出現了類似 VO, DTO 這種只有資料的而沒有邏輯的、只允許擁有 getter, setter 的類別。雖然目前大部分系統都是基於 MVC 架構來開發的，但它卻違反了物件導向 OOP 設計風格，破壞了物件的封裝特性，讓開發者更容易寫出**程序式導向**的設計，讓它的和這個對象緊耦合起來。

## **問題**


```java
public class User {

    private int age;
    private int point;
    private LocalDate registerDate;

    // getter, setter
}
```

如果在 Service 中，有一個有 VIP 資格的條件為「年齡大於18歲、紅利點數大於1000點、註冊滿一年的會員」：

```java
// In Service
public void myFunction(User user) {
    if (user.getAge() >= 18 &&
        user.getPoint() >= 1000 &&
        DAYS.between(user.getRegisterDate(), today()) >= 365) {
        doSomethingForVip();
    } else {
        doAnother();
    }    
}
```

因為 vip 的條件判斷也跟著暴露了，因此在單元測試中，使人更容易寫出敏感的 test case，一點風吹草動就可能導致測試失敗。

```java

@Test
public void vip() {
    User user = new User();
    user.setAge(20);
    user.setPoint(1000);
    user.setRegisterDate(LocalDate.of(2010, 1, 1)); // 暴露過多 user 的細節

    service.myFunction(user);

    verify(service).doSomethingForVip();
}

@Test
public void not_vip() {
    User user = new User();
    user.setAge(17);
    user.setPoint(1000);
    user.setRegisterDate(LocalDate.of(2010, 1, 1));  // 暴露過多 user 的細節

    service.myFunction(user);

    verify(service).doAnother();
}

```

像這樣詢問物件的狀態，導致它本不應該被暴露的內部狀態都洩漏出去了，這也是其中一種壞味道 Feature Envy，意思是某函式對於另一個 class 的興趣高於自己本身的 class，例如函式常常需要取得另一個 class 的多個成員變數，或是呼叫另一個 class 的多個 method。這會導致 class 的細節一旦有變， service 就不得不跟著變，造成了雙方緊密耦合。

物件設計的重點在於將資料與這些資料的操作行為封裝在一起，則這個函式也許就不該屬於這個class，此時，可以用 Move Method將該函式移到另一個class。 

## **改善**
Tell, Don't Ask 原則提醒開發者：所謂 OOP 就是將資料與操作該資料的 method 綁在一起。與其在 service 層用了一堆物件的 getter 然後進行邏輯運算，不如直接請它做並回傳結果，此時可以用 Move Method 的重構手法，將這段邏輯移動到 User class 之中

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

    // 這裡就可不必寫 getter
}
```

```java
// In Service
public void myFunction(User user) {
    if (user.isVip()) {
        doSomethingForVip();
    } else {
        doAnother();
    }
}
```

在編碼風格中，一個對象的實現與其鄰居及其鄰居的鄰居的結構耦合，這種風格的代碼很難用 Mock Objects 進行測試。您會發現自己創建了許多模擬對象，這些模擬對像只是為了讓被測對像到達它實際使用的對象而存在。這是代碼需要重構的一個強烈跡象：您可以通過在被測對象的直接鄰居中引入新方法來簡化代碼。
從測試的角度來看，這也簡化了替換實作的麻煩。以上面的例子來說，我們不用費心生出各種 User 來測試不同行為。也避免更動 User 判斷條件後，會需要修改許多測試碼的成本。現在只要控制 mocked user 與驗證 service 即可，意圖變得更明確了。

```java

@Test
public void vip() {
    User user = mock(User.class);
    when(user.isVip()).thenReturn(true);

    service.myFunction(user);

    verify(service).doSomethingForVip();
}

@Test
public void not_vip() {
        User user = mock(User.class);
    when(user.isVip()).thenReturn(false);

    service.myFunction(user);

    verify(service).doAnother();
}

```

根據我的經驗，面向對象的Tell, Don't Ask 原則 方式編寫，則更易於理解和維護。
Service 不再耦合 vip 的具體實作細節，這樣也變得很好測試。

在這種風格中，對象僅使用它們內部保存的信息或它們作為消息參數接收的信息來做出決策。他們不使用其他對象持有的信息做出決定。也就是說，對象通過相互發送命令來告訴對方要做什麼，它們不會互相詢問信息，然後根據這些查詢的結果做出決定。這種風格的最終結果是很容易將一個對象交換為另一個可以響應相同命令但


你應該努力告訴對像你想讓他們做什麼；不要問他們關於他們的狀態的問題，做出決定，然後告訴他們該怎麼做。

問題是，作為調用者，您不應該根據被調用對象的狀態做出決定，從而導致您更改對象的狀態。您正在實現的邏輯可能是被調用對象的職責，而不是您的職責。讓你在對象之外做出決定違反了它的封裝。


## **結論**
物件導向設計的核心思想之一是封裝狀態，Tell, Don’t Ask 建議我們直接要求物件達成目標，而不要外露物件內部狀態和使用內部狀態的邏輯。如此一來，可降低物件之間的耦合性，提供日後改變實作的彈性。

一個好的經驗法則是，一起改變的事物應該在一起。
讓設計更為內聚的作法：「把每次異動發生時，總會需要同時被修改到的程式碼放在一起。」

### **更多你可能會感興趣的文章**
- [常見的 Interface 錯誤用法](/articles/anti-pattern-of-java-interface-impl-style)
- [如何提高程式碼的可測試性 (Testability)](/articles/testability)