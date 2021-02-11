---
title: HTL JavaScript Use-API
description: HTML範本語言- HTL - JavaScript Use-API可讓HTL檔案存取以JavaScript編寫的輔助程式碼。
translation-type: tm+mt
source-git-commit: f7e46aaac2a4b51d7fa131ef46692ba6be58d878
workflow-type: tm+mt
source-wordcount: '324'
ht-degree: 2%

---


# HTL JavaScript Use-API {#htl-javascript-use-api}

HTML範本語言(HTL)JavaScript Use-API可讓HTL檔案存取以JavaScript編寫的輔助程式碼。 這可讓所有複雜的商業邏輯封裝在JavaScript程式碼中，而HTL程式碼則僅處理直接標籤製作。

使用下列慣例。

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

## 簡單範例{#a-simple-example}

我們定義一個元件`info`，位於

`/apps/my-example/components/info`

它包含兩個檔案：

* **`info.js`**:定義use-class的JavaScript檔案。
* **`info.html`**:定義元件的HTL檔案 `info`。此程式碼將透過use-API使用`info.js`的功能。

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

我們也會建立使用`info`元件的內容節點，位於

`/content/my-example`, with屬性：

* `sling:resourceType = "my-example/component/info"`
* `title = "My Example"`
* `description = "This is some example content."`

以下是生成的儲存庫結構：

### 儲存庫結構{#repository-structure}

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

請考慮下列元件範本：

```xml
<section class="component-name" data-sly-use.component="component.js">
    <h1>${component.title}</h1>
    <p>${component.description}</p>
</section>
```

可使用下列位於範本旁的`component.js`檔案中的伺服器端JavaScript來編寫對應的邏輯：

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

這會嘗試從不同的來源取用`title`，並將說明裁切為50個字元。

## 相依關係 {#dependencies}

假設我們有一個公用程式類別，它已配備智慧功能，例如導覽標題的預設邏輯，或將字串精心剪下至特定長度：

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

## 擴展{#extending}

依賴性模式也可用來擴充其他元件的邏輯（通常是目前元件的`sling:resourceSuperType`）。

假設父元件已提供`title`，我們也要添加`description`:

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

## 將參數傳遞至範本{#passing-parameters-to-a-template}

在`data-sly-template`陳述式中，如果陳述式可獨立於元件，則將參數傳遞至相關的Use-API會很有用。

因此，在我們的元件中，讓我們呼叫位於不同檔案中的範本：

```xml
<section class="component-name" data-sly-use.tmpl="template.html" data-sly-call="${tmpl.templateName @ page=currentPage}"></section>
```

這就是位於`template.html`的範本：

```xml
<template data-sly-template.templateName="${@ page}" data-sly-use.tmpl="${'template.js' @ page=page, descriptionLength=50}">
    <h1>${tmpl.title}</h1>
    <p>${tmpl.description}</p>
</template>
```

對應的邏輯可以使用下列伺服器端JavaScript來寫入，位於範本檔案旁的`template.js`檔案中：

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

傳遞的參數會設定在`this`關鍵字上。
