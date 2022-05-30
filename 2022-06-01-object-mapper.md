---
layout: post
title: "Jackson ObjectMapper 的錯誤用法"
author: "Kai-Sheng"
permalink: /articles/object-mapper
categories: [Java]
--- 

ObjectMapper 是一款相當受歡迎而且非常好用的工具，可以幫助我們完成 json 和 Java 的 Object 的互相轉換，ObjectMapper 的應用非常廣泛，所以錯誤的寫法也層出不窮，如果沒有按照 Best Practice，將容易導致問題，本文將描述如何改善。



![json](/assets/image/object-mapper.png?size=full)


使用 Jackson 前，需要在 pom.xml 加入 dependency，如果是使用 Spring boot，可以直接引用 `spring-boot-starter-web`，因為 Spring boot 執行序列化/反序列化預設使用 Jackson，從這點就不難看出 Jackson 的影響力:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

備註
- 序列化 (Serialize): 將 Object 轉成 json
- 反序列化 (Deserialize) : 將 json 轉成 Object
 
## **問題描述**

你能看出這段程式碼有什麼問題嗎 ?

```java
public String toJson(Something something) throws JsonProcessingException {
    ObjectMapper objectMapper = new ObjectMapper();
    return objectMapper.writeValueAsString(something);
}
```

答案是 `new ObjectMapper()`

這段程式碼這是很經典的錯誤，而且這種錯誤充斥在各處，很多人卻沒注意到。事實上，執行 `new ObjectMapper()` 是非常昂貴的，若系統流量較大，這種寫法很容易會出現效能瓶頸。根據[這篇文章](https://theartofdev.com/2014/07/20/jackson-objectmapper-performance-pitfall/)的實驗，如果每次序列化/反序列化都 `new ObjectMapper`，比起共用一個 ObjectMapper，執行時間至少相差五倍，因此要盡量修正這樣的程式。

---

## **解法**
解法很簡單，根據[官方文件](https://fasterxml.github.io/jackson-databind/javadoc/2.6/com/fasterxml/jackson/databind/ObjectMapper.html)指出，ObjectMapper 是 thread-safe，因此只要共用同一個實例，而不要每次都 `new` 即可。我常用的作法有:
 
### **方式1. 共用成員變數**

如果你要用預設的 ObjectMapper，其實 Spring 已經幫我們建好了，直接注入即可:

```java
@Service
public class MyService {

    private final ObjectMapper objectMapper;
    @Autowired
    public MyService(@ObjectMapper objectMapper) {
        this.objectMapper = objectMapper;
    }

    public String toJson(Something something) throws JsonProcessingException {
        return objectMapper.writeValueAsString(something);
    }
}
```

或是你要直接宣告並 config

```java
@Service
public class MyService {

    private static final ObjectMapper objectMapper =
         new ObjectMapper().configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false);

    public String toJson(Something something) throws JsonProcessingException {
        return objectMapper.writeValueAsString(something);
    }
}
```

### **方式2. Configuration**

如果你需要全域設定，或是有多種不同設定的 ObjectMapper，建議使用此方法，並且注入時要用 `@Qualifier`，否則會注入預設的 ObjectMapper Bean。

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
### **方式3. 包裝成 Util**

如果專案中只有一種 ObjectMapper 時，可以包裝成 Util 就可以很方便的全域使用。例外處理的部分，就依各個專案需求而定，沒有最佳的設計，只有最適合的設計，例外拋出給 caller 處理也是選項之一。`configure` 則視專案需求，若都不設定也是可以的。
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
String json = JsonUtil.toJson(something);
Something something = JsonUtil.toObject(json, Something.class);
```
 
### **References**
- https://stackoverflow.com/questions/3907929/should-i-declare-jacksons-objectmapper-as-a-static-field
- https://developer.51cto.com/article/706590.html
- https://theartofdev.com/2014/07/20/jackson-objectmapper-performance-pitfall/