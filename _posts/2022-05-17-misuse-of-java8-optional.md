---
layout: post
title: "多此一舉! 不要這樣用 Java 8 Optional"
tagline: ""
author: "Kai-Sheng"
permalink: /articles/misuse-of-java-8-optional
categories: [Design, Java]
image: /assets/image/site-image-small.png
--- 

我們都知道盡量不要回傳 null，而 Java 8 新加入了 Optional 類別可避免 NullPointerException 問題與繁瑣的 null check，可以讓程式邏輯看起來更簡潔、易讀，也能清楚表達 method 可能沒有結果值。但我卻看到了不少錯誤的用法，反而讓 Optional 顯得多此一舉。本篇探討這些錯誤的用法，以及如何正確使用。
 

 
## **錯誤1. isPresent() and get()**
假設有一個 `studentService` 可利用 id 查詢學生資料，我們為了避免 return null 而後續可能導致 NPE，我們就必需在 `studentService.readById` 回傳結果時先做 null check，因此傳統寫法會像這樣:

```java
public Student readById(String id) {
    Student student = studentService.readById(id);
    if (student != null) {
        return student;
    } else {
        throw new NotFoundException(id); 
    }
}
```
若改成 `Optional` 寫法，並將 `studentService.readById` 改為回傳 `Optional<Student>` 後，有些人可能會寫成這樣 :

```java
public Student readById(String id) {
    Optional<Student> student = studentService.readById(id);
    if (student.isPresent()) {
        return student.get();
    } else {
        throw new NotFoundException(id); 
    }
}
```

很不幸的是，**這應該是最常見的錯誤用法了**，我們不難發現上面的 `isPresent()`, `get()` 和傳統寫法本質上是一樣的，且增加了不必要的複雜度，可謂多此一舉。

正確使用 Optional 方式如下:
 
```java
public Student readById(String id) {
    return studentService.readById(id).orElseThrow(() -> new NotFoundException(id));
}
```

`orElseThrow` 會判斷 Optional 的內容，若有值時則直接回傳 Student；若沒有，則拋出例外。其實 Optional 是與 Java 8 functional programming 寫法相輔相成的，所以使用 Optional 時應搭配如 `filter()`, `map()`, `orElseThrow()` 等的 functional programming 風格的寫法會比較適合。

## **錯誤2. 一定有值，卻依然使用 Optional**
Optional 設計的意義就是用來表示 method 的回傳值可能會是空的。但在某些**一定會有回傳值**情況下，開發者卻依然使用 Optional，這就造成了過度包裝與多此一舉。承上學生系統的例子，假設我們要查詢全體學生中的第一名:
```java
public Optional<Student> readTopScoreStudent() {
    // ...
}
```
正常來說，這個系統並不會沒有學生資料（否則一切都是空談），因此這個 method 肯定會有回傳值，不需使用 Optional。

## **錯誤3. 作為參數**
有些人會將 Optional 作為參數，意圖表示這個參數可能是非必要的:

```java
public void setName(Optional<String> name) {
    if (name.isPresent()) {
        this.name = name.get();
    } else {
        this.name = "無名氏";
    }
}
```

但這是個不好的寫法，因為這裡的 `Optional<String>` 參數有三種可能的值:
1. 有內容值的 Optional 
2. Optional.empty()
3. null

**Optional 也有可能是個 null**，當然有機會引發 NPE，讓人更摸不著頭緒，因此請不要使用 Optional 作為參數。此外，這樣的寫法會讓 caller 很痛苦，因為他們必需將參數多包一層 Optional，變得不容易使用:

```java
setName(Optional.of("Jason"));
setName(Optional.empty());
```

因此，比較好的設計是透過 `overloading`，讓參數有值或沒有值的意圖與結果更加明確：

```java
public void setName() {
    this.name = "無名氏";
}

public void setName(String name) {
    this.name = name;
}
```

-------

另外，有此一說，`Optional` 若作為 Spring controller 的參數，則更能表達該參數是非必要的，例如: 

```java
@RequestMapping (value = "/submit/id/{id}", method = RequestMethod.GET, produces="text/xml")
public String showLoginWindow(@PathVariable("id") String id,
                              @RequestParam("username") Optional<String> username,
                              @RequestParam("password") Optional<String> password) { ... }
```

在 Spring 4.1.1 後已經可以妥善處理這裡的 Optional，它將不會是 null，再加上它是 controller，所以也不會難以被呼叫，因此有些人覺得這種作法比較好，這就見仁見智了。

## **錯誤4. 作為 class field**

```java
public class Student {

    private Optional<String> name;
    // ...
}
```

因為 Optional 是設計用來 method 的回傳型態，因此它並沒有實作序列化 `Serializable` 介面，在特定狀況下(如網路傳輸)需要物件序列化時將會出現問題。

## **錯誤5. Collection and Optional**
因為 `Optional` 本身就是一個容器，如果內容又是另一個容器，例如回傳 `Optional<List<Student>>`，這不僅比較複雜以外，在語意上還代表著三種可能的回傳值:
1. 一個有內容的 List
2. 一個空的 List
3. Optional.empty()

這樣容易造成程式複雜與混淆，比較好的方式是：如果真的沒有回傳值，那就回傳一個**空的容器**就好了：

```java
public List<Student> readAllStudentsInClass(String classId) {
    // ... 
    return result.isEmpty() ? Collections.emptyList(): new ArrayList<>(result);
}
```
 
## **錯誤6. Map and Optional**
不要將 Optional 放入 Map，例如 `Map<String, Optional<Student>>`，原因和上述類似，在呼叫 `map.get(key)` 的回傳值會是:
1. Optional (可能有 Student 與可能沒有 Student)
2. null

像這種錯誤用法都會提高不必要的複雜性。

## **References**
- [java-8-optional-use-cases](http://dolszewski.com/java/java-8-optional-use-cases/)
- [@RequestParam in Spring MVC handling optional parameters](https://stackoverflow.com/questions/22373696/requestparam-in-spring-mvc-handling-optional-parameters)
- [RSPEC-3553](https://rules.sonarsource.com/java/tag/clumsy/RSPEC-3553)
- Item 55: Return optionals judiciously (Effective Java 3rd)

## **更多你可能會感興趣的文章**
- [常見的 Interface 錯誤用法](/articles/anti-pattern-of-java-interface-impl-style)
- [SimpleDateFormat 的錯誤用法](/articles/simple-date-format)