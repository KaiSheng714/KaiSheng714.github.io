---
layout: post
title: "優秀的單元測試如同好用的新冠快篩"
author: "Kai-Sheng"
permalink: /articles/good-unit-test
categories: [Design, Unit-testing]
image: /assets/image/good-unit-test.png
--- 

單元測試已是軟體工程師必備的技能，但在我看過許多新進同仁寫的單元測試中，有些測試看起來好像有那麼一回事，但實際上好像又沒測到重點，而且還很容易測試失敗。如果像這樣的不良測試持續增加，不僅沒有帶來好處，還使專案更不穩健，因此遵循 Best Pracetice 是很重要的。巧合的是，優秀的單元測試和新冠快篩居然有異曲同工之妙。

 ![good-unit-test](/assets/image/good-unit-test.png?style=center)

想像一下這樣的場景：

>
> 新人小明的第一個任務是新增一個簡單的功能，也許只需要10幾行程式。
>
> 但是當他完成後，自動測試系統卻回報了大量的測試失敗。
>
> 小明很難去理解這些測試的內容，以及為什麼會失敗，因為他這次寫的小功能看起來和測試沒有關係。
>
> 本應很快完成的工作卻花了小明好幾天。扼殺了他的工作效率，也挫傷了他的士氣。
>

大家都知道單元測試有許多好處，前輩們也很負責的將產品加入了許多單元測試。但不幸的是，他們寫的這些單元測試卻產生了反效果
- 測試結果不準確：這次的更改並沒有加入真正的 bug，但是測試卻失敗了。
- 測試意義不清楚：小明很難確定哪出了問題、如何修復，以及這些測試最初應該做什麽。

這種案例層出不窮，我也當過小明，世界上也不會有最後一個小明。要如何盡量避免這種情況發生呢 ?

## **好用的新冠快篩**
新冠快篩 (COVID-19 rapid test kit)，大家應該都用過吧，但你有注意到嗎? 它也是一種測試 (Test)，大家可以自己問自己：你會想買什麼樣的快篩呢？不外乎有以下幾個特點:

- **準確**: 一條線就是陰性，沒有感染新冠；兩條線就是陽性，確診新冠。好的快篩就是要準確率高，不會結果不一、時好時壞、偽陰偽陽 (false negative, false positive)
- **價格便宜**: 能以低成本的方式輕鬆取用。一支快篩如果要價1000台幣，而且每個月還要多繳100，我想應該沒有人想買。
- **快速**: 好用的快篩，應該要很快就有結果，越快越好。如果某人疑似確診，快篩驗完要還等個3~5天才有結果，那他在這段期間有可能造成更多的感染，讓結果惡化。
- **結果一目了然**: 一條線就是陰性，沒有感染新冠；兩條線就是陽性，確診新冠。結果簡單清楚一目了然，不允許有難以判讀的情況，不應該有三條線或沒有線。檢驗結果也要鐵口直斷，不能將兩條線說明成B肝、愛滋等等不相干的疾病。

## **優秀的單元測試如同好用的新冠快篩**
同理可證，你希望產品中有什麼樣子的單元測試呢？通常不外乎有以下幾個特點:

- **準確**: 有 bug 時就要盡可能抓出來；沒有 bug 就不應該亮紅燈。
- **維護成本便宜**: 一旦寫好後就不會再改，除非產品需求異動。
- **執行速度快，越快越好**: 能快速得到 feedback，開發人員也才能快速的做出對應。
- **測試結果好理解、直觀**: 開發人員才能知道哪一行出了什麼問題，並最小化問題的範圍。

擁有這些特點，就可以算是優秀的單元測試了！這也是優秀的開發者們應努力追求的目標。但要寫出優秀的單元測試並不容易，我將介紹幾個手段來更靠近這個目標，但前提是要提升產品程式碼的可測試性。

## **提升產品程式碼的可測試性**
可測試性和優秀的單元測試是高度相關的。通常而言，可測試性太低的程式碼，是無法寫出優秀的單元測試的。例如程式中含有大量的 bad smell，這種情況下的單元測試也會一團亂。因此必須先提高程式碼的可測試性，才較有機會寫出優秀的單元測試。

