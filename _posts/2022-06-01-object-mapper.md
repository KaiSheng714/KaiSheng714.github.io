---
layout: post
title: "Jackson ObjectMapper 的錯誤用法"
author: "Kai-Sheng"
permalink: /articles/object-mapper
categories: [Java]
image: /assets/image/cover0.png
--- 

ObjectMapper 是一個相當實用的工具，它可以幫助我們完成 JSON 和 Object 之間的轉換。ObjectMapper 的應用非常廣泛，Spring Framework 的序列化/反序列化就預設使用 ObjectMapper，可見其影響力。但畢竟使用的人多，錯誤的寫法也就層出不窮，如果沒有按照正確做法，將很容易導致問題，本文將描述如何避免與改善。

 
## **問題描述**

你能看出這段程式碼有什麼問題嗎 ?

```java
public String toJson(Something something) throws JsonProcessingException {
    ObjectMapper objectMapper = new ObjectMapper();
    return objectMapper.writeValueAsString(something);
}
```

問題在於 `new ObjectMapper()`

這段程式碼是很經典的錯誤，而且這種錯誤隨處可見，很多人卻沒注意到。事實上，執行 `new ObjectMapper()` 是非常昂貴的，在系統遭遇高併發（High Concurrency）情況下，這種寫法很容易出現效能瓶頸。根據[這篇文章](https://theartofdev.com/2014/07/20/jackson-objectmapper-performance-pitfall/)的實驗，若每次序列化/反序列化都使用 `new ObjectMapper`，比起共用一個 ObjectMapper，執行時間至少相差五倍，因此要盡快修正。

---

## **解法**
解法很簡單，根據[官方文件](https://fasterxml.github.io/jackson-databind/javadoc/2.6/com/fasterxml/jackson/databind/ObjectMapper.html)指出，ObjectMapper 是 thread-safe，因此只要共用同一個實例，而不要每次都 `new` 即可，否則代價很高。我常用的作法有:
 
### **解法1. 宣告成員變數**

若你的 ObjectMapper 不需要任何 configure，其實 Spring 已經幫我們建好一個預設的，直接注入即可，當然這裡還是建議使用 Constructor Based Dependency Injection，可以看我寫的[這篇文章](/articles/analyzing-dependency-injection-patterns-in-spring) 

```java
@Service
public class MyService {

    private final ObjectMapper objectMapper;
    
    @Autowired
    public MyService(ObjectMapper objectMapper) {
        this.objectMapper = objectMapper;
    }

    public String toJson(Something something) throws JsonProcessingException {
        return objectMapper.writeValueAsString(something);
    }
}
```

ObjectMapper 強大的地方在於，它有很多參數可視需求設定。這種寫法可以在 new 的同時一起做 configure:

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

### **解法2. @Configuration**

如果你需要全域設定，或是有多個不同設定的 ObjectMapper，建議使用此方法，並且注入時要用 `@Qualifier`，否則將會注入預設的 ObjectMapper Bean。

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

### **解法3. 包裝成 Util**
這是我最常用的作法，我在專案中通常都只有一個 ObjectMapper，因此共用它就夠了，這時可包裝成 Util 方便全域使用。例外處理的部分，就依各專案需求而定，沒有最佳的設計，只有最適合自己的設計。

```java
public class JsonUtil {

    private final static ObjectMapper objectMapper = 
                new ObjectMapper()
                    .configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false);

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
 
## **結論**
盡量不要在每次序列化/反序列化使用時都 `new ObjectMapper();`，這樣的代價是昂貴的，比起共用同一個實例，兩者的效能可以相差很多倍。ObjectMapper 是 thread-safe 的物件，所以本文介紹的解法概念上是一樣的，就是請放心的`共用`同一個 ObjectMapper 實例。這是很重要的，雖然簡單但別小看它，也許一個小動作可以拯救你的一天。

### **References**
- [Should I declare Jackson's ObjectMapper as a static field?](https://stackoverflow.com/questions/3907929/should-i-declare-jacksons-objectmapper-as-a-static-field)
- [Jackson ObjectMapper performance pitfall](]https://theartofdev.com/2014/07/20/jackson-objectmapper-performance-pitfall/)