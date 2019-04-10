---
title: 綜覽
seo-title: 綜覽
description: AEM支援HTL的目的，在於提供具高生產力的企業級網頁架構，以提高安全性，並讓不具備Java知識的HTML開發人員更能參與AEM專案。
seo-description: HTML範本語言- HTL-由Adobe Experience Manager支援的目的，在於提供具高生產力的企業級網頁架構，以提高安全性，並讓不具備Java知識的HTML開發人員更能參與AEM專案。
uuid: 8f 486325-a1 b-4186-a998-96fc0034 c44 a
contentOwner: 使用者
products: SG_ PERIENCENCENAGER/HTL
topic-tags: 簡介
content-type: 引用
discoiquuid: 8f779e08-94c7-43bc-a6 e5-d81 a9 f818 c5 c
mwpw-migration-script-version: 2017-10-12T214658.665-0400
translation-type: tm+mt
source-git-commit: 271c355ae56e16e309853b02b8ef09f2ff971a2e

---


# 綜覽 {#overview}

Adobe Experience Manager(AEM)支援的HTML範本語言(HTL)目的，在於提供具高生產力的企業級網頁架構，以提高安全性，並讓不具備Java知識的HTML開發人員更能參與AEM專案。

HTML範本語言已隨附於AEM6.0，並將JSP(JavaServer Pages)取代為偏好且建議的伺服器端範本系統。對於需要建立強大企業網站的網頁開發人員，HTML範本語言有助於提高安全性和開發效率。

## 提高安全性 {#increased-security}

與JSP和大部分其他範本系統一樣，HTML範本語言可提高在實施中使用的網站安全性，因為HTL可自動套用適當內容感知逸出至輸出至表現層的所有變數。HTL可讓這種情況成為可能，因為它瞭解HTML語法，並使用該知識根據他們在標記中的位置調整運算式的必要逸出。這將會導致置入的運算式 `href` 或 `src` 屬性偏離放置在其他屬性中或其他位置的運算式。

雖然可使用JSP語言(例如JSP)達成相同的結果，但開發人員必須手動確定適當逸出會套用至每個變數。由於套用逸出的疏忽或錯誤可能足以造成跨網站指令碼(XSS)弱點，所以我們決定使用HTL將此項工作自動化。如果有需要，開發人員仍可指定不同的運算式逸出，但使用HTL時，預設行為更有可能對應至所需行為，以降低發生錯誤的可能性。

## 簡化開發 {#simplified-development}

HTML範本語言易學好用，其功能明確局限於確保簡單而直接。它也具有強大的機制來structuring標記和叫用邏輯，同時始終嚴格分離標記與邏輯之間的顧慮。HTL本身是標準HTML5，它使用運算式和資料屬性來註解標記，並加上所需的動態行為，這表示它不會破壞標記的有效性並保持可讀性。請注意，對運算式和資料屬性的評估完全是從伺服器端完成，而不會出現在用戶端，而任何需要的JavaScript架構都可以使用而不會干擾。

這些功能可讓沒有Java知識且具備少量產品特定知識的HTML開發人員編輯HTL範本，讓他們成為開發團隊的一員，並簡化與完整堆疊Java開發人員的合作。反之亦然，Java開發人員可專注於後端程式碼，而不必擔心HTML。

## 降低成本 {#reduced-costs}

提高安全性、簡化開發和團隊協作、為AEM專案翻譯以降低工作量、縮短上市時間，並降低總擁有成本(TCO)。

相反地，從使用HTML範本語言重新實施「Adobe.com」網站時觀察到的是，專案的成本和期間可能減少25%左右。

![](assets/chlimage_1.png)

以上圖表顯示HTL可能帶來的效率改善：

* **HTML/CSS/JS：** 由於HTML開發人員可直接編輯HTL範本，所以前端設計不需要另外與AEM專案實作，但可直接在實際AEM元件上實作。如此可減少與完整堆疊Java開發人員的煩擾。
* **JSP/HTL：** 由於HTL本身不需要任何Java知識，所以要直接編寫，任何具備HTML專業知識的開發人員都有能力編輯範本。
* **Java：** 由於使用HTL提供的「Use-API」清晰簡單易用，因此釐清了與商業邏輯的介面，同時也有利於Java開發整體。

**下一步閱讀：**

* [HTML範本語言快速入門](getting-started.md)

