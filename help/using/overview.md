---
title: HTL 概觀
description: 了解 AEM 如何支援 HTL (HTML 範本語言) 以提供可增強安全性的高效企業級 Web 框架。對於不具備 Java 知識的 HTML 開發人員，該框架能讓他們更輕鬆參與 AEM 專案。
exl-id: 5d06ff25-d681-4b95-8375-c28a8364eb7e
source-git-commit: addc69e4b4e56a9b1c5f91ce9af26fa2d326d981
workflow-type: tm+mt
source-wordcount: '677'
ht-degree: 98%

---


# 概觀 {#overview}

>[!TIP]
>
>**您是否考慮過適用於 AEM 的 Edge Delivery Services？**
>
>現有的專案可以繼續使用本文件中所述的方法。不過，若是新專案，Adobe建議使用[Edge Delivery Services.](https://experienceleague.adobe.com/zh-hant/docs/experience-manager-cloud-service/content/edge-delivery/overview)

HTML Template Language (HTL) 由 Adobe Experience Manager (AEM) 支援，旨在提供高生產力的企業級網頁框架，同時提升安全性。對於不具備 Java 知識的 HTML 開發人員，此語言還可讓他們更輕鬆參與 AEM 專案。

[在 AEM 6.0 引進的](history.md) HTML 範本語言是 AEM 中首選且推薦使用的 HTML 伺服器端範本系統。對需要建立強大企業網站的網頁開發人員而言，HTML 範本語言有助於提升安全性和開發效率。

## 提升安全性 {#increased-security}

HTML 範本語言 (HTL) 會自動將內容感知轉義套用於所有輸出變數來加強網站安全性，使其比大多數其他範本系統更安全。HTL 之所以能讓這個方法更安全，是因為其了解 HTML 語法，並運用相關知識，根據運算式在標記中的位置調整其所需的跳脫處理。這個方法會造成放在 `href` 或 `src` 屬性中的運算式之跳脫處理方式，與放在其他屬性或其他地方的運算式不同。

雖然使用 JSP 等範本語言可以達到相同結果，但開發人員必須手動操作，才能確保適當的逸出套用至每個變數。由於套用的逸出只要一有遺漏或錯誤，就可能足以造成跨網站指令碼 (XSS) 出現弱點，因此 Adobe 決定使用 HTL 將此工作自動化。如有必要，開發人員仍可在運算式上指定其他逸出，但使用 HTL 的話，預設行為更可能對應到希望的行為，減少發生錯誤的機率。

## 簡化開發工作 {#simplified-development}

HTML 範本語言簡單易學，其功能刻意有所限制，以確保其簡單明瞭。此外，這種範本語言也有強大機制足以建立標記架構及叫用邏輯，同時一律強制分割標記和邏輯面向。HTL 是標準的 HTML5，使用表達式和資料屬性來註解具有動態行為的標記。這種方法保持了標記的有效性和可讀性。評估運算式和資料屬性的工作完全在伺服器端進行，用戶端看不見，因此用戶端可以不受干擾地使用想要的 JavaScript 框架。

對於不具備 Java 知識的 HTML 開發人員，這些功能可讓他們編輯 HTL 範本、整合至開發團隊，並簡化與全端 Java 開發人員的協作。換個角度來說，這能讓 Java 開發人員專注於後端程式碼，不必擔心 HTML。

## 降低成本 {#reduced-costs}

提高安全性、簡化開發過程更以及改進團隊共同作業，促使執行 AEM 專案可以事半功倍，上市時間更短，而且降低了總體擁有成本 (TCO)。

以 HTML 樣板語言重新建置 Adobe.com 網站後顯示，專案成本與時程最多可降低約 25%。

![提高效率並降低成本](assets/chlimage_1.png)

上圖說明 HTL 提升效率的下列成效:

* **HTML / CSS / JS：** HTML 開發人員可以直接編輯 HTL 範本，讓前端設計直接在 AEM 元件上實施，而無需單獨實施。這個方法可減少痛苦的迭代作業，裨益全端 Java 開發人員。
* **JSP/HTL:** HTL 本身不需搭配任何 Java 知識，且容易撰寫，因此所有具備 HTML 專業的開發人員都能編輯這類範本。
* **Java:** HTL 提供清楚且易於使用的 Use-API，讓具商業邏輯的介面更清晰明瞭，同時有助於整體 Java 開發。

## 影片介紹 {#video}

以下影片取自 [AEM Gems 研討會](https://experienceleague.adobe.com/zh-hant/docs/events/experience-manager-gems-recordings/gems2014/aem-introduction-to-htl)，概要說明 HTL 的用途以及實施範例。

>[!VIDEO](https://video.tv.adobe.com/v/19504/?quality=9)

請注意，影片中提到 HTL 時，均是使用[其以前的名稱 Sightly](history.md)。

## 後續步驟 {#next-steps}

現在您已經了解了 HTL 的目標和優勢，因此可以開始使用此語言。請參閱[開始使用 HTML 範本語言](getting-started.md)。
