---
layout: post
title: "使用 Log4j 解決 CSV 檔中文亂碼問題"
tagline: ""
author: "Kai-Sheng"
permalink: /articles/log4j-to-utf8-csv-for-excel
categories: [Java]
--- 

我最近需要用到 log4j 來記錄使用者的行為，並輸出成 UTF-8 csv，輸出完成後，用文字編輯器如 VS code 打開一切正常，但一使用 Excel 2016 時就會看到亂碼。
查閱資料後才發現原來 csv 檔的開頭沒有帶著 BOM (byte-order mark) ，導致 Excel 不知道要以 UTF-8 編碼來讀此 CSV 檔，因而出現亂碼。

### 解法1.  用 shell 指令在檔頭加入 BOM

這是一個很簡單粗暴的解法:

```shell
#!/usr/bin/env bash

#Add BOM to the new file
printf '\xEF\xBB\xBF' > with_bom.csv

# Append the content of the source file to the new file
cat source_file.txt >> with_bom.csv
```

網路上大部分的解法都是像這樣，是事後手動在檔頭補上 BOM，雖然簡單，但這不符合我的需求，我希望能一氣呵成。

所幸後來我在 stackoverflow 上發問後，有一個熱心網友給了答案:

### (推薦) 解法2. 在 header 裡加入 BOM

`CsvParameterLayout` 是 Log4j 推出的專門給 CSV 的 Layout 類型，只需要簡單的設定就能符合我的需求。

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

關鍵在於 `&#xFEFF;`，這個就是上述的 UTF-8 BOM，只要寫入後就能成功解決 Excel 中文亂碼問題。

---
### **Reference**

https://zh.wikipedia.org/zh-tw/%E4%BD%8D%E5%85%83%E7%B5%84%E9%A0%86%E5%BA%8F%E8%A8%98%E8%99%9F

https://stackoverflow.com/questions/3127436/adding-bom-to-utf-8-files

https://stackoverflow.com/questions/71943217/log4j2-write-to-csv-for-excel-without-garbled-characters/71944893#71944893