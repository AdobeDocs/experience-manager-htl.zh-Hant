---
title: HTL 全域物件
description: 了解 HTL 中的可列舉物件、Java 支援的物件和 JavaScript 支援的物件。
exl-id: ca590b92-f1b3-4e44-a04a-a2c10dff256f
source-git-commit: 88edbd2fd66de960460df5928a3b42846d32066b
workflow-type: tm+mt
source-wordcount: '200'
ht-degree: 100%

---


# HTL 全域物件 {#htl-global-objects}

HTL 允許開發人員存取許多實用的物件，且不必指定任何事。這些物件是透過 [Use-API](java-use-api.md) 引進的物件以外的物件。

>[!NOTE]
>
>對於熟悉 AEM 中 JSP 開發的開發人員而言，在包含 `global.jsp` 之後，HTL 允許他們存取 JSP 中提供的所有常用物件。

## 可列舉的物件 {#enumerable-objects}

這些物件可讓您方便地存取常用資訊。 您可以使用點標記法存取這些物件的內容，也可以使用 `data-sly-list` 或 `data-sly-repeat` 逐一查看這些物件。

| 變數名稱 | 說明 | 提供支援 |
|--- |--- |--- |
| `properties` | 目前資源的屬性清單 | [`org.apache.sling.api.resource.ValueMap`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/org/apache/sling/api/resource/ValueMap.html) |
| `pageProperties` | 目前頁面的頁面屬性清單 | [`org.apache.sling.api.resource.ValueMap`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/org/apache/sling/api/resource/ValueMap.html) |
| `inheritedPageProperties` | 目前頁面所繼承的頁面屬性清單 | [`org.apache.sling.api.resource.ValueMap`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/org/apache/sling/api/resource/ValueMap.html) |

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
