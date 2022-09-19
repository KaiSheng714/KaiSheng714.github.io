---
layout: post
title: "優秀的單元測試如同好用的新冠快篩"
author: "Kai-Sheng"
permalink: /articles/good-unit-test
categories: [Design]
image: /assets/image/good-unit-test.png
--- 

單元測試已是軟體工程師必備的技能，但在我的工作經驗中，看過許多新進同仁寫的單元測試，看起來好像有那麼一回事，但實際上好像又沒測到重點，而且還很容易測試失敗。如果這樣的測試持續增長，單元測試不僅沒有帶來好處，反而使專案更不穩健，優秀的軟體開發者不可不知如何寫出優秀的單元測試。有趣的是，單元測試和新冠快篩有異曲同工之妙。

 ![good-unit-test](/assets/image/good-unit-test.png?style=center)

想像一下這樣的場景：

>
> 新人小明的第一個任務是新增一個簡單的功能，也許只需要10幾行程式。
>
> 但是當他完成後，自動測試系統卻回報了大量的測試失敗。
>
> 小明很難去理解這些測試的內容，以及為什麼會失敗，他這次寫的小功能看起來和測試沒有關係。
>
> 本應很快完成的工作卻花了小明好幾天。扼殺了他的工作效率，也挫傷了他的士氣。
>

大家都知道單元測試帶來的好處，前輩們也很負責的將產品加入了許多單元測試，但不幸的是，他們寫的這些單元測試卻產生了反效果
- 測試結果不準確：這次的更改並沒有加入真正的 bug，但是測試卻失敗了。
- 測試意義不清楚：小明很難確定哪出了問題、如何修復，以及這些測試最初應該做什麽。

## **優秀的單元測試就像好用的新冠快篩**
新冠快篩 (COVID-19 rapid test kit)


## **產品程式碼**
- 延伸閱讀: [如何提高程式碼的可測試性 (Testability)](/articles/testability)

## **Best Practice**
0. 一個測試案例只驗證一個行為
0. 回傳值驗證所佔的比例應要最高

0. 測試案例之間無相依性
測試案例之間應該要各自獨立。如果測試案例相依，萬一發生測試失敗時，容易火燒連環船、一發不可收拾，無法快速釐清問題以及很難發現問題的根源。

0. 測試案例的命名盡量清楚、口語化

```java
@Test
@DisplayName("我的測試")

```
0. 測試案例不具備邏輯

```java
@Test
public void shouldNavigateToAlbumsPage() {
    String baseUrl = "http://photos.google.com/";
    Navigator nav = new Navigator(baseUrl);
    nav.goToAlbumPage();
    assertThat(nav.getCurrentUrl()).isEqualTo(baseUrl + "/albums");
}


@Test
public void shouldNavigateToPhotosPage() {
    Navigator nav = new Navigator("http://photos.google.com/");
    nav.goToPhotosPage();
    assertThat(nav.getCurrentUrl()))
    .isEqualTo("http://photos.google.com//albums"); // Oops!
}
```


0. 驗證時，不過度指定  (over specification)

```java
@Test 
public void display_greeting_render_userName() {
    when(mockUserService.getUserName()).thenReturn("Fake User");

    userGreeter.displayGreeting(); 
        
    verify(userPrompt, times(1)).setText("Fake User", "Good morning!", "Version 2.1");
    verify(userPrompt, times(1)).setIcon(IMAGE_SUNSHINE);
}
```


```java
@Test 
public void displayGreeting_renderUserName() {    
    when(mockUserService.getUserName()).thenReturn("Fake User");

    userGreeter.displayGreeting(); 

    // 只驗證我們真正在意的 userName
    verify(userPrompter).setText(eq("Fake User"), any(), any());
}


@Test 
public void displayGreeting_timeIsMorning_useMorningSettings() {
    setTimeOfDay(TIME_MORNING);

    userGreeter.displayGreeting(); 
    
    // 只驗證我們真正在意的 "早安、早晨的 icon"
    verify(userPrompt).setText(any(), eq("Good morning!"), any());
    verify(userPrompt).setIcon(IMAGE_SUNSHINE);
}
```

0. 不過度依賴 mock framework

```java
@Test 
public void credit_card_is_charged() {    
    // 過度依賴 mock framework
    paymentProcessor =
        new PaymentProcessor(mockCreditCardServer, mockTransactionProcessor);    
    when(mockCreditCardServer.isServerAvailable()).thenReturn(true);
    when(mockTransactionProcessor.beginTransaction()).thenReturn(transaction);
    when(mockCreditCardServer.initTransaction(transaction)).thenReturn(true);
    when(mockCreditCardServer.pay(transaction, creditCard, 500)).thenReturn(false);
    when(mockTransactionProcessor.endTransaction()).thenReturn(true);

    paymentProcessor.processPayment(creditCard, Money.dollars(500));

    // 太多假的行為，根本不曉得 transaction 有沒有真正成功，即測試結果可能不準確
    verify(mockCreditCardServer, times(1)).pay(transaction, creditCard, 500);
}
```


0. 進行互動驗證時，只驗證會改變外部的行為

```java
@Test 
public void grant_user_permission() {   
    // ... arrange, act ...

    verify(mockPermissionDatabase, times(1)).addPermission(FAKE_USER, USER_ACCESS);
    verify(mockPermissionDatabase, times(1)).getPermission(FAKE_USER);
}
```

```java
@Test 
public void grant_user_permission() {   
    // ... arrange, act ...

    // 合理, 應該只要 verify "會影響外界"的行為
    verify(mockPermissionDatabase, times(1)).addPermission(FAKE_USER, USER_ACCESS);

    // 不應 verify "不會影響外界"的行為
    verify(mockPermissionDatabase, times(1)).getPermission(FAKE_USER);
}
```

## **後記**
 

### **References**
- [Software Engineering at Google: Lessons Learned from Programming Over Time](https://www.amazon.com/Software-Engineering-Google-Lessons-Programming/dp/1492082791)

 
## **更多你可能會感興趣的文章**
- [常見的 Interface 錯誤用法](/articles/anti-pattern-of-java-interface-impl-style)
- [如何提高程式碼的可測試性 (Testability)](/articles/testability)
- [軟體設計原則 YAGNI (You aren't gonna need it!)](/articles/testability)