- 延伸閱讀: [如何提高程式碼的可測試性 (Testability)](/articles/testability)

## **優秀的單元測試之設計原則**

### **良好的結構** 
盡量以 `Arrange, Act, Assert` 或 `Given, When, Then` 的 pattern 去寫單元測試。透過這樣的 pattern 讓測試案例比較能表達**一種行為**，會比較貼近使用者，畢竟單元測試就是在**模擬使用者如何使用該產品程式碼**。

### **一個測試案例只驗證一個行為**
如同好的快篩應只能篩一種病，一個測試案例也應只驗證一種行為；如果不是這個測試案例要檢驗的行為，就不應該把它們寫在一起，而是把它們拆分成多個測試案例。

### **測試案例之間無相依性**
測試案例之間應該要各自獨立。如果測試案例相依，萬一其中一個發生測試失敗時，容易讓其他測試案例一起失敗，火燒連環船、一發不可收拾，無法快速釐清問題以及很難發現問題的根源。

### **測試案例的命名盡量清楚、口語化**

命名是一件很高深的學問，對於母語非英語的我們更是難以做出清楚的命名。不過幸好 Junit 5 之後就可以透過 `@DisplayName` 來命名中文 test case，而且它也會被輸出到 test report 中，相當實用：

```java
@Test
@DisplayName("我的測試")
void my_test() { ... }

```

### **測試案例不具備邏輯運算**

如果在測試案例中加入邏輯運算，例如 `if`, `else`, 加減乘除等等運算，甚至迴圈，會讓測試變得更複雜。試問：萬一測試的一段邏輯有 bug，到底是產品程式出問題？還是測試本身的問題？ 這是一個在測試中加入邏輯的不良示範：

```java
@Test
public void shouldNavigateToAlbumsPage() {
    String baseUrl = "http://photos.google.com/";
    Navigator nav = new Navigator(baseUrl);
    nav.goToAlbumPage();    
    assertThat(nav.getCurrentUrl()).isEqualTo(baseUrl + "/albums"); // Oops!，這個結果會多一個 slash
}
```

看到了嗎，這個例子用字串 `+` 運算把 bug 也埋進來了。測試案例就是應該清楚直觀，盡量將預期結果直接 hard code 清楚的寫出來，而不要用運算的。


### **進行相依驗證時，只驗證會改變外部的行為**

實務上我們常用 `Mockito.verify` 來驗證帶測物件與相依物件的互動。例如有一支 grant user 程式的單元測試如下：

```java
@Test 
public void grant_user_permission() {   
    // ... arrange, act ...

    verify(mockPermissionDatabase, times(1)).addPermission(FAKE_USER, USER_ACCESS);
    verify(mockPermissionDatabase, times(1)).getPermission(FAKE_USER);
}
```

其實 `getPermission` 是不用 `verify` 的。我們應該要想清楚這個測試在驗證什麼？什麼才是我們最關心的？通常就是會影響外界的行為，或著是這個 method 有沒有 side-effect，有 side-effect 的行為我們才需要 verify。這是個取捨，如果程式每個路徑都去 verify，確實最有可能會抓到 bug、保護力最高。但是我的經驗是，這樣容易導致讓測試變得太敏感，就像剛剛小明的例子一樣，動不動就失敗。因此比較好的做法是 **只 verify 有 side-effect** 的行為。

### **驗證時，不過度指定  (over specification)**
如果我們驗證的時候過度指定，會讓測試程式變得很敏感，也容易出現不準確的問題。例如有一個打招呼的程式，測試如下：

```java
@Test 
public void display_greeting_render_userName() {
    when(mockUserService.getUserName()).thenReturn("Fake User");

    userGreeter.displayGreeting(); 
        
    verify(userPrompt, times(1)).setText("Fake User", "Good morning!", "Version 2.1");
    verify(userPrompt, times(1)).setIcon(IMAGE_SUNSHINE);
}
```

