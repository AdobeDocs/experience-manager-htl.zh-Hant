---
title: HTL Use-API
seo-title: Adobe HTL Use-API
description: 有兩種 API 適用於 HTL - Java Use-API 和 Javascript Use-API
seo-description: 有兩種 API 適用於 Adobe HTL - Java Use-API 和 Javascript Use-API
uuid: ab44aa5c-ce7e-40b9-97fb-e86c6a28405c
contentOwner: 使用者
products: SG_EXPERIENCEMANAGER/HTL
topic-tags: html-template-language
content-type: 引用
discoiquuid: 89004426-eb59-4b63-913f-51bf98662773
mwpw-migration-script-version: 2017-10-12T21 46 58.665-0400
translation-type: tm+mt
source-git-commit: 5cbaf9c747acf748d12559c2c8e3aba4600cf9a4

---


# HTL使用API {#htl-use-api}

下表概述每個API的優點和缺點。

|  | **Java Use-API** | **JavaScript Use-API** |
|--- |--- |--- |
| **專業人員** | <ul><li>更快速</li><li>可使用除錯程式進行檢查</li><li>易於單元測試</li></ul> | <ul><li>可由前端開發人員修改</li><li>位於元件中，使元件的檢視邏輯與其對應的範本保持接近</li></ul> |
| **缺點** | <ul><li>無法由前端開發人員修改</li></ul> | <ul><li>較慢</li><li>no debugger (yet)</li><li>harder to unit-test</li></ul> |


For page components, it is recommended to use a mixed model, where all model logic is located in Java, providing clear APIs that are agnostic to anything that happens in the view (i.e. within the components). AEM comes with great default models like the Page or the Resource API that should be able to cover most cases.

All view logic that is specific to a component should be placed within that component as JavaScript, because it belongs to that component.
