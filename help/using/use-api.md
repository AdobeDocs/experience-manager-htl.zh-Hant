---
title: HTL Use-API
description: 有兩種 API 適用於 HTL - Java Use-API 和 Javascript Use-API
exl-id: 8f95707e-952c-4efe-a44e-9a1ad501605e
source-git-commit: 8e70ee4921a7ea071ab7e06947824c371f4013d8
workflow-type: ht
source-wordcount: '183'
ht-degree: 100%

---

# HTL Use-API {#htl-use-api}

HTL 不允許商業邏輯與標記混合，鼓勵將關注點分離。 可透過 Use-API 實作商業邏輯。

下表提供每個 API 的優缺點概觀。

|  | **Java Use-API** | **JavaScript Use-API** |
|--- |--- |--- |
| **優點** | <ul><li>更快</li><li>可使用偵錯工具進行檢查</li><li>可輕鬆進行單元測試</li></ul> | <ul><li>可由前端開發人員修改</li><li>位在元件內，讓元件的檢視邏輯靠近其對應的範本</li></ul> |
| **缺點** | <ul><li>無法由前端開發人員修改</li></ul> | <ul><li>較慢</li><li>還沒有偵錯工具</li><li>進行單元測試較困難</li></ul> |

如果是頁面元件，建議使用混合模型，讓所有模型邏輯都位在 Java 中，提供與檢視內 (亦即元件內) 發生的所有事情都無關的清晰 API。 AEM 隨附優質預設模型 (像是 Page 或 Resource API)，應該能夠涵蓋大多數的使用情況。

元件特有的所有檢視邏輯都應該當作 JavaScript 置於該元件中，因為它屬於該元件。
