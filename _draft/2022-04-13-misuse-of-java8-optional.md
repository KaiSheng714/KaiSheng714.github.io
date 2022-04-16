---
layout: post
title: "Java 8 Optional 常見的錯誤用法"
tagline: ""
meta: design,clean-code,java
author: "Kai-Sheng"
permalink: /blog/misuse-of-java8-optional
categories: [Design, Java]
--- 


https://stackoverflow.com/questions/71856929/i-want-to-return-a-exception-while-my-return-type-is-dto

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
