---
title: HTL JavaScript使用API
seo-title: HTL JavaScript使用API
description: HTML範本Langgae- HTL- JavaScript Use-API可讓HTL檔案存取以JavaScript編寫的helper程式碼。
seo-description: HTML範本Langgae- HTL- JavaScript Use-API可讓HTL檔案存取以JavaScript編寫的helper程式碼。
uuid: 7ab34b10-30ac-44d6-926b-0224f52e5541
contentOwner: 使用者
products: SG_ PERIENCENCENAGER/HTL
topic-tags: html-template-language
content-type: 引用
discoiquuid: 18871af8-e44 b-4eec-a483-cfc765 gf58 f
mwpw-migration-script-version: 2017-10-12T214658.665-0400
translation-type: tm+mt
source-git-commit: 271c355ae56e16e309853b02b8ef09f2ff971a2e

---


# HTL JavaScript使用API {#htl-javascript-use-api}

HTML範本Langgae(HTL) JavaScript Use-API可讓HTL檔案存取以JavaScript編寫的helper程式碼。這可讓所有複雜的商業邏輯封裝在JavaScript程式碼中，而HTL程式碼只能與直接標記製作搭配使用。

## 簡單範例 {#a-simple-example}

我們定義一個元件， `info`位於

**`/apps/my-example/components/info`**

它包含兩個檔案：

* **`info.js`**：定義使用類別的JavaScript檔案。
* `info.html`：定義元件 `info`的HTL檔案。此程式碼將使用 `info.js` API的功能。

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

我們也會建立使用 **`info`** 元件的內容節點，

**`/content/my-example`**以及屬性：

* `sling:resourceType = "my-example/component/info"`
* `title = "My Example"`
* `description = "This is some example content."`

以下是產生的儲存庫結構：

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

考慮下列元件範本：

```xml
<section class="component-name" data-sly-use.component="component.js">
    <h1>${component.title}</h1>
    <p>${component.description}</p>
</section>
```

對應的邏輯可使用位於範本旁邊的檔案，使用下列 ***伺服器端***`component.js` JavaScript編寫：

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

這會嘗試從 `title` 不同來源擷取，並將描述裁切到50個字元。

## 相依關係 {#dependencies}

假設我們的公用程式類別已經配備了智慧功能，就像導覽標題的預設邏輯，或將字串切切到特定長度：

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

## 擴充功能 {#extending}

相依性模式也可以用來延伸另一個元件的邏輯(通常是目前元件的名稱 `sling:resourceSuperType` )。

假設父元件已提供， `title`而且我們也想要新增 **`description`** ：

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

**`data-sly-template`** 對於可獨立於元件的陳述式，將參數傳遞至關聯的Use-API可能很有用。

因此，我們的元件可以呼叫位於不同檔案中的範本：

```xml
<section class="component-name" data-sly-use.tmpl="template.html" data-sly-call="${tmpl.templateName @ page=currentPage}"></section>
```

Then this is the template in `template.html`：

```xml
<template data-sly-template.templateName="${@ page}" data-sly-use.tmpl="${'template.js' @ page=page, descriptionLength=50}">
    <h1>${tmpl.title}</h1>
    <p>${tmpl.description}</p>
</template>
```

對應的邏輯可使用下列 ***位於*** 範本檔案旁邊 `template.js` 的伺服器端JavaScript來編寫：

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

傳遞的參數會設定在 `this` 關鍵字上。