看到了嗎？這個測試案例所關心的事情應該是 `userName`，如同它的本身命名一樣。但是它 verify 太多不相干的雜訊，如果 `setText` 其它兩個參數一但改變，就會亮起紅燈。比較好的做法是先想清楚**到底要測什麼**？將我們要測的行為想好，一個測試案例就只關注一件事：

```java
@Test 
public void displayGreeting_renderUserName() {    
    when(mockUserService.getUserName()).thenReturn("Fake User");

    userGreeter.displayGreeting(); 

    // 只驗證我們真正在意的第一件事: userName
    verify(userPrompter).setText(eq("Fake User"), any(), any());
}


@Test 
public void displayGreeting_timeIsMorning_useMorningSettings() {
    setTimeOfDay(TIME_MORNING);

    userGreeter.displayGreeting(); 
    
    // 只驗證我們真正在意的第2件事: icon
    verify(userPrompt).setIcon(IMAGE_SUNSHINE);
}
```

這樣做的好處是：
- 如果是上面的測試結果是錯的，我就可以明確知道，我的程式沒有把名字寫對。
- 如果是下面的測試結果是錯的，我就可以明確知道，我的程式可能把早安說成晚安了。

### **不過度依賴 mock framework**
過度依賴 mock framework 是很多人會犯的錯，因為大家都想把單元測試寫出來，所以用了各式各樣的 mock 技巧，但如果 mock 越多，越會讓測試結果與事實結果越背離。例如有一個刷卡交易的程式，驗證信用卡是否有被扣款的單元測試如下：

```java
@Test 
public void credit_card_is_charged() {    
    paymentProcessor =
        new PaymentProcessor(mockCreditCardServer, mockTransactionProcessor);    
    when(mockCreditCardServer.isServerAvailable()).thenReturn(true);
    when(mockTransactionProcessor.beginTransaction()).thenReturn(transaction);
    when(mockCreditCardServer.initTransaction(transaction)).thenReturn(true);
    when(mockCreditCardServer.pay(transaction, creditCard, 500)).thenReturn(false);
    when(mockTransactionProcessor.endTransaction()).thenReturn(true);

    paymentProcessor.processPayment(creditCard, Money.dollars(500));

    verify(mockCreditCardServer, times(1)).pay(transaction, creditCard, 500);
}
```

可以看到過度依賴 mock framework 了。太多假的行為，根本不曉得 transaction 有沒有真正成功，即測試結果可能不準確。這樣看起來好像有在測，但測試實際給予我們的信心卻很低，講白了就是**太假了**。因此這個測試案例可能無法測出真實的問題。

解決辦法就是不要寫單元測試，應該將它昇華到**整合測試**階段，雖然整合測試的成本較高，但結果也會比較貼近真實、有價值。


### **盡可能驗證回傳值**
如同前面幾個例子，用 `verify` 作為單元測試的驗證有很多需要注意的細節，所以盡量測那些有回傳值的行為，例如能用 `assertEquals`, `assertTrue` 等方式驗證。畢竟有回傳值，一翻兩瞪眼，是非分明。 

## **結語**
信任，是單元測試的基石。如果單元測試寫得不好，結果不準、時好時壞，久而久之大家都不會相信這個測試，甚至最後把它砍了，造成白忙一場、浪費大家時間，**不好的測試還不如不寫**。要寫出優秀的單元測試並不容易，本文提供了一些作法給大家參考，如果好好掌握這些技巧，相信就能讓單元測試更優秀。

### **References**
- [Software Engineering at Google: Lessons Learned from Programming Over Time](https://www.amazon.com/Software-Engineering-Google-Lessons-Programming/dp/1492082791)

## **更多你可能會感興趣的文章**
- [常見的 Interface 錯誤用法](/articles/anti-pattern-of-java-interface-impl-style)
- [如何提高程式碼的可測試性 (Testability)](/articles/testability)
- [軟體設計原則 YAGNI (You aren't gonna need it!)](/articles/testability)