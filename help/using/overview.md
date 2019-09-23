---
title: AEM HTL Overview
seo-title: Overview of AEM HTL technical documentation.
description: AEM支援的HTL旨在提供高生產力的企業層級網頁架構，以提高安全性，並讓不具Java知識的HTML開發人員更能參與AEM專案。
seo-description: This document lays out the principles and purpose of HTML Template Language - HTL - supported by Adobe Experience Manager. HTL is a highly productive enterprise-level web framework that increases security, and allows HTML developers without Java knowledge to better participate in AEM projects.
uuid: 8f486325-0a1b-4186-a998-96fc0034c44a
contentOwner: 使用者
products: SG_EXPERIENCEMANAGER/HTL
topic-tags: 簡介
content-type: 引用
discoiquuid: 8f779e08-94c7-43bc-a6e5-d81a9f818c5c
mwpw-migration-script-version: 2017-10-12T21 46 58.665-0400
translation-type: tm+mt
source-git-commit: 1d4565a4cffa6e5d42d6a5242f7ce62203dc7c63

---


# 綜覽 {#overview}

HTML範本語言(HTL)由Adobe Experience Manager(AEM)支援，其目的在於提供高生產力的企業層級網路架構，以提高安全性，並讓不具Java知識的HTML開發人員更能參與AEM專案。

AEM 6.0已推出HTML範本語言，取代JSP(JavaServer Pages)，成為HTML的偏好和建議的伺服器端範本系統。 對於需要建立強穩企業網站的網頁開發人員，HTML範本語言有助於提高安全性和開發效率。

## 提高安全性 {#increased-security}

與JSP和大部分其他範本系統相比，HTML範本語言可提高實作中使用HTML的網站的安全性，因為HTL可自動將適當的上下文感應逸出套用至輸出至表現層的所有變數。 HTL之所以能夠做到這一點，是因為它瞭解HTML語法，並使用這些知識根據運算式在標籤中的位置來調整必要的逸出。 例如，這會導致放置在或屬性中的運算式與放 `href` 入其他屬 `src` 性或其他位置的運算式不同地逸出。

雖然使用JSP等範本語言可以取得相同的結果，但開發人員必須手動確保將適當的逸出套用至每個變數。 由於所套用逸出的單一遺漏或錯誤可能足以造成跨網站指令碼(XSS)弱點，我們決定使用HTL將此工作自動化。 如果需要，開發人員仍可在運算式上指定不同的逸出，但是使用HTL，預設行為更可能對應至所要的行為，降低發生錯誤的可能性。

## Simplified Development {#simplified-development}

HTML範本語言易學，其功能有意限制，以確保其簡單明瞭。 It also has powerful mechanisms for structuring the markup and invoking logic, while always enforcing strict separation of concerns between markup and logic. HTL itself is standard HTML5 as it uses expressions and data attributes to annotate the markup with the desired dynamic behavior, meaning that it doesn't break the validity of the markup and keeps it readable. Note that the evaluation of the expressions and data attributes is done entirely server-side and won't be visible on the client-side, where any desired JavaScript framework can be used without interfering.

這些功能可讓不具備Java知識的HTML開發人員，而且幾乎不具備特定產品知識的HTML開發人員編輯HTL範本，讓他們成為開發團隊的一員，並簡化與完整Java開發人員的協作。 And vice versa this allows Java developers to focus on the back-end code without worrying about HTML.

## 降低成本 {#reduced-costs}

提高安全性、簡化開發並改善團隊協作，讓AEM專案的工作量變得更輕、上市時間(TTM)更快，而且總擁有成本(TCO)也更低。

具體而言，從使用HTML範本語言重新實作Adobe.com網站時觀察到的情況來看，專案的成本和持續時間可減少約25%。

![](assets/chlimage_1.png)

上圖顯示HTL可能改善的效率：

* **** HTML / CSS / JS:由於HTML開發人員可直接編輯HTL範本，所以前端設計不必再與AEM專案分開實施，而可以直接在實際的AEM元件上實施。 This reduces painful iterations with the full-stack Java developers.
* **** JSP / HTL: Since HTL itself doesn't require any Java knowledge and is straight-forward to write, any developer with HTML expertise is empowered to edit the templates.
* **** Java:由於HTL提供的Use-API使用簡單明瞭，所以與商業邏輯的介面也清晰明瞭，這也對Java開發整體有利。

**閱讀下一頁：**

* [HTML範本語言快速入門](getting-started.md)

