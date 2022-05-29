---
layout: post
title: "ObjectMapper 的錯誤用法"
author: "Kai-Sheng"
permalink: /articles/object-mapper
categories: [Java]
--- 


 
ObjectMapper類是Jackson庫的主要類。它提供一些功能將轉換成Java對象匹配JSON結構，反之亦然。它使用JsonParser和JsonGenerator的實例實現JSON實際的讀/寫。



![object-mapper](/assets/image/object-mapper.png?size=full)


 ObjectMapper 是一款非常好用的 json 轉換工具，可以幫助我們完成 json 和 Java 的 Object 的互相轉換

什麼是 Serialize 和 Deserialize？
Serialize : 將 Java Object 轉換成 json
Deserialize : 將 json 轉換成 Java Object
在 Spring Boot 裡使用 ObjectMapper
ObjectMapper 是由 Jackson library 所提供的一個功能，所以只要在 maven 中加入 spring-boot-starter-web 的 dependency 就可以了

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>


```java
public String getCarString(Car car){
    ObjectMapper objectMapper = new ObjectMapper();
    String str = objectMapper.writeValueAsString(car);
    return str;
}
```
 