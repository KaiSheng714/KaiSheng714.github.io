---
layout: post
title: "軟體設計原則 DRY (Don't repeat yourself)"
meta: design,clean-code
author: "Kai-Sheng"
permalink: /articles/dry-principle
categories: [Design]
--- 

Don't repeat yourself，DRY 的原則是許多軟體開發者的開發準則，DRY 旨在消除程式中的重複，且必須能夠表達所應表達的內容。但此原則常常被誤解，不少人認為只要兩個程式片段長得一樣就是違反了 DRY 原則。事實上，有些情況中，重複並不是一件壞事，甚至有些沒有重複的程式卻違反了 DRY 原則。本文將探討 DRY 原則的運用情境。

---

![dry-principle](/assets/image/dry-principle.png)


## 違反 DRY 案例1: copy & paste

如果是因為偷懶，或是沒有正當理由的 copy & paste，其實就沒什麼好討論的了，確實違反 DRY 原則。應該盡快改善。常見的重構手段如 `Extract Method`, `Form Template Method`, `Extract Class`。如果是有理由的 copy & paste，重複一兩次也許能勉強接受，實際上需不需要重構得依專案性質或團隊討論而定。

## 違反 DRY 案例2: 知識的重複

例如某購物網站，網站 VIP 會員享有9折優惠，因此有關於計算金額的部分，都需要套用此邏輯，例如購物車、訂單:

```java
...
public class Cart {
  ...
  // 計算購物車金額
  if (user.isVip()) {
    amount = amount * 0.9;
  }
  ...
}
 

public class Order {
  ...
  // 計算訂單金額
  if (user.isVip()) {
    amount = amount * 0.9;
  }
  ...
}

```

很明顯，`網站 VIP 會員享有9折優惠` 這個知識重複了，如果這個計算邏輯重複出現在許多 class 中，就明顯違反 DRY 原則。因為，如果新需求要將9折修改為8折時，就得一一更改，甚至很可能漏改。因此，正確的做法是將此邏輯抽出 `calculateAmount` method，所有關於計算金額的地方都應該統一直接使用此 method:

```java
public int calculateAmount(User user, int amount) {
  return user.isVip() ? amount * 0.9 : amount;
}
```

## 違反 DRY 案例3: 實作重複

例如專案中出現兩個 method:

```java
public int calculateAmount(User user, int amount) {
  return user.isVip() ? amount * 0.9 : amount;
}

public int getAmountForVip(User user, int amount) {
  return user.isVip() ? amount * 0.9 : amount;
}

```
雖然名稱不同，但背後的邏輯和知識是一模一樣的。這個問題很有可能是因為工程師之間因為沒有溝通好，導致他們都實作了重複的東西。除了事前充分討論，也可以透過 code reivew 來避免這種問題。

另一個經典例子就是 **自己造輪子**，有些常見的功能可以直接引用可靠、開源的第三方 library，不必再做一個一模一樣的東西。例如會員驗證功能，可以用 JWT, Spring Security 等方案，不需要自己再實作一個驗證功能。通常開源、可靠的 library 都是千錘百鍊、優化過的，自己再做一個同樣的東西，通常不會比較好，反而可能造出「方的輪子」。


## 違反 DRY 案例4: 執行重複

例如專案中出現兩個


## 未違反 DRY 案例 :

再次強調，DRY 原則並不是單純的描述重複的程式碼(Duplicated Code)，而是知識的重複。適度的重複程式碼，反而不是個壞事。例如兩個類別 `Student`, `Teacher` 有共同的屬性 `name`, `age`，有些人可能會將 `name`, `age` 抽出並建立父類別 `Person`，這樣做雖然減少了重複程式碼沒錯，但這並不是 DRY 原則的最佳實踐，但也許造成較程式變得較不直觀，不能一目了然。

```java
public class Order {
  private String id;
  private int amount;
  private int amount;
}

public class Item {
  private String id;
  private String name;
  private int price;

}
```

```java
public class Person {
  protected String name;
  protected int age;
}

public class Teacher extends Person {
  private int salary;
}

public class Student extends Person {
   private int score;
}
```

## 結論
DRY 原則所指出的論點並不僅僅是程式上的重複，更正確地來說是指知識上的重複，而盲目追尋 DRY 原則很可能適得其反。

### **References**
 https://enterprisecraftsmanship.com/posts/dry-revisited/
 http://www.rocidea.com/one?id=33839
