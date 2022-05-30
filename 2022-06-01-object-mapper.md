---
layout: post
title: "Jackson ObjectMapper 的錯誤用法"
author: "Kai-Sheng"
permalink: /articles/object-mapper
categories: [Java]
--- 

ObjectMapper 類是 Jackson 庫的主要類。它提供一些功能將轉換成 Java 對象匹配JSON結構，反之亦然。它使用JsonParser和JsonGenerator的實例實現JSON實際的讀/寫。
市面上用於在 Java 中解析 Json 的第三方庫，隨便一搜不下幾十種，其中的佼佼者有 Google 的 Gson， Alibaba 的 Fastjson 以及本文的 jackson。三者不相伯仲，隨著掌握一個都能滿足項目中的 json 解析操作，因為 Spring Boot Web 組件默認使用的是  jackson，所以掌握 Jackjson 是非常有必要的

要想使用 Jackson，需要在 pom.xml 文件中添加 Jackson 的依賴。


![json](/assets/image/object-mapper.png?size=full)

ObjectMapper 是一款非常好用的 json 轉換工具，可以幫助我們完成 json 和 Java 的 Object 的互相轉換

什麼是 Serialize 和 Deserialize？
Serialize : 將 Java Object 轉換成 json
Deserialize : 將 json 轉換成 Java Object
在 Spring Boot 裡使用 ObjectMapper

如果是使用 Spring boot，可以使用這包就可以直接使用 Jackson

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```
 
## 問題

下面一段程式碼這是很經典的錯誤，這種錯誤充斥在各處，而且很多人沒有注意到

```java
public String toJson(Something something) throws JsonProcessingException {
    ObjectMapper objectMapper = new ObjectMapper();
    return objectMapper.writeValueAsString(something);
}
```

執行 `new ObjectMapper()` 是非常昂貴的，如果系統流量大，就會出現效能瓶頸。根據[這篇](https://theartofdev.com/2014/07/20/jackson-objectmapper-performance-pitfall/) 的實驗，如果每次序列化/反序列化都 `new ObjectMapper`，比起共用一個 ObjectMapper，執行時間至少相差五倍。因此要盡量修正這樣的程式。

---

根據[官方文件](https://fasterxml.github.io/jackson-databind/javadoc/2.6/com/fasterxml/jackson/databind/ObjectMapper.html)， ObjectMapper 在 multi-thread 環境下是安全的，因此最好是共用同一個實例

我常用的作法有
 
## 方式1. 設定 Bean

如果需要全域設定而且有多種 ObjectMapper，建議使用此方法

```java
@Configuration
public class JacksonConfiguration {

    @Bean("customObjectMapper")
    public ObjectMapper customObjectMapper() {
        return new ObjectMapper()
            .configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false)
            .configure(DeserializationFeature.FAIL_ON_INVALID_SUBTYPE, false);
    }

}
```

再注入需要使用的 service 就可以了。值得注意的是，因為 Spring boot 已經是先幫我們建立好預設的 ObjectMapper，所以如果不使用 `@Qualifier` 注入，則會注入 Spring boot 所建立的 ObjectMapper Bean

```java
@Service
public class MyService {

    private final ObjectMapper objectMapper;

    @Autowired
    public ServiceClassA(@Qualifier("customObjectMapper") ObjectMapper objectMapper) {
        this.objectMapper = objectMapper;
    }

}
```
 
## 方式2. 包裝成 Util

如果需要全域只有一種 ObjectMapper 時，可以包裝成 Util，方便使用。例外處理的部分，就依各個專案需求而定。沒有最佳的設計，只有最適合的設計。例外拋出給 caller 處理也是選項之一`configure` 可依個人設定，若都不設定也是可以的，ObjectMapper 將會套用預設值，其實就能符合大部分的需求。

```java
public class JsonUtil {

    private final static ObjectMapper objectMapper = 
                new ObjectMapper()
                    .configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false)
                    .configure(DeserializationFeature.FAIL_ON_INVALID_SUBTYPE, false);

    private JsonUtil() {

    }

    public static String toJson(Object obj) {
        try {
            return objectMapper.writeValueAsString(obj);
        } catch (JsonProcessingException e) {
            log.error("Occur error during parsing data to json: {}", obj, e);
            return "";
        }
    }

    public static <T> T toObject(String json, Class<T> objectClass) {
        try {
            return objectMapper.readValue(json, objectClass);
        } catch (IOException e) {
            log.error("Occur error during mapping json to an object: {}", json, e);
            return null;
        }
    }
}
```

使用起來非常方便簡單，封裝性也較佳。

```java
String studentJson = JsonUtil.toJson(student);
Student student = JsonUtil.toObject(studentJson, Student.class);
```
 
### **References**


- https://stackoverflow.com/questions/3907929/should-i-declare-jacksons-objectmapper-as-a-static-field
- https://developer.51cto.com/article/706590.html
- https://theartofdev.com/2014/07/20/jackson-objectmapper-performance-pitfall/