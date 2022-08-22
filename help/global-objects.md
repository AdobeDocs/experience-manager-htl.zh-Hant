---
title: HTL 全域物件
description: 瞭解HTL中可枚舉的對象、Java支援的對象和JavaScript支援的對象。
exl-id: ca590b92-f1b3-4e44-a04a-a2c10dff256f
source-git-commit: 5ab1275c984135fe946f36905bbc979cf19edd80
workflow-type: tm+mt
source-wordcount: '200'
ht-degree: 32%

---


# HTL 全域物件 {#htl-global-objects}

無需指定任何內容，HTL便可訪問對開發人員有用的許多對象。 這些對象除了通過 [使用 — API。](java-use-api.md)

>[!NOTE]
>
>對於熟悉JSP開發的開發人員AEM,HTL提供了對JSP中通常可用的所有對象的訪問 `global.jsp`。

## 可列舉的物件 {#enumerable-objects}

這些物件可讓您方便地存取常用資訊。 它們的內容可以用點標籤法訪問，並且可以使用 `data-sly-list` 或 `data-sly-repeat`。

| 變數名稱 | 說明 | 支援者 |
|--- |--- |--- |
| `properties` | 當前資源的屬性清單 | [`org.apache.sling.api.resource.ValueMap`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/org/apache/sling/api/resource/ValueMap.html) |
| `pageProperties` | 當前頁的頁屬性清單 | [`org.apache.sling.api.resource.ValueMap`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/org/apache/sling/api/resource/ValueMap.html) |
| `inheritedPageProperties` | 當前頁繼承的頁屬性清單 | [`org.apache.sling.api.resource.ValueMap`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/org/apache/sling/api/resource/ValueMap.html) |

## Java 支援的物件 {#java-backed-objects}

以下每個物件都受到相對應的 Java 物件所支援。

| 變數名稱 | 說明 |
|---|---|
| `component` | `com.day.cq.wcm.api.components.Component` |
| `componentContext` | `com.day.cq.wcm.api.components.ComponentContext` |
| `currentContentPolicy` | `com.day.cq.wcm.api.policies.ContentPolicy` |
| `currentContentPolicyProperties` | `com.day.cq.wcm.api.policies.ContentPolicy` |
| `currentDesign` | `com.day.cq.wcm.api.designer.Design` |
| `currentNode` | `javax.jcr.Node` |
| `currentPage` | `com.day.cq.wcm.api.Page` |
| `currentSession` | `javax.servlet.http.HttpSession` |
| `currentStyle` | `com.day.cq.wcm.api.designer.Style` |
| `designer` | `com.day.cq.wcm.api.designer.Designer` |
| `editContext` | `com.day.cq.wcm.api.components.EditContext` |
| `log` | `org.slf4j.Logger` |
| `out` | `java.io.PrintWriter` |
| `pageManager` | `com.day.cq.wcm.api.PageManager` |
| `reader` | `java.io.BufferedReader` |
| `request` | `org.apache.sling.api.SlingHttpServletRequest` |
| `resolver` | `org.apache.sling.api.resource.ResourceResolver` |
| `resource` | `org.apache.sling.api.resource.Resource` |
| `resourceDesign` | `com.day.cq.wcm.api.designer.Design` |
| `resourcePage` | `com.day.cq.wcm.api.Page` |
| `response` | `org.apache.sling.api.SlingHttpServletResponse` |
| `sling` | `org.apache.sling.api.scripting.SlingScriptHelper` |
| `slyWcmHelper` | `com.adobe.cq.sightly.WCMScriptHelper` |
| `wcmmode` | `com.adobe.cq.sightly.SightlyWCMMode` |
| `xssAPI` | `com.adobe.granite.xss.XSSAPI` |

## JavaScript 支援的物件 {#javascript-backed-objects}

使用 JavaScript 支援 HTL 邏輯是可行的。 不過，首選或推薦的方法是使用 [Sling 模型。](https://sling.apache.org/documentation/bundles/models.html)
