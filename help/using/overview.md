---
title: HTL 總覽
description: 了解 AEM 如何透過支援 HTML 範本語言 (HTL) 來提供高生產力的企業級網頁架構，不但安全性更高，也能讓不具備 Java 知識的 HTML 開發人員能夠參與 AEM 專案。
exl-id: 5d06ff25-d681-4b95-8375-c28a8364eb7e
source-git-commit: 88edbd2fd66de960460df5928a3b42846d32066b
workflow-type: tm+mt
source-wordcount: '711'
ht-degree: 100%

---


# 總覽 {#overview}

Adobe Experience Manager (AEM) 支援的 HTML 範本語言 (HTL) 旨在提供高生產力的企業級網頁架構，除了提升安全性，也讓不具備 Java 知識的 HTML 開發人員更能參與 AEM 專案。

[在 AEM 6.0 引進的](history.md) HTML 範本語言是 AEM 中首選且推薦使用的 HTML 伺服器端範本系統。對需要建立強大企業網站的網頁開發人員而言，HTML 範本語言有助於提升安全性和開發效率。

## 提升安全性 {#increased-security}

網站在實施時使用 HTML 範本語言，可提高網站的安全性，成效比大多數其他範本系統更好，因為 HTL 能夠針對輸出到表現層的變數，自動套用正確的格式文法感知跳脫處理。HTL 之所以能做到這點，是因為其了解 HTML 語法，並運用相關知識，根據運算式在標記中的位置調整其所需的跳脫處理。例如，這樣會造成放在 `href` 或 `src` 屬性中的運算式之跳脫處理方式，與放在其他屬性或其他地方的運算式不同。

雖然使用 JSP 等範本語言可以達到相同結果，但開發人員必須手動操作，才能確保適當的逸出套用至每個變數。由於套用的逸出只要一有遺漏或錯誤，就可能足以造成跨網站指令碼 (XSS) 出現弱點，因此我們決定使用 HTL 將此工作自動化。如有必要，開發人員仍可在運算式上指定其他逸出，但使用 HTL 的話，預設行為更可能對應到希望的行為，減少發生錯誤的機率。

## 簡化開發工作 {#simplified-development}

HTML 範本語言簡單易學，其功能刻意有所限制，以確保其簡單明瞭。此外，這種範本語言也有強大機制足以建立標記架構及叫用邏輯，同時一律強制分割標記和邏輯面向。HTL 本身屬於標準 HTML5，因其使用運算式和資料屬性，以符合需求的動態行為標註標記，也就是說，這不會破壞標記的有效性，並使標記持續可供讀取。請注意，評估運算式和資料屬性的工作完全在伺服器端進行，用戶端看不見，因此用戶端可以不受干擾地使用想要的 JavaScript 框架。

在這些功能的輔助下，HTML 開發人員可在不具 Java 知識，且幾乎不懂產品專門知識的情況下編輯 HTL 範本，進而協助他們加入開發團隊，並簡化與全端 Java 開發人員的協同合作。換個角度來說，這能讓 Java 開發人員專注於後端程式碼，不必擔心 HTML。

## 降低成本 {#reduced-costs}

提高安全性、簡化開發過程更以及改進團隊共同作業，促使執行 AEM 專案可以事半功倍，上市時間更短，而且降低了總體擁有成本 (TCO)。

具體而言，在使用 HTML 範本語言重新實施 Adobe.com 網站的過程中，我們發現成本和專案所需時間可減少約 25%。

![提高效率並降低成本](assets/chlimage_1.png)

上圖說明 HTL 提升效率的下列成效:

* **HTML/CSS/JS:** 由於 HTML 開發人員能直接編輯 HTL 範本，前端設計不必再從 AEM 專案分別實作，而是能在實際 AEM 元件上直接實作。這能減少痛苦的迭代作業，裨益全端 Java 開發人員。
* **JSP/HTL:** HTL 本身不需搭配任何 Java 知識，且容易撰寫，因此所有具備 HTML 專業的開發人員都能編輯這類範本。
* **Java:** HTL 提供清楚且易於使用的 Use-API，讓具商業邏輯的介面更清晰明瞭，同時有助於整體 Java 開發。

## 影片介紹 {#video}

以下影片取自 [AEM Gems 研討會](https://experienceleague.adobe.com/docs/experience-manager-gems-events/gems/gems2014/aem-introduction-to-htl.html)，概要說明 HTL 的用途以及實施範例。

>[!VIDEO](https://video.tv.adobe.com/v/19504/?quality=9)

請注意，影片中提到 HTL 時，均是使用[其以前的名稱 Sightly。](history.md)

## 後續步驟 {#next-steps}

您現在已經了解 HTL 的目的和優點，開始使用此語言時，請仔細閱讀文件[「開始使用 HTML 範本語言」。](getting-started.md)
