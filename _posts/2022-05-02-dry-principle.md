---
layout: post
title: "軟體設計原則 DRY (Don't repeat yourself)"
author: "Kai-Sheng"
permalink: /articles/dry-principle
categories: [Design]
image: /assets/image/dry-principle.png
--- 

**DRY (Don't repeat yourself)**，是敏捷開發的核心設計原則之一。DRY 原則規定，對於每個**知識**點，系統中都只有一個明確而權威的表示。這個原則倡導單一事實來源(single source of truth) 的哲學，適用於所有的軟體開發工作，包含文件、設計、測試。但此原則常常被誤解，不少人認為只要兩個程式片段長得一樣就是違反了 DRY  原則。事實上，有些情況中的重複並不是一件壞事，甚至有些沒有重複的程式卻違反 DRY 原則。本文將探討 DRY 原則的運用情境。
 

![dry-principle](/assets/image/dry-principle.png?size=full)


## **違反 DRY 案例 1: copy & paste**

如果這麼做的**意圖**是因為偷懶，或是沒有正當理由的 copy & paste，其實就沒什麼好討論的了，確實違反 DRY 原則，應該盡快改善。常見的重構手段如 `Extract Method`, `Form Template Method`, `Extract Class`。如果是有理由的 copy & paste，重複一兩次也許能勉強接受，而到底需不需要重構，依然得依專案性質或團隊討論而定。例如，有個 class 或模組被3個專案引用，而引用的方式是 copy & paste，那到底該不該把這個 class 或模組抽出來呢? 這個問題沒有標準答案，但我認為事不過三，超過3個就是一個警示了，應該盡快移除重複，否則日後維護成本(技術債)會非常可觀。

## **違反 DRY 案例2: 知識的重複**

例如某購物網站，網站 VIP 會員享有9折優惠，因此有關於計算金額的部分，都需要套用此邏輯，例如購物車、訂單中的金額:

```java
public class Cart {
  // 計算購物車金額
  if (user.isVip()) {
    total = total * 0.9;
  }
  // ...
}
```
 
```java
public class Order {
  // 計算訂單金額
  if (user.isVip()) {
    total = total * 0.9;
  } 
  //...
}
```

很明顯，`網站 VIP 會員享有9折優惠` 這個**知識**重複了，如果這個計算邏輯重複出現在許多 class 中，就確實違反 DRY 原則。因為，如果有新需求要將9折修改為8折時，就得在專案中一一更改，甚至很可能漏改。因此，正確的做法是將此邏輯抽出 `calculateTotal` method，所有關於計算金額的地方都應該統一直接呼叫此 method:

```java
public int calculateTotal(User user, int total) {
  return user.isVip() ? total * 0.9 : total;
}
```

## **違反 DRY 案例3: 實作重複**

例如購物網站專案中，某功能是判斷購物車內是否有商品，但在專案裡的 `Cart`, `CartService` 出現兩個看起來相似，實際上卻相同的 method:

```java
// Cart
public boolean isEmpty() {
  return this.items.size() == 0;
}
```

```java
// CartService
public boolean isCartEmpty(Cart cart) {
  List<Item> items = cart.getItems();
  return items.isEmpty();
}
```

雖然名稱不同，但背後的**邏輯**、**意圖**是一模一樣的。這個問題很有可能是因為工程師之間因為沒有溝通好，導致他們都實作了重複的東西。除了事前充分討論，也可以透過 code reivew 來避免這種問題。

另一個經典例子就是 **自己造輪子**，有些常見的功能可以直接引用可靠、開源的第三方 library，不必再做一個一模一樣的東西。例如會員驗證功能，可以用 JWT, Spring Security 等方案，不需要自己再實作一個驗證功能。通常開源、可靠的 library 都是千錘百鍊、優化過的，自己再做一個同樣的東西，通常不會比較好，反而可能造出「方的輪子」。
 
## **未違反 DRY 案例:**

再次強調，DRY 原則並不是單純的消除重複的程式碼(Duplicated Code)，而是**知識**、**意圖**的重複。適度的重複程式碼，反而不是個壞事。例如訂單與購物車兩個類別 `Order`, `Cart` 有共同的屬性 `items`, `total`。
```java
public class Order {
  private List<Item> items;
  private int total;
  private LocalDatetime time;
  // ...
}

public class Cart {
  private List<Item> items;
  private int total;
  private boolean isExpired;
  // ...
}
```
有些人可能會將共同屬性抽出並建立父類別 `Data`:


```java
public class Data {  
  protected List<Item> items;
  protected int total;
}

public class Order extends Data {  
  private LocalDatetime time;
  // ...
}

public class Cart extends Data {  
  private boolean isExpired;
  // ...
}
```

這樣做雖然減少了重複程式碼，但這並不是 DRY 原則的最佳實踐，因為修改後的程式反而變得較不直觀，不能一目了然。在 DDD 的觀點，這兩個類別屬於不同的兩個 Entity，擁有不同的 domain knowledge，因此將他們獨立開會比較適合。

## **結論**
DRY 原則所指出的論點並不僅僅是程式碼的重複，更正確地來說是指**知識**、**意圖**上的重複。有些重複的程式，沒有違反 DRY 原則；有些不重複的程式，卻違反 DRY 原則。而盲目追尋 DRY 原則可能適得其反，導致閱讀程式時變得更不直觀。有些時候，我們不要被設計原則的名詞所蒙蔽了，其背後的思想才是我們真正該學習的。

## **References** 
- [Dry Revisited](https://enterprisecraftsmanship.com/posts/dry-revisited/)
- [DRY原則的誤區](http://www.yinwang.org/blog-cn/2015/06/14/dry-principle)
- 設計重構：25個管理技術債的技巧消除軟體設計臭味
