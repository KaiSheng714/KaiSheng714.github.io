layout: post
title:  
author: "Kai-Sheng"
permalink: /articles/kiss-principle
categories: [Design]
-----

Test stub
用來提供 `indirect input`，讓待測程式(SUT)可以走我們想要模擬的情境 (例如 valid/invalid)

```
Profile profile = Mockito.mock(Profile.class);
when(profile.getPassword("Jason)).thenReturn("12345");
```

這段程式的行為就稱作 stub
-----

Mock object 
前置驗證的物件，用來驗證測試程式的 `indirect output`。我們需要在執行 SUT 的程式前，先規定好此物件的行為。

----- 
Test spy 

後置驗證的物件，用來驗證測試程式的 `indirect output`。在 SUT 的程式執行完後，我們再去驗證這個物件與 SUT 有沒有如預期互動。例如範例中的

```
verify(notification, times(1)).send("some message");
```

這裡的 notification，稱作 test spy 

----- 
- 主流的 Mock Framework 為了遵循單元測試的 3A 原則 (Arrange, Act, Assert)，因此並不支援前置驗證，而且 Mockito 也只支援後置驗證。
- Mockito.mock 與 Mockito.spy 並非上述 test double 裡定義的 mock, spy。這是兩回事，雖然名字一樣，但請勿混為一談。

 