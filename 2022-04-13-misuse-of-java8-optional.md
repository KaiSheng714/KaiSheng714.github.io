---
layout: post
title: "[Java] 不要這樣使用 Optional"
tagline: ""
meta: design,clean-code,java
author: "Kai-Sheng"
permalink: /articles/do-not-java-optional
categories: [Design, Java]
--- 



![java8-optional](/assets/image/optional.png?size=small)

## 1. 用在參數

```java
public int calculateSomething(Optional<String> p1, Optional<BigDecimal> p2) {
    // my logic
}
```


## 2. 用在 if

```java
 if (p2.isPresent()) {
   
 }
```


### **Reference**

https://stackoverflow.com/questions/71856929/i-want-to-return-a-exception-while-my-return-type-is-dto