---
title: 擴AEM展
description: AEM為了方便您作為開AEM發人員，提供HTL規範的擴展。
source-git-commit: 6d97bc5d0ab89dffaf56a54c73c94d069bb31ca6
workflow-type: tm+mt
source-wordcount: '308'
ht-degree: 0%

---


# 擴AEM展 {#aem-extensions}

與 [HTL規範的Apache Sling擴展，](https://sling.apache.org/documentation/bundles/scripting/scripting-htl.html#extensions-of-the-htl-specification-1) 提AEM供了一些附加表達式選項，AEM使直接在HTL指令碼中處理概念更簡單。

## i18n {#i18n}

相同 [三個附加選項](https://sling.apache.org/documentation/bundles/scripting/scripting-htl.html#i18n) 與Apache Sling一樣， `i18n`:

* `locale`
* `hint`
* `basename`

然而AEM, [國際化支援](https://experienceleague.adobe.com/docs/experience-manager-65/developing/components/internationalization/i18n-dev.html) HTL是在API的幫助下從 `com.day.cq.i18n` 檔案。

## 資料漏包 {#data-sly-include}

在AEM, `data-sly-include` 可以另外 `wcmmode` 選項 [WCM模式](https://developer.adobe.com/experience-manager/reference-materials/cloud-service/javadoc/com/day/cq/wcm/api/WCMMode.html) 的下界。 允許的值是可用枚舉常數的名稱。

## 資料機密資源 {#data-sly-resource}

除了路徑和 `Resources`，也請參見Wiki頁。 `data-sly-resource` 塊元素也可以 [`Maps`](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Map.html) 或 [`Records`。](https://github.com/apache/sling-org-apache-sling-scripting-sightly-runtime/blob/master/src/main/java/org/apache/sling/scripting/sightly/Record.java) 在兩種方法下， `resourceName` 必須提供字串屬性。 其值用於建立 [合成資源](https://www.javadoc.io/doc/org.apache.sling/org.apache.sling.api/latest/org/apache/sling/api/resource/SyntheticResource.html) 將包含在呈現上下文中。 其餘屬性 `Record` 或 `Map` 被傳到 `data-sly-resource` 將被用作正常 `Resource` 屬性。 如果 `sling:resourceType` 此映射中缺少屬性，將假定資源類型為 `resourceType` [表達式選項](https://github.com/adobe/htl-spec/blob/1.4/SPECIFICATION.md#229-resource) 或驅動呈現的當前資源的資源類型。

在指令碼作用域中提供以下映射/記錄屬性，如 `map`:

```javascript
{
    resourceName: "myText",
    "sling:resourceType": "core/wcm/components/text/v2/text",
    "text": "Hello World!"
}
```

給出以下標籤：

```html
<div class="outer" data-sly-resource="${map}"></div>
```

需要以下輸出：

```html
<div class="outer">
    <div class="myText">
        <div data-cmp-data-layer="{&quot;text-e58d65c472&quot;:{&quot;@type&quot;:&quot;core/wcm/components/text/v2/text&quot;,&quot;xdm:text&quot;:&quot;<p>Hello world!</p>&quot;}}" id="text-e58d65c472" class="cmp-text">
            <p>Hello world!</p>
        </div>
  </div>
</div>
```
