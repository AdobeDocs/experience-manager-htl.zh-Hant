---
title: HTL Java Use-API
description: HTML 範本語言 - HTL - Java Use-API 可讓 HTL 檔案存取自訂 Java 類別中的 Helper 方法。
exl-id: 9a9a2bf8-d178-4460-a3ec-cbefcfc09959
source-git-commit: 8e70ee4921a7ea071ab7e06947824c371f4013d8
workflow-type: tm+mt
source-wordcount: '2558'
ht-degree: 1%

---

# HTL Java Use-API {#htl-java-use-api}

HTML範本語言(HTL)Java Use-API可讓HTL檔案透過`data-sly-use`存取自訂Java類別中的Helper方法。 這可將所有複雜的商業邏輯封裝在Java程式碼中，而HTL程式碼僅處理直接標籤生產。

Java Use-API物件可以是簡單的POJO，由特定實作透過POJO的預設建構函式實例化。

Use-API POJO也可以公開名為init的公用方法，並具有下列簽名：

```java
    /**
     * Initializes the Use bean.
     *
     * @param bindings All bindings available to the HTL scripts.
     **/
    public void init(javax.script.Bindings bindings);
```

`bindings`對應可包含物件，這些物件提供目前執行之HTL指令碼的內容，Use-API物件可用於處理該指令碼。

## 簡單範例{#a-simple-example}

我們將從沒有use-class的HTL元件開始。 它包含單個檔案`/apps/my-example/components/info.html`

### `/apps/my-example/component/info/info.html` {#apps-my-example-component-info-info-html}

```xml
<div>
    <h1>${properties.title}</h1>
    <p>${properties.description}</p>
</div>
```

我們也會為此元件新增一些內容，以在`/content/my-example/`呈現：

### `http://<host>:<port>/content/my-example.json` {#http-localhost-content-my-example-json}

```java
{
    "sling:resourceType": "my-example/component/info",
    "title": "My Example",
    "description": "This Is Some Example Content."
}
```

存取此內容時，會執行HTL檔案。 在HTL程式碼中，我們使用內容物件`properties`來存取目前資源的`title`和`description`並顯示它們。 輸出HTML將是：

### `view-source:http://<host>:<port>/content/my-example.html` {#view-source-http-localhost-content-my-example-html}

```xml
<div>
    <h1>My Example</h1>
    <p>This Is Some Example Content.</p>
</div>
```

### 添加Use-Class {#adding-a-use-class}

**info**&#x200B;元件如此，就不需要use-class來執行其（非常簡單）函式。 不過，在某些情況下，您必須執行無法在HTL中完成的工作，因此需要使用類別。 但請記住以下幾點：

>[!NOTE]
>
>只有在無法單獨在HTL中完成某項作業時，才應使用使用類別。

例如，假設您希望`info`元件顯示資源的`title`和`description`屬性，但全部以小寫顯示。 由於HTL沒有小寫字串的方法，因此您需要use-class。 我們可以通過添加Java use-class並按如下方式更改`info.html`來完成此操作：

### `/apps/my-example/component/info/info.html` {#apps-my-example-component-info-info-html-1}

```xml
<div data-sly-use.info="Info">
    <h1>${info.lowerCaseTitle}</h1>
    <p>${info.lowerCaseDescription}</p>
</div>
```

### `/apps/my-example/component/info/Info.java` {#apps-my-example-component-info-info-java}

```java
package apps.my_example.components.info;

import com.adobe.cq.sightly.WCMUsePojo;

public class Info extends WCMUsePojo {
    private String lowerCaseTitle;
    private String lowerCaseDescription;

    @Override
    public void activate() throws Exception {
        lowerCaseTitle = getProperties().get("title", "").toLowerCase();
        lowerCaseDescription = getProperties().get("description", "").toLowerCase();
    }

    public String getLowerCaseTitle() {
        return lowerCaseTitle;
    }

    public String getLowerCaseDescription() {
        return lowerCaseDescription;
    }
}
```

在以下各節中，我們將逐步了解程式碼的不同部分。

### 本地與捆綁Java類{#local-vs-bundle-java-class}

Java use-class可以通過兩種方式安裝：**local**&#x200B;或&#x200B;**bundle**。 此範例使用本機安裝。

