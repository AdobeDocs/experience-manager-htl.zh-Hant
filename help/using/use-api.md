---
title: HTL Use-API
description: 有兩種 API 適用於 HTL - Java Use-API 和 Javascript Use-API
translation-type: tm+mt
source-git-commit: d7efae3d1b4d1bc22c63c21f544a99bf0ae4b3c9
workflow-type: tm+mt
source-wordcount: '183'
ht-degree: 8%

---


# HTL Use-API {#htl-use-api}

HTL不允許商業邏輯與標籤混為一談，以鼓勵分離顧慮。 商業邏輯可透過Use-API實作。

下表概述了每個API的優點和缺點。

|  | **Java Use-API** | **JavaScript Use-API** |
|--- |--- |--- |
| **優勢** | <ul><li>更快速</li><li>可使用除錯程式進行檢查</li><li>易於單元測試</li></ul> | <ul><li>前端開發人員可修改</li><li>位於元件內，讓元件的檢視邏輯與其對應的範本保持接近</li></ul> |
| **缺點** | <ul><li>前端開發人員無法修改</li></ul> | <ul><li>較慢</li><li>尚無除錯程式（尚未）</li><li>單位測試難度較大</li></ul> |

對於頁面元件，建議使用混合模型，其中所有模型邏輯都位於Java中，以提供不受檢視（即元件內）中任何情況影響的清晰API。 AEM隨附絕佳的預設模型，例如應能涵蓋大部分案例的「頁面」或「資源API」。

元件專屬的所有檢視邏輯都應以JavaScript的形式放在該元件中，因為它屬於該元件。
