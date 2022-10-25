---
layout: post
title: "使用 Log4j2 輸出 CSV 檔，並輕鬆解決 Excel 中文亂碼問題"
tagline: ""
author: "Kai-Sheng"
permalink: /articles/log4j2-to-utf8-csv-for-excel
categories: [Java]
image: /assets/image/site-image-small.png
--- 

我最近在 Java 專案中使用 Log4j2 做即時日誌並輸出 CSV，輸出完成後，文字編輯器打開一切正常，但以 Excel 開啟時卻出現亂碼，明明是 UTF-8，怎麼還會有亂碼呢? 查資料後才發現原來 CSV 的檔頭沒有帶著 BOM (byte-order mark) 導致。本文介紹兩種解決辦法。

![log4j-to-utf8-csv-for-excel](/assets/image/log4j.png?size=medium)
 
##  **解法1. 用 shell 指令在檔頭加入 BOM**

網路上大部分的解法都是像這樣，以事後透過指令或程式手動在檔頭補上 BOM。如下所示，以下這段指令是先將 BOM 碼寫入一個空的檔案內，再將 CSV 資料倒入這個檔案中，這方法看似簡單卻又帶有設計巧思。

```shell
#!/usr/bin/env bash

#Add BOM to the new file
printf '\xEF\xBB\xBF' > with_bom.csv

# Append the content of the source file to the new file
cat source_file.csv >> with_bom.csv
```

雖然簡單，但這並不符合我的需求，這種解法需要多一個處理步驟，但我希望能**使用 Log4j2 一氣呵成**。

在研究無果後，我只好在 Stackoverflow 上發問，所幸有一位熱心網友給了答案:

## **解法2. 在 header 裡加入 BOM (推薦)**

`CsvParameterLayout` 是 Log4j2 推出的專門給 CSV 的 Layout 類型，只需要簡單的設定就能輸出成 CSV 檔。Log4j2 版本為 2.17.0，以下是我的 log4j2.xml 程式片段:

```xml
<RollingFile 
  name="Chat-Appender" 
  fileName="output.csv">
  <CsvParameterLayout 
      delimiter="," 
      format="Excel"
      header="id,question,answer\n"
      charset="UTF-8" />
</RollingFile>
```

至於如何解決 CSV 檔中文亂碼問題呢？ 關鍵在於 `header` 欄位需加入 `&#xFEFF;`:

`header="&#xFEFF;id,question,answer\n"`

`&#xFEFF;` 就是上述的 UTF-8 BOM，而此解法最大的優點在於: 利用 Log4j2 `CsvParameterLayout` 原有的 header 屬性，借力使力，輕鬆解決 Excel 開啟 CSV 中文亂碼問題。
 

---
### **References**
- [Byte order mark](https://en.wikipedia.org/wiki/Byte_order_mark)
- [Adding BOM to UTF-8 files](https://stackoverflow.com/q/3127436/5485454)
- [Log4j2 write to CSV for Excel without garbled characters](https://stackoverflow.com/q/71943217/5485454)

### **更多你可能會感興趣的文章**
- [如何寫出優秀的單元測試 (Best Practice)](/articles/good-unit-test)
- [常見的 Interface 錯誤用法](/articles/anti-pattern-of-java-interface-impl-style)