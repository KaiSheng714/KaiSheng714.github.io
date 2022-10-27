---
layout: post
title: "Java Lombok @Builder "
author: "Kai-Sheng"
permalink: /articles/lombok-builder
categories: [Java]
image: /assets/image/lombok-cover.png
--- 


## **我常用的方式**
綜上所述，我認為 Lombok 還是最適合用於 Entity,Model，這樣用就足以應付大部分的狀況了：

```java
@Log4j2
@Data
@Accessors(chain = true)
public class Student {
    private int id;
    private String name;
    private String email;
    // field ...
    // method ...
}
```

在 class 標註 `@Accessors(chain=true)` 後就能鏈式串接所有 setter，使用上相當方便：

```java
Student student = new Student()
    .setId(12345)
    .setName("KaiSheng")
    .setEmail("KaiSheng714@github.com");
```

可能有些人會問我：用 `@Builder` 不是也差不多嗎？但我的經驗是，因為我有時會用 Factory 來建立物件，這時如果在工廠裡再用 builder pattern 反而有點多此一舉，此外，我的方式不需要額外呼叫 `builder()`, `build()`，也可以和 Jackson 正常配合。

另外，標註 `@Log4j2` 或 `@Slf4j`，就可以使用 log 功能，非常方便，適用於任何 class，通常建議是需要時再標註上去即可。



因此應盡量按規格在必填的 field 加上 Lombok 的 `@NonNull`：如果沒有賦值，則立即拋出 NullPointerException，fail fast。又或者可以透過重構手段，**讓 class 變小一點**，減少開發者忘記賦值的機率。

備註: `@NonNull` 僅適用於非 primitive type。


### **References**
- [Be Careful With Lombok](https://levelup.gitconnected.com/be-careful-with-lombok-2e2edfc01110)
- [5 Tips For Using Lombok In Production](https://dzone.com/articles/5-tips-for-using-lombok-in-production)

### **更多你可能會感興趣的文章**
- [軟體設計原則 Tell, Don't Ask](/) 
- [Java Jackson ObjectMapper 教學與注意事項](/articles/object-mapper)
- [分析 Spring 的依賴注入模式](https://kaisheng714.github.io/articles/analyzing-dependency-injection-patterns-in-spring)