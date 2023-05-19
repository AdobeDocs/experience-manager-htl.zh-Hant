---
title: AEM 擴充功能
description: AEM 提供 AEM 之 HTL 規格擴充功能，方便作為開發人員的您使用。
exl-id: d78cb84d-f958-45e2-9c6c-df86a68277d5
source-git-commit: 151b40ee204f1afa7a772afd64cbac4790c02cc2
workflow-type: tm+mt
source-wordcount: '308'
ht-degree: 100%

---

# AEM 擴充功能 {#aem-extensions}

與 HTL 規格的 [Apache Sling 擴充功能](https://sling.apache.org/documentation/bundles/scripting/scripting-htl.html#extensions-of-the-htl-specification-1)相似，AEM 提供其他的運算式選項，使得直接在 HTL 指令碼內使用 AEM 概念運作更容易。

## i18n {#i18n}

與 Apache Sling 相同的[三個其他選項](https://sling.apache.org/documentation/bundles/scripting/scripting-htl.html#i18n)可以搭配 `i18n` 一起使用：

* `locale`
* `hint`
* `basename`

不過，在 AEM 中，要實施 HTL 的[國際化支援](https://experienceleague.adobe.com/docs/experience-manager-65/developing/components/internationalization/i18n-dev.html)，需要 `com.day.cq.i18n` 套件的 API 提供協助。

## data-sly-include {#data-sly-include}

在 AEM，`data-sly-include` 可以選擇一個其他`wcmmode`選項，用來控制所包含指令碼的 [WCM 模式](https://developer.adobe.com/experience-manager/reference-materials/cloud-service/javadoc/com/day/cq/wcm/api/WCMMode.html)。 允許的值包括可用列舉常數的名稱。

## data-sly-resource {#data-sly-resource}

除了路徑和 `Resources` 以外，`data-sly-resource` 區塊元素也可以搭配 [`Maps`](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Map.html) 或 [`Records` 運作。](https://github.com/apache/sling-org-apache-sling-scripting-sightly-runtime/blob/master/src/main/java/org/apache/sling/scripting/sightly/Record.java) 使用兩種方法都必須提供 `resourceName` 字串屬性。 我們使用它的值來建立一個 [Synthetic Resource](https://www.javadoc.io/doc/org.apache.sling/org.apache.sling.api/latest/org/apache/sling/api/resource/SyntheticResource.html) 並把它包含到演算格式文法中。 其餘來自 `Record` 或 `Map` 的屬性若已傳到 `data-sly-resource`，則會當作正常的 `Resource` 屬性使用。 如果地圖上缺少 `sling:resourceType` 屬性，我們會假設資源類型是 `resourceType` [運算式選項](https://github.com/adobe/htl-spec/blob/1.4/SPECIFICATION.md#229-resource)的值，或是驅動演算的目前資源的資源類型。

假定下列指令碼範圍內可用的地圖/記錄屬性是 `map`：

```javascript
{
    resourceName: "myText",
    "sling:resourceType": "core/wcm/components/text/v2/text",
    "text": "Hello World!"
}
```

假定使用下列標記：

```html
<div class="outer" data-sly-resource="${map}"></div>
```

預期會顯示下列輸出：

```html
<div class="outer">
    <div class="myText">
        <div data-cmp-data-layer="{&quot;text-e58d65c472&quot;:{&quot;@type&quot;:&quot;core/wcm/components/text/v2/text&quot;,&quot;xdm:text&quot;:&quot;<p>Hello world!</p>&quot;}}" id="text-e58d65c472" class="cmp-text">
            <p>Hello world!</p>
        </div>
  </div>
</div>
```
