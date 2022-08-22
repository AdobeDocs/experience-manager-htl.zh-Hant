---
title: HTL 概覽
description: 瞭解如AEM何支援HTML模板語言(HTL)，以提供高效的企業級Web框架，從而提高安全性，並允許沒有Java知識的HTML開發人員更好地參與AEM項目。
exl-id: 5d06ff25-d681-4b95-8375-c28a8364eb7e
source-git-commit: 5ab1275c984135fe946f36905bbc979cf19edd80
workflow-type: tm+mt
source-wordcount: '711'
ht-degree: 69%

---


# 總覽 {#overview}

Adobe Experience Manager (AEM) 支援的 HTML 範本語言 (HTL) 旨在提供高生產力的企業級網頁架構，除了提升安全性，也讓不具備 Java 知識的 HTML 開發人員更能參與 AEM 專案。

[6.AEM0版，](history.md) HTML模板語言是中HTML的首選和推薦的伺服器端模板系AEM統。 對需要建立強大企業網站的網頁開發人員而言，HTML 範本語言有助於提升安全性和開發效率。

## 提升安全性 {#increased-security}

與大多數其他模板系統相比，HTML模板語言提高了在實現中使用該語言的站點的安全性，因為HTL能夠自動將適當的上下文感知轉義應用於輸出到演示層的所有變數。 HTL 之所以能做到這點，是因為其了解 HTML 語法，並運用相關知識，根據運算式在標記中的位置調整其必要逸出。例如，這將導致將表達式放在 `href` 或 `src` 要轉義的屬性與放置在其他屬性或其他地方的表達式不同。

雖然使用 JSP 等範本語言可以達到相同結果，但開發人員必須手動操作，才能確保適當的逸出套用至每個變數。由於套用的逸出只要一有遺漏或錯誤，就可能足以造成跨網站指令碼 (XSS) 出現弱點，因此我們決定使用 HTL 將此工作自動化。如有必要，開發人員仍可在運算式上指定其他逸出，但使用 HTL 的話，預設行為更可能對應到希望的行為，減少發生錯誤的機率。

## 簡化開發工作 {#simplified-development}

HTML 範本語言簡單易學，其功能刻意有所限制，以確保其簡單明瞭。此外，這種範本語言也有強大機制足以建立標記架構及叫用邏輯，同時一律強制分割標記和邏輯面向。HTL 本身屬於標準 HTML5，因其使用運算式和資料屬性，以符合需求的動態行為標註標記，也就是說，這不會破壞標記的有效性，並使標記持續可供讀取。請注意，評估運算式和資料屬性的工作完全在伺服器端進行，用戶端看不見，因此用戶端可以不受干擾地使用想要的 JavaScript 框架。

在這些功能的輔助下，HTML 開發人員可在不具 Java 知識，且幾乎不懂產品專門知識的情況下編輯 HTL 範本，進而協助他們加入開發團隊，並簡化與全端 Java 開發人員的協同合作。換個角度來說，這能讓 Java 開發人員專注於後端程式碼，不必擔心 HTML。

## 降低成本 {#reduced-costs}

提高安全性、簡化開發和改進團隊協作，AEM從而減少工作量、加快產品上市時間(TTM)和降低總擁有成本(TCO)。

具體而言，使用 HTML 範本語言重新實作 Adobe.com 網站，可發現成本和專案所需時間減少約 25%。

![有效地增加和降低成本](assets/chlimage_1.png)

上圖說明 HTL 提升效率的下列成效:

* **HTML/CSS/JS:** 由於 HTML 開發人員能直接編輯 HTL 範本，前端設計不必再從 AEM 專案分別實作，而是能在實際 AEM 元件上直接實作。這能減少痛苦的迭代作業，裨益全端 Java 開發人員。
* **JSP/HTL:** HTL 本身不需搭配任何 Java 知識，且容易撰寫，因此所有具備 HTML 專業的開發人員都能編輯這類範本。
* **Java:** HTL 提供清楚且易於使用的 Use-API，讓具商業邏輯的介面更清晰明瞭，同時有助於整體 Java 開發。

## 視頻簡介 {#video}

以下視頻來自 [AEM寶石會議，](https://experienceleague.adobe.com/docs/experience-manager-gems-events/gems/gems2014/aem-introduction-to-htl.html) 概述了HTL的目的和實施示例。

>[!VIDEO](https://video.tv.adobe.com/v/19504/?quality=9)

請注意，視頻是指HTL [叫艾特利。](history.md)

## 後續步驟 {#next-steps}

現在您瞭解了HTL的目標和優勢，通過查看文檔開始使用語言 [HTML模板語言入門。](getting-started.md)
