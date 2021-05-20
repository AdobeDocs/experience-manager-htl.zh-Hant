---
title: HTL Use-API
description: 有兩種 API 適用於 HTL - Java Use-API 和 Javascript Use-API
exl-id: 8f95707e-952c-4efe-a44e-9a1ad501605e
source-git-commit: 8e70ee4921a7ea071ab7e06947824c371f4013d8
workflow-type: tm+mt
source-wordcount: '183'
ht-degree: 8%

---

# HTL Use-API {#htl-use-api}

HTL不允許商業邏輯與標籤混合，借此鼓勵區分關注點。 商業邏輯可透過Use-API實作。

下表概述各API的優點和缺點。

|  | **Java Use-API** | **JavaScript Use-API** |
|--- |--- |--- |
| **優勢** | <ul><li>更快</li><li>可使用偵錯工具進行檢查</li><li>易於單元測試</li></ul> | <ul><li>可由前端開發人員修改</li><li>位於元件內，使元件的檢視邏輯與其對應的範本保持接近</li></ul> |
| **缺點** | <ul><li>無法由前端開發人員修改</li></ul> | <ul><li>慢</li><li>尚無除錯程式</li><li>更難進行單元測試</li></ul> |

若是頁面元件，建議使用混合模型，其中所有模型邏輯都位於Java中，提供不受檢視中任何情況（即元件內）限制的清晰API。 AEM隨附絕佳的預設模型，例如應能涵蓋大部分案例的頁面或資源API。

特定於元件的所有檢視邏輯都應以JavaScript的形式放在該元件內，因為它屬於該元件。
