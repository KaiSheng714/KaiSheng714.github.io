---
layout: post
title: "使用 Log4j 輕鬆解決 CSV 檔中文亂碼問題"
tagline: ""
author: "Kai-Sheng"
permalink: /articles/log4j-to-utf8-csv-for-excel
categories: [Java]
--- 

我最近需要用到 log4j 做記錄日誌，並輸出成 UTF-8 csv 檔案。在輸出完成後，用文字編輯器如 VS code 打開一切正常，但使用 Excel 2016 時就會看到亂碼。查閱資料後才發現原來 csv 檔的開頭沒有帶著 BOM (byte-order mark)，導致 Excel 不知道要以 UTF-8 編碼格式讀取，因而出現亂碼。

![log4j-to-utf8-csv-for-excel](/assets/image/log4j.png?size=medium)

------

###  **解法1. 用 shell 指令在檔頭加入 BOM**

這是一個很簡單粗暴的解法:

```shell
#!/usr/bin/env bash

#Add BOM to the new file
printf '\xEF\xBB\xBF' > with_bom.csv

# Append the content of the source file to the new file
cat source_file.txt >> with_bom.csv
```

網路上大部分的解法都是像這樣，是事後透過指令或程式手動在檔頭補上 BOM。雖然簡單，但這不符合我的需求，我希望能使用 log4j 一氣呵成。

研究無果後，所幸我在 stackoverflow 上發問，有一位熱心網友給了答案:

### **解法2. 在 header 裡加入 BOM (推薦)**

`CsvParameterLayout` 是 Log4j 推出的專門給 CSV 的 Layout 類型，只需要簡單的設定就能輸出成 csv 檔。

```xml
<RollingFile 
  name="Chat-Appender" 
  fileName="output.csv">
  <CsvParameterLayout 
      delimiter="," 
      format="Excel"
      header="&#xFEFF;id,question,answer\n"
      charset="UTF-8" />
</RollingFile>
```

至於解決 CSV 檔中文亂碼問題呢? 關鍵在於 `&#xFEFF;`，這個就是上述的 UTF-8 BOM。利用 Log4j 原有的 header 屬性，借力使力輕鬆解決 Excel 開啟 CSV 中文亂碼問題。

---
### **Reference**

[Byte order mark](https://en.wikipedia.org/wiki/Byte_order_mark)

[Adding BOM to UTF-8 files](https://stackoverflow.com/q/3127436/5485454)

[Log4j2 write to CSV for Excel without garbled characters](https://stackoverflow.com/q/71943217/5485454)