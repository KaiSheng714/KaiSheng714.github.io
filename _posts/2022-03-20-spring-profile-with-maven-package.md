---
layout: post
title: "Spring + Maven + IntelliJ 多環境 (Profile) 整合技巧"
tagline: "一個更容易開發、部署 Spring Appication 的方式"
meta: java,spring,maven,intellij,profile
author: "Kai-Sheng"
---

在 Spring 專案中，profile 是用於區分各種環境的，例如本機環境、開發環境、測試環境、正式環境等等。

你是否曾在 release day 當天，才在打包好的 war 檔中，手動調整/替換成正式環境的 application.yml / application.properties 呢 ?

又或者，整份專案只有正式環境使用的 .yml / .properties ，每當你在開發時都要自己生一份屬於本機開發的 .yml / .properties，而且還要小心翼翼的不要放到 git 上呢 ?

其實有一個更好的方式可以減少這些不必要的人工步驟，從開發到部署，透過指令就可以輕易完成。

![Spring + Maven + IntelliJ profile integration 多環境 (Profile) 整合技巧](https://miro.medium.com/max/1400/1*oXVJyfl-jV39yo8hEUdYnw.png?style=center) 

## 1. 開發階段 — 準備所有 application.properties

首先準備好所有需要的 .properties / .yml，根據自己的需求而定。例如開發環境 (dev) 、正式環境 (prod) 等，以及一個主要的 application.properties。

application.properties 只有一行 **spring.profiles.active=@activeProfile@**

![Spring + Maven + IntelliJ profile integration 多環境 (Profile) 整合技巧](https://miro.medium.com/max/836/1*JBGOY1prMFtvnT6z65nowg.png?style=center)

開發環境與正式環境，根據你的需求而定，本文以 server.port 為例。在開發環境中，我希望使用 8888 port ，而在正式環境中使用 9999 port。

![Spring + Maven + IntelliJ profile integration 多環境 (Profile) 整合技巧](https://miro.medium.com/max/486/1*NRwTGDKFcfH1qqMbErg_BQ.png?style=center)


![Spring + Maven + IntelliJ profile integration 多環境 (Profile) 整合技巧](https://miro.medium.com/max/476/1*_OWBIOG2g0p4QjWgSgRgKg.png?style=center)

準備完成後，專案結構會像這樣:

![Spring + Maven + IntelliJ profile integration 多環境 (Profile) 整合技巧](https://miro.medium.com/max/686/1*PP5qtWuk8PhAT28dhSriTQ.png?style=center)

接著在 pom.xml 加入以下程式碼:

```xml
  <profiles>
    <profile>
      <id>dev</id>
      <activation>
        <activeByDefault>true</activeByDefault>
      </activation>
      <properties>
        <activeProfile>dev</activeProfile>
      </properties>
    </profile>
    <profile>
      <id>prod</id>
      <properties>
        <activeProfile>prod</activeProfile>
      </properties>
    </profile>
  </profiles>
```

## 2. 開發階段 — 設定 IntelliJ

在開發的階段，我們必須告訴 IDE ，請它套用適合本機開發環境的 profile。設定 Run Configuration， ‘Active profiles’ 填入 dev

![Spring + Maven + IntelliJ profile integration 多環境 (Profile) 整合技巧](https://miro.medium.com/max/1400/1*9P0Rqnf7iBG5cYfZ1KhOug.png?style=center)

按下 Run 之後， IntelliJ 就會使用 ‘dev’ profile 執行 Spring boot 程式，程式成功在 port 8888 執行。

![Spring + Maven + IntelliJ profile integration 多環境 (Profile) 整合技巧](https://miro.medium.com/max/1400/1*Qjynz8QB7K9B5Ll0B72lwg.png?style=center)

## 3. 部署階段 - Maven

當我們開發完成，利用 mvn package / install 指令來打包時，需加入 -P 參數，告訴 maven 幫我們將 application.properties 的 '**@activeProfile@' 字串替換成 ‘prod’，**指令如下:

```
mvn clean package -P prod
```

我們將打包好的 war 檔打開來看，可以發現 3 個 .properties 都已經打包進來

![Spring + Maven + IntelliJ profile integration 多環境 (Profile) 整合技巧](https://miro.medium.com/max/1332/1*OUI5SPrkPe25QTmYGVTTPw.png?style=center)

接著打開 **application.properties**

![Spring + Maven + IntelliJ profile integration 多環境 (Profile) 整合技巧](https://miro.medium.com/max/1080/1*pJ6eNigl9qnrD9XbmWKRpQ.png?style=center)

可以看到 **@activeProfile@** 已經自動被替換成 **prod** 了。

接著再執行我們的 war 檔

```
java -jar <my-artifact>.war
```

可以看見 Spring boot 如我們預期的使用 prod profile .

![Spring + Maven + IntelliJ profile integration 多環境 (Profile) 整合技巧](https://miro.medium.com/max/1400/1*Kos3ZVHzgHSyi59zqtJ8ig.png?style=center)

### 後記

透過本文介紹的方式，可以讓開發人員輕鬆的在任何環境中套用 Spring 專案中的 .properties / .yml，不需要再根據環境來手動調整。當然也可以將這方法套用在 jenkins 來做 CI/CD ，在日後千百次的打包/執行/部署的過程中，能替我們節省許多成本。

**幫團隊節省成本，就是替幫公司賺錢。**

本文範例完整程式碼 [https://github.com/KaiSheng714/spring-maven-profile-integration-demo](https://github.com/KaiSheng714/spring-maven-profile-integration-demo)