在本機安裝中，Java來源檔案會放置在HTL檔案旁，位於相同的存放庫資料夾中。 源將按需自動編譯。 不需要單獨的編譯或打包步驟。

在套件安裝中，必須使用標準AEM套件部署機制在OSGi套件內編譯和部署Java類（請參閱[套件Java類](#bundled-java-class)）。

>[!NOTE]
>
>當use-class特定於相關元件時，建議使用&#x200B;**本地Java use-class**。
>
>當Java程式碼實作從多個HTL元件存取的服務時，建議使用&#x200B;**套件組合Java use-class**。

### Java包是儲存庫路徑{#java-package-is-repository-path}

使用本地安裝時，use-class的包名稱必須與儲存庫資料夾位置的名稱一致，且路徑中的任何連字型大小在包名稱中都以下划線取代。

在這種情況下，`Info.java`位於`/apps/my-example/components/info`，因此包是`apps.my_example.components.info`:

### `/apps/my-example/component/info/Info.java` {#apps-my-example-component-info-info-java-1}

```java
package apps.my_example.components.info;

import com.adobe.cq.sightly.WCMUsePojo;

public class Info extends WCMUsePojo {

   ...

}
```

>[!NOTE]
>
>在AEM開發中，建議在存放庫項目名稱中使用連字型大小。 不過，連字型大小在Java套件名稱中是非法的。 因此，儲存庫路徑中的&#x200B;**所有連字型大小必須轉換為封裝名稱**&#x200B;中的底線。

### 擴展`WCMUsePojo` {#extending-wcmusepojo}

雖然有許多將Java類別與HTL結合的方法（請參閱`WCMUsePojo`的替代方法），但最簡單的方式是延伸`WCMUsePojo`類別：

#### `/apps/my-example/component/info/Info.java` {#apps-my-example-component-info-info-java-2}

```java
package apps.my_example.components.info;

import com.adobe.cq.sightly.WCMUsePojo;

public class Info extends WCMUsePojo

    ...
}
```

### 初始化類{#initializing-the-class}

從`WCMUsePojo`擴展use-class時，通過覆蓋`activate`方法來執行初始化：

### /apps/my-example/component/info/Info.java {#apps-my-example-component-info-info-java-3}

```java
...

public class Info extends WCMUsePojo {
    private String lowerCaseTitle;
    private String lowerCaseDescription;

    @Override
    public void activate() throws Exception {
        lowerCaseTitle = getProperties().get("title", "").toLowerCase();
        lowerCaseDescription = getProperties().get("description", "").toLowerCase();
    }

...

}
```

### 上下文 {#context}

通常， [activate](https://helpx.adobe.com/experience-manager/6-5/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUsePojo.html)方法會用來根據目前的內容（例如目前的要求和資源），預先計算並儲存（在成員變數中）HTL程式碼中需要的值。

`WCMUsePojo`類提供對與HTL檔案中可用的一組上下文對象的訪問（請參閱[全局對象](global-objects.md)）。

在擴展`WCMUsePojo`的類中，可以使用名稱訪問上下文對象

[`<T> T get(String name, Class<T> type)`](https://helpx.adobe.com/experience-manager/6-5/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUsePojo.html)

或者，常用的上下文對象可通過相應的&#x200B;**方便方法**&#x200B;直接訪問：

|  |  |
|---|---|
| [PageManager](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/day/cq/wcm/api/PageManager.html) | [getPageManager()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getPageManager()) |
| [頁面](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/day/cq/wcm/api/Page.html) | [getCurrentPage()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getCurrentPage()) |
| [頁面](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/day/cq/wcm/api/Page.html) | [getResourcePage()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getResourcePage()) |
| [ValueMap](https://helpx.adobe.com/experience-manager/6-5/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/resource/ValueMap.html) | [getPageProperties()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getPageProperties()) |
| [ValueMap](https://helpx.adobe.com/experience-manager/6-5/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/resource/ValueMap.html) | [getProperties()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getProperties()) |
| [設計工具](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/day/cq/wcm/api/designer/Designer.html) | [getDesigner()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getDesigner()) |
| [設計](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/day/cq/wcm/api/designer/Design.html) | [getCurrentDesign()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getCurrentDesign()) |
| [樣式](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/day/cq/wcm/api/designer/Style.html) | [getCurrentStyle()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getCurrentStyle()) |
| [元件](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/day/cq/wcm/api/components/Component.html) | [getComponent()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getComponent()) |
| [ValueMap](https://helpx.adobe.com/experience-manager/6-5/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/resource/ValueMap.html) | [getInheritedProperties()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getInheritedProperties) |
| [資源](https://helpx.adobe.com/experience-manager/6-5/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/resource/Resource.html) | [getResource()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getResource()) |
| [ResourceResolver](https://helpx.adobe.com/experience-manager/6-5/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/resource/ResourceResolver.html) | [getResourceResolver()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getResourceResolver()) |
| [SlingHttpServletRequest](https://helpx.adobe.com/experience-manager/6-5/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/SlingHttpServletRequest.html) | [getRequest()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getRequest()) |
| [SlingHttpServletResponse](https://helpx.adobe.com/experience-manager/6-5/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/SlingHttpServletResponse.html) | [getResponse()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getResponse()) |
| [SlingScriptHelper](https://helpx.adobe.com/experience-manager/6-5/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/scripting/SlingScriptHelper.html) | [getSlingScriptHelper()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getSlingScriptHelper()) |

### Getter方法{#getter-methods}

use-class初始化後，HTL檔案即會執行。 在此階段中，HTL通常會提取使用類別中各種成員變數的狀態，並轉譯為呈現。

若要從HTL檔案內提供對這些值的存取，您必鬚根據下列命名慣例在use-class中定義自訂getter方法：

* `getXyz`格式的方法將在HTL檔案內公開名為`xyz`的物件屬性。

在下列範例中，方法`getTitle`和`getDescription`導致物件屬性`title`和`description`可在HTL檔案的內容中存取：

### `/apps/my-example/component/info/Info.java` {#apps-my-example-component-info-info-java-4}

```java
...

public class Info extends WCMUsePojo {

    ...

    public String getLowerCaseTitle() {
        return lowerCaseTitle;
    }

    public String getLowerCaseDescription() {
        return lowerCaseDescription;
    }
}
```

### data-sly-use屬性{#data-sly-use-attribute}

`data-sly-use`屬性可用來初始化HTL程式碼內的use-class。 在本例中， `data-sly-use`屬性聲明我們要使用類`Info`。 由於我們使用本機安裝（將Java來源檔案放在與HTL檔案相同的資料夾中），因此我們只能使用類別的本機名稱。 如果使用捆綁安裝，則必須指定完全限定的類名。

### `/apps/my-example/component/info/info.html` {#apps-my-example-component-info-info-html-2}

```xml
<div data-sly-use.info="Info">
    <h1>${info.lowerCaseTitle}</h1>
    <p>${info.lowerCaseDescription}</p>
</div>
```

### 本地標識符{#local-identifier}

識別碼`info`（`data-sly-use.info`中的點後面）會用於HTL檔案中以識別類別。 此識別碼在宣告後，在檔案中為全域。 它不限於包含`data-sly-use`陳述式的元素。

### `/apps/my-example/component/info/info.html`{#apps-my-example-component-info-info-html-3}

```xml
<div data-sly-use.info="Info">
    <h1>${info.lowerCaseTitle}</h1>
    <p>${info.lowerCaseDescription}</p>
</div>
```

### 獲取屬性{#getting-properties}

然後，標識符`info`用於訪問通過getter方法`Info.getTitle`和`Info.getDescription`公開的對象屬性`title`和`description`。

### `/apps/my-example/component/info/info.html` {#apps-my-example-component-info-info-html-4}

```xml
<div data-sly-use.info="Info">
    <h1>${info.lowerCaseTitle}</h1>
    <p>${info.lowerCaseDescription}</p>
</div>
```

### 輸出 {#output}

現在，當我們存取`/content/my-example.html`時，它會傳回下列HTML:

### `view-source:http://<host>:<port>/content/my-example.html` {#view-source-http-localhost-content-my-example-html-1}

```xml
<div>
    <h1>my example</h1>
    <p>this is some example content.</p>
</div>
```

## 超越基本{#beyond-the-basics}

在本節中，我們將介紹一些超越上述簡單範例的其他功能：

* 將參數傳遞至use-class。
* 捆綁的Java使用類。
* `WCMUsePojo`的替代方案

### 傳遞參數{#passing-parameters}

初始化時，參數可傳遞至use類。 例如，我們可以執行下列動作：

### `/content/my-example/component/info/info.html` {#content-my-example-component-info-info-html}

```xml
<div data-sly-use.info="${'Info' @ text='Some text'}">
    <h1>${info.lowerCaseTitle}</h1>
    <p>${info.lowerCaseDescription}</p>
    <p>${info.upperCaseText}</p>
</div>
```

我們在此處傳遞名為`text`的參數。 use-class接著將我們擷取的字串加上，並使用`info.upperCaseText`顯示結果。 以下是調整後的使用類別：

### `/apps/my-example/component/info/Info.java` {#apps-my-example-component-info-info-java-5}

```java
package apps.my_example.components.info;

import com.adobe.cq.sightly.WCMUsePojo;

public class Info extends WCMUsePojo {

    ...

    private String reverseText;

    @Override
    public void activate() throws Exception {

        ...

        String text = get("text", String.class);
        reverseText = new StringBuffer(text).reverse().toString();

    }

    public String getReverseText() {
        return reverseText;
    }

    ...
}
```

參數可透過`WCMUsePojo`方法[`<T> T get(String paramName, Class<T> type)`](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html)存取

在本案中，聲明：

`get("text", String.class)`

然後，字串會透過方法反轉並公開：

`getReverseText()`

### 僅從data-sly-template {#only-pass-parameters-from-data-sly-template}傳遞參數

雖然上述範例在技術上正確，但如果相關值可用於HTL程式碼的執行內容中，（或實際上，如上所述，值為靜態值），從HTL傳遞值以初始化use-class實際上並沒什麼意義。

原因在於use-class一律可存取與HTL程式碼相同的執行內容。 這引出了最佳實務匯入點：

>[!NOTE]
>
>只有在`data-sly-template`檔案中使用use-class時，才應將參數傳遞至use-class，而此檔案本身是從另一個HTL檔案呼叫，其中包含需要傳遞的參數。

例如，我們將在現有範例旁建立個別的`data-sly-template`檔案。 我們將調用新檔案`extra.html`。 它包含名為`extra`的`data-sly-template`塊：

### `/apps/my-example/component/info/extra.html` {#apps-my-example-component-info-extra-html}

```xml
<template data-sly-template.extra="${@ text}"
          data-sly-use.extraHelper="${'ExtraHelper' @ text=text}">
  <p>${extraHelper.reversedText}</p>
</template>
```

範本`extra`會採用單一參數`text`。 然後，它以本地名稱`extraHelper`初始化Java use-class `ExtraHelper`，並將模板參數`text`的值作為use-class參數`text`傳遞。

範本的內文會取得屬性`extraHelper.reversedText`（其實在引擎罩下呼叫`ExtraHelper.getReversedText()`）並顯示該值。

我們也會調整現有的`info.html`以使用此新範本：

### `/apps/my-example/component/info/info.html` {#apps-my-example-component-info-info-html-5}

```xml
<div data-sly-use.info="Info"
     data-sly-use.extra="extra.html">

  <h1>${info.lowerCaseTitle}</h1>
  <p>${info.lowerCaseDescription}</p>

  <div data-sly-call="${extra.extra @ text=properties.description}"></div>

</div>
```

檔案`info.html`現在包含兩個`data-sly-use`語句，其中原始語句導入`Info` Java use-class，新語句導入本地名稱`extra`下的模板檔案。

請注意，我們本可以將範本區塊放在`info.html`檔案內，以避免第二個`data-sly-use`，但單獨的範本檔案較常見，且可重複使用。

`Info`類與以前一樣使用，通過其對應的HTL屬性`info.lowerCaseTitle`和`info.lowerCaseDescription`調用其getter方法`getLowerCaseTitle()`和`getLowerCaseDescription()`。

然後，我們對範本`extra`執行`data-sly-call`，並將值`properties.description`傳遞為參數`text`。

Java use-class `Info.java`已更改，以處理新文本參數：

### `/apps/my-example/component/info/ExtraHelper.java` {#apps-my-example-component-info-extrahelper-java}

```java
package apps.my_example.components.info;

import com.adobe.cq.sightly.WCMUsePojo;

public class ExtraHelper extends WCMUsePojo {
    private String reversedText;
    ...

    @Override
    public void activate() throws Exception {
        String text = get("text", String.class);
        reversedText = new StringBuilder(text).reverse().toString();

        ...
    }

    public String getReversedText() {
        return reversedText;
    }
}
```

使用`get("text", String.class)`擷取`text`參數時，值會反轉，並透過getter `getReversedText()`以HTL物件`reversedText`的形式提供使用。

### 捆綁Java類{#bundled-java-class}

若使用套件使用類別，必須使用標準OSGi套件部署機制，在AEM中編譯、封裝及部署類別。 與本機安裝不同， use-class **package declaration**&#x200B;應正常命名：

### `/apps/my-example/component/info/Info.java` {#apps-my-example-component-info-info-java-6}

```java
package org.example.app.components;

import com.adobe.cq.sightly.WCMUsePojo;

public class Info extends WCMUsePojo {
    ...
}
```

和，`data-sly-use`語句必須引用完全限定的類名，而不只是本地類名：

### `/apps/my-example/component/info/info.html` {#apps-my-example-component-info-info-html-6}

```xml
<div data-sly-use.info="org.example.app.components.info.Info">
  <h1>${info.title}</h1>
  <p>${info.description}</p>
</div>
```

### `WCMUsePojo` {#alternatives-to-wcmusepojo}的替代方案

建立Java use-class的最常見方式是擴展`WCMUsePojo`。 不過，還有其他許多選項。 若要了解這些變體，請了解HTL `data-sly-use`陳述式在外罩下的運作方式。

假設您有下列`data-sly-use`陳述式：

**`<div data-sly-use.`** `localName`**`="`**`UseClass`**`">`**

系統會依下列方式處理陳述式：

(1)

* 如果與HTL檔案位於同一目錄中，存在本地檔案`UseClass.java`，請嘗試編譯並載入該類。 如果成功，請前往(2)。
* 否則，將`UseClass`解釋為完全限定的類名，並嘗試從OSGi環境中載入它。 如果成功，請前往(2)。
* 否則，將`UseClass`解譯為HTL或JavaScript檔案的路徑，並載入該檔案。 如果成功轉到(4)。

(2)

* 嘗試將當前`Resource`調整為`UseClass`。 如果成功，請前往(3)。
* 否則，嘗試將當前`Request`調整為`UseClass`。 如果成功，請前往(3)。
* 否則，嘗試使用零參數建構子實例化`UseClass`。 如果成功，請前往(3)。

(3)

* 在HTL內，將新調整或建立的物件系結至名稱`localName`。
* 如果`UseClass`實作[`io.sightly.java.api.Use`](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/io/sightly/java/api/Use.html)，則調用`init`方法，傳遞當前執行上下文（以`javax.scripting.Bindings`對象的形式）。

(4)

* 如果`UseClass`是包含`data-sly-template`之HTL檔案的路徑，請準備範本。
* 否則，如果`UseClass`是JavaScript use-class的路徑，請準備use-class（請參閱[JavaScript Use-API](use-api-javascript.md)）。

上述說明的幾點要點：

* 任何可從`Resource`調整、從`Request`調整的類或具有零參數建構子的類都可以是使用類。 類不必擴展`WCMUsePojo`甚至實現`Use`。
* 不過，如果use-class *does*&#x200B;實作`Use`，則其`init`方法將自動與目前內容呼叫，讓您將初始化程式碼放置在依賴該內容的位置。
* 延伸`WCMUsePojo`的使用類別只是實作`Use`的特殊案例。 它提供了方便的上下文方法，並且其`activate`方法從`Use.init`中自動調用。

### 使用{#directly-implement-interface-use}直接實作介面

建立use-class的最常見方式是擴展`WCMUsePojo`，但也可以直接實施[`io.sightly.java.api.Use`](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/io/sightly/java/api/Use.html)介面本身。

`Use`介面僅定義一個方法：

[`public void init(javax.script.Bindings bindings)`](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/io/sightly/java/api/Use#init(javax.script.Bindings))

在初始化類時，將調用`init`方法，該類具有包含所有上下文對象和傳遞到use-class的任何參數的`Bindings`對象。

必須使用[`javax.script.Bindings`](http://docs.oracle.com/javase/7/docs/api/javax/script/Bindings.html)物件明確實作所有其他功能（例如`WCMUsePojo.getProperties()`的等同功能）。 例如：

### `Info.java` {#info-java}

```java
import io.sightly.java.api.Use;

public class MyComponent implements Use {
   ...
    @Override
    public void init(Bindings bindings) {

        // All standard objects/binding are available
        Resource resource = (Resource)bindings.get("resource");
        ValueMap properties = (ValueMap)bindings.get("properties");
        ...

        // Parameters passed to the use-class are also available
        String param1 = (String) bindings.get("param1");
    }
    ...
}
```

要自行實施`Use`介面而不是擴展`WCMUsePojo`，主要情況是希望將已存在類的子類用作use-class。

### 可從資源{#adaptable-from-resource}調整

另一個選項是使用可從`org.apache.sling.api.resource.Resource`調整的幫助程式類。

假設您需要撰寫HTL指令碼，以顯示DAM資產的中型。 在此案例中，您知道呼叫HTL指令碼時，該指令碼會位於`Resource`的內容中，該內容以nodetype `dam:Asset`包住JCR `Node`。

您知道`dam:Asset`節點的結構如下：

### 儲存庫結構{#repository-structure}

```java
{
  "content": {
    "dam": {
      "geometrixx": {
        "portraits": {
          "jane_doe.jpg": {
            ...
          "jcr:content": {
            ...
            "metadata": {
              ...
            },
            "renditions": {
              ...
              "original": {
                ...
                "jcr:content": {
                  "jcr:primaryType": "nt:resource",
                  "jcr:lastModifiedBy": "admin",
                  "jcr:mimeType": "image/jpeg",
                  "jcr:lastModified": "Fri Jun 13 2014 15:27:39 GMT+0200",
                  "jcr:data": ...,
                  "jcr:uuid": "22e3c598-4fa8-4c5d-8b47-8aecfb5de399"
                }
              },
              "cq5dam.thumbnail.319.319.png": {
                  ...
              },
              "cq5dam.thumbnail.48.48.png": {
                  ...
              },
              "cq5dam.thumbnail.140.100.png": {
                  ...
              }
            }
          }  
        }
      }
    }
  }
}
```

這裡我們顯示隨附預設安裝AEM的資產（JPEG影像），作為範例專案geometrixx的一部分。 資產稱為`jane_doe.jpg`，其mimetype為`image/jpeg`。

若要從HTL記憶體取資產，您可以宣告[`com.day.cq.dam.api.Asset`](https://helpx.adobe.com/experience-manager/6-5/sites/developing/using/reference-materials/javadoc/com/adobe/granite/asset/api/Asset.html)為`data-sly-use`陳述式中的類別，然後使用`Asset`的get方法來擷取所需資訊。 例如：

### `mimetype.html` {#mimetype-html}

```xml
<div data-sly-use.asset="com.day.cq.dam.api.Asset">
  <p>${asset.mimeType}</p>
</div>
```

`data-sly-use`陳述式會指示HTL將目前的`Resource`調整為`Asset`，並為其指定本機名稱`asset`。 然後，它會使用HTL getter速記呼叫`Asset`的`getMimeType`方法：`asset.mimeType`。

### 可從請求{#adaptable-from-request}調整

還可以將任何從[`org.apache.sling.api.SlingHttpServletRequest`](https://helpx.adobe.com/experience-manager/6-5/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/SlingHttpServletRequest.html)可適應的類作為使用類

與上述從`Resource`適應的使用類一樣，可在`data-sly-use`語句中指定從[`SlingHttpServletRequest`](https://helpx.adobe.com/experience-manager/6-5/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/SlingHttpServletRequest.html)適應的使用類。 執行時，目前的要求會與指定的類別相適應，而產生的物件將可在HTL中使用。
