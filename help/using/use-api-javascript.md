---
title: HTL javaScript使用-API
seo-title: HTL javaScript使用-API
description: HTML範本語言- HTL - javaScript Use-API可讓HTL檔案存取以JavaScript編寫的輔助程式碼。
seo-description: HTML範本語言- HTL - javaScript Use-API可讓HTL檔案存取以JavaScript編寫的輔助程式碼。
uuid: 7ab34b10-30ac-44d6-926b-0234f52e5541
contentOwner: 使用者
products: SG_EXPERIENCEMANAGER/HTL
topic-tags: html-template-language
content-type: 引用
discoiquuid: 18871af8-e44b-4eec-a483-fcc765dae58f
mwpw-migration-script-version: 2017-10-12T21 46 58.665-0400
translation-type: tm+mt
source-git-commit: bd1962e25d152be4f1608d0a83d8d5b3e728b4aa

---


# HTL javaScript使用-API {#htl-javascript-use-api}

HTML範本語言(HTL)JavaScript Use-API可讓HTL檔案存取以JavaScript編寫的輔助程式碼。 這可讓所有複雜的商業邏輯封裝在JavaScript程式碼中，而HTL程式碼則僅處理直接標籤製作。

## 簡單範例 {#a-simple-example}

我們定義一個元件， `info`位於

**`/apps/my-example/components/info`**

它包含兩個檔案：

* **`info.js`**:定義use-class的JavaScript檔案。
* `info.html`:定義元件的HTL檔案 `info`。 此程式碼將透過use-API `info.js` 使用的功能。

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

我們也會建立內容節點，此節點使用 **`info`** 元件於

**`/content/my-example`**, with屬性：

* `sling:resourceType = "my-example/component/info"`
* `title = "My Example"`
* `description = "This is some example content."`

以下是生成的儲存庫結構：

### 儲存庫結構 {#repository-structure}

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

可使用下列位於範本旁 ***的檔案中的伺服器端*** JavaScript `component.js` 來編寫對應的邏輯：

```
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

這會嘗試從不 `title` 同來源擷取說明，並裁切為50個字元。

## 相依關係 {#dependencies}

假設我們有一個公用程式類別，它已配備智慧功能，例如導覽標題的預設邏輯，或將字串精心剪下至特定長度：

```
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

## 延伸 {#extending}

依賴性模式也可用於擴展另一個元件(通常是當前元件 `sling:resourceSuperType` 的邏輯)的邏輯。

假設父元件已提供 `title`，我們也想新增 **`description`** :

```
use(['../parent-component/parent-component.js'], function (component) {
    var Constants = {
        DESCRIPTION_PROP: "jcr:description",
        DESCRIPTION_LENGTH: 50
    };
 
    component.description = utils.shortenString(properties.get(Constants.DESCRIPTION_PROP, ""), Constants.DESCRIPTION_LENGTH);
 
    return component;
});
```

## 將參數傳遞至範本 {#passing-parameters-to-a-template}

對於可獨立於 **`data-sly-template`** 元件的陳述式，將參數傳遞至相關的Use-API會很有用。

因此，在我們的元件中，讓我們呼叫位於不同檔案中的範本：

```xml
<section class="component-name" data-sly-use.tmpl="template.html" data-sly-call="${tmpl.templateName @ page=currentPage}"></section>
```

此範本位於 `template.html`:

```xml
<template data-sly-template.templateName="${@ page}" data-sly-use.tmpl="${'template.js' @ page=page, descriptionLength=50}">
    <h1>${tmpl.title}</h1>
    <p>${tmpl.description}</p>
</template>
```

可使用下列伺服器端 ***JavaScript來編寫對應的邏輯*** ，位於範本 `template.js` 檔案旁的檔案中：

```
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

傳遞的參數會設定在關鍵字 `this` 上。
