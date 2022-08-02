---
title: HTL JavaScript Use-API
description: 了解 HTML 範本語言 (HTL) JavaScript Use-API 如何讓 HTL 檔案存取以 JavaScript 撰寫的 helper 程式碼。
exl-id: e98bfbd5-fa64-48c7-bd14-477d4c5e1788
source-git-commit: 7b53eff0652f650ffb8caae0e69aa349b5c548eb
workflow-type: ht
source-wordcount: '326'
ht-degree: 100%

---

# HTL JavaScript Use-API {#htl-javascript-use-api}

HTML 範本語言 (HTL) JavaScript Use-API 讓 HTL 檔案能夠存取以 JavaScript 撰寫的 helper 程式碼。 如此可讓所有複雜的商業邏輯都封裝在 JavaScript 程式碼中，而 HTL 程式碼只需處理直接標記的生產。

以下是使用的慣例。

```javascript
/**
 * In the following example '/libs/dep1.js' and 'dep2.js' are optional
 * dependencies needed for this script's execution. Dependencies can
 * be specified using an absolute path or a relative path to this
 * script's own path.
 *
 * If no dependencies are needed the dependencies array can be omitted.
 */
use(['dep1.js', 'dep2.js'], function (Dep1, Dep2) {
    // implement processing
  
    // define this Use object's behavior
    return {
        propertyName: propertyValue
        functionName: function () {}
    }
});
```

## 簡單範例 {#a-simple-example}

我們定義一個元件 `info`，它位在

`/apps/my-example/components/info`

它包含兩個檔案：

* **`info.js`**：定義 use 類別的 JavaScript 檔案。
* **`info.html`**：定義元件 `info` 的 HTL 檔案。 此程式碼將透過 use-API 使用 `info.js` 的功能。

### /apps/my-example/component/info/info.js {#apps-my-example-component-info-info-js}

```java
"use strict";
use(function () {
    var info = {};
    info.title = resource.properties["title"];
    info.description = resource.properties["description"];
    return info;
});
```

### /apps/my-example/component/info/info.html {#apps-my-example-component-info-info-html}

```xml
<div data-sly-use.info="info.js">
    <h1>${info.title}</h1>
    <p>${info.description}</p>
</div>
```

我們也會建立一個使用 `info` 元件的內容節點，它位在

`/content/my-example`，具有以下屬性：

* `sling:resourceType = "my-example/component/info"`
* `title = "My Example"`
* `description = "This is some example content."`

以下是產生的存放庫結構：

### 存放庫結構 {#repository-structure}

```java
{
  "apps": {
    "my-example": {
      "components": {
        "info": {
          "info.html": {
            ...
          },
          "info.js": {
            ...
          }
        }
      }
    }
 },
 "content": {
    "my-example": {
      "sling:resourceType": "my-example/component/info",
      "title": "My Example",
      "description": "This is some example content."
    }
  }
}
```

請考量下列元件範本：

```xml
<section class="component-name" data-sly-use.component="component.js">
    <h1>${component.title}</h1>
    <p>${component.description}</p>
</section>
```

可使用位於範本旁邊的 `component.js` 檔案中的以下伺服器端 JavaScript 編寫對應的邏輯：

```javascript
use(function () {
    var Constants = {
        DESCRIPTION_PROP: "jcr:description",
        DESCRIPTION_LENGTH: 50
    };

    var title = currentPage.getNavigationTitle() || currentPage.getTitle() || currentPage.getName();
    var description = properties.get(Constants.DESCRIPTION_PROP, "").substr(0, Constants.DESCRIPTION_LENGTH);

    return {
        title: title,
        description: description
    };
});
```

這樣會嘗試從不同來源取得 `title`，並將說明裁切成 50 個字元。

## 相依性 {#dependencies}

假設我們有一個公用程式類別已配備智慧功能，像是導覽標題的預設邏輯或是完美地將字串切割成某個長度：

```javascript
use(['../utils/MyUtils.js'], function (utils) {
    var Constants = {
        DESCRIPTION_PROP: "jcr:description",
        DESCRIPTION_LENGTH: 50
    };

    var title = utils.getNavigationTitle(currentPage);
    var description = properties.get(Constants.DESCRIPTION_PROP, "").substr(0, Constants.DESCRIPTION_LENGTH);

    return {
        title: title,
        description: description
    };
});
```

## 擴充 {#extending}

此相依性模式也可用來擴充另一個元件的邏輯 (該元件通常是目前元件的 `sling:resourceSuperType`)。

假設父系元件已提供 `title`，而我們也想要新增 `description`：

```javascript
use(['../parent-component/parent-component.js'], function (component) {
    var Constants = {
        DESCRIPTION_PROP: "jcr:description",
        DESCRIPTION_LENGTH: 50
    };

    component.description = utils.shortenString(properties.get(Constants.DESCRIPTION_PROP, ""), Constants.DESCRIPTION_LENGTH);

    return component;
});
```

## 傳遞參數給範本 {#passing-parameters-to-a-template}

在使用可獨立於元件之外的 `data-sly-template` 陳述式時，將參數傳遞給相關 Use-API 會很有用。

所以在我們的元件中，讓我們來呼叫位於不同檔案中的範本：

```xml
<section class="component-name" data-sly-use.tmpl="template.html" data-sly-call="${tmpl.templateName @ page=currentPage}"></section>
```

然後，這是位在 `template.html` 中的範本：

```xml
<template data-sly-template.templateName="${@ page}" data-sly-use.tmpl="${'template.js' @ page=page, descriptionLength=50}">
    <h1>${tmpl.title}</h1>
    <p>${tmpl.description}</p>
</template>
```

可使用位於範本檔案旁邊的 `template.js` 檔案中的以下伺服器端 JavaScript 編寫對應的邏輯：

```javascript
use(function () {
    var Constants = {
        DESCRIPTION_PROP: "jcr:description"
    };

    var title = this.page.getNavigationTitle() || this.page.getTitle() || this.page.getName();
    var description = this.page.getProperties().get(Constants.DESCRIPTION_PROP, "").substr(0, this.descriptionLength);

    return {
        title: title,
        description: description
    };
});
```

傳遞的參數是在 `this` 關鍵字上設定。
