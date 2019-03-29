---
title: HTL Use-API
seo-title: Adobe HTL Use-API
description: 有兩種 API 適用於 HTL - Java Use-API 和 Javascript Use-API
seo-description: 有兩種 API 適用於 Adobe HTL - Java Use-API 和 Javascript Use-API
uuid: ab44aa5c-ce7 e-40b9-97fb-e86 c6 a28405 c
contentOwner: 使用者
products: SG_ PERIENCENCENAGER/HTL
topic-tags: html-template-language
content-type: 引用
discoiquuid: 89004426-eb59-4b63-913f-51bb98626773
mwpw-migration-script-version: 2017-10-12T214658.665-0400
translation-type: tm+mt
source-git-commit: 796c55d3d85e6b5a3efaa5c04a25be1b0b4e54dd

---


# HTL Use-API {#htl-use-api}

下表概述每個API的專業人士和並行作業。

|  | **Java Use-API** | **JavaScript使用API** |
|--- |--- |--- |
| **Pros** | <ul><li>faster</li><li>可使用除錯程式進行檢查</li><li>輕鬆單元測試</li></ul> | <ul><li>可由前端開發人員修改</li><li>位於元件內，將元件的檢視邏輯保持在對應的範本中</li></ul> |
| **Cons** | <ul><li>無法由前端開發人員修改</li></ul> | <ul><li>slow</li><li>無除錯程式(但)</li><li>更難以單元測試</li></ul> |


對於頁面元件，建議使用混合模型，其中所有模型邏輯都位於Java中，提供清楚的API，對於檢視中發生的任何項目都不一致(亦即元件內)。AEM隨附絕佳的預設模型，例如「頁面」或「資源API」，這些模型應該能夠涵蓋大多數案例。

所有特定元件的檢視邏輯都應放在該元件中，因為它屬於JavaScript，因為它屬於該元件。
