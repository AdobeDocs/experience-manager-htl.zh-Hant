---
title: HTL Java Use-API
seo-title: HTL Java Use-API
description: HTML範本語言- HTL- Java使用API可讓HTL檔案存取自訂Java類別中的helper方法。
seo-description: HTML範本語言- HTL- Java使用API可讓HTL檔案存取自訂Java類別中的helper方法。
uuid: b340f8f7-a193-45c8-aa39-5c6 e2 c0144 ea
contentOwner: 使用者
products: SG_ PERIENCENCENAGER/HTL
topic-tags: html-template-language
content-type: 引用
discoiquuid: 126ebc9d-5f7b-7a4-aea2-c8840 d34864 c
mwpw-migration-script-version: 2017-10-12T214658.665-0400
translation-type: tm+mt
source-git-commit: 271c355ae56e16e309853b02b8ef09f2ff971a2e

---


# HTL Java Use-API{#htl-java-use-api}

HTML範本語言(HTL) Java使用API可讓HTL檔案存取自訂Java類別中的helper方法。這可讓所有複雜的商業邏輯封裝在Java程式碼中，而HTL程式碼只會與直接標記製作搭配使用。

## 簡單範例 {#a-simple-example}

我們將從沒有使用類別 *的HTL* 元件開始。它包含一個檔案， `/apps/my-example/components/info.html`

### `/apps/my-example/component/info/info.html` {#apps-my-example-component-info-info-html}

```xml
<div>
    <h1>${properties.title}</h1>
    <p>${properties.description}</p>
</div>
```

我們也會新增部分內容， **`/content/my-example/`**以便在下列位置轉換：

### `http://localhost:4502/content/my-example.json` {#http-localhost-content-my-example-json}

```java
{
    "sling:resourceType": "my-example/component/info",
    "title": "My Example",
    "description": "This Is Some Example Content."
}
```

存取此內容時，會執行HTL檔案。在HTL程式碼中，我們使用上下文物件****`properties`來存取目前資源 `title` 並 `description` 顯示它們。輸出HTML將為：

### `view-source:http://localhost:4502/content/my-example.html` {#view-source-http-localhost-content-my-example-html}

```xml
<div>
    <h1>My Example</h1>
    <p>This Is Some Example Content.</p>
</div>
```

### 新增使用類別 {#adding-a-use-class}

此 **資訊** 元件不需要使用類別來執行其(非常簡單)函數。但是，在某些情況下，您需要完成無法在HTL中完成的工作，因此需要使用類別。但請記住：

>[!NOTE]
>
>*僅當無法在HTL中完成某事時，才應使用類別。*

例如，假設您要 `info` 元件顯示資源的屬性 `title` 和 **`description`** 屬性，但全部以小寫顯示。由於HTL沒有小寫字串的方法，所以您需要使用類別。若要這麼做，我們可以新增Java使用類別並變更如下 **`info.html`** ：

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

在下列章節中，我們會逐步檢視程式碼的不同部分。

### 本機與Bundle Java類別 {#local-vs-bundle-java-class}

Java使用類別可透過兩種方式安裝： **本機** 或 **搭售**。*此範例使用本機安裝。*

在本機安裝中，Java來源檔案會與HTL檔案一起放置在同一個存放庫檔案夾中。來源會根據需求自動編譯。不需要個別編譯或封裝步驟。

在安裝套件時，必須使用標準的AEM Bundle部署機制(請參閱 [搭售的Java類別](#bundled-java-class))，將Java類別編譯並部署在OSGi套裝中。

>[!NOTE]
>
>當使用類別特定元件時，建議使用 **本機Java使用類別** 。
>
>當Java程式碼實作從多個HTL元件存取的服務時，建議 **將Java使用類別** 。

### Java套件是存放庫路徑 {#java-package-is-repository-path}

使用本機安裝時，使用類別的套件名稱必須符合存放庫檔案夾位置的名稱，而路徑中的任何連字號則會以套件名稱中的底線取代。

在此案例 **`Info.java`** 中， **`/apps/my-example/components/info`** 此套件 **`apps.my_example.components.info`** 為：

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
>在AEM開發中建議使用存放庫項目名稱中的連字號。但是，連字號在Java套件名稱中是非法的。因此，存放庫路徑中 **所有連字號都必須轉換為套件名稱**中的底線。

### 擴充功能 `WCMUsePojo`{#extending-wcmusepojo}

雖然有許多方式可將Java類別與HTL搭配使用(請參閱替代項目) `WCMUsePojo`，最簡單的是擴充 `WCMUsePojo` 類別：

#### `/apps/my-example/component/info/Info.java` {#apps-my-example-component-info-info-java-2}

```java
package apps.my_example.components.info;
 
import com.adobe.cq.sightly.WCMUsePojo;
 
public class Info extends WCMUsePojo {
    
    ...
}
```

### 初始化類別 {#initializing-the-class}

從使用類別擴充時 **`WCMUsePojo`**，會覆寫 **`activate`** 方法來執行初始化：

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

通常 [，啓動](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html) 方法會用來根據目前內容(例如目前的請求和資源)，預先計算並儲存HTL程式碼中所需的值。

`WCMUsePojo` 類別可讓您存取與HTL檔案中可用的相同上下文物件集(請參閱 [全域物件](global-objects.md))。

在延伸 **`WCMUsePojo`**的類別中，可以使用名稱來存取 *上下文* 物件

[`<T> T get(String name, Class<T> type)`](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html)

或者，您可以使用適當 **的便利方法直接存取常用的上下文物件**：

|  |
|---|---|
| [PageManager](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/day/cq/wcm/api/PageManager.html) | [getPageManager()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getPageManager()) |
| [頁面](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/day/cq/wcm/api/Page.html) | [getCurrentPage()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getCurrentPage()) |
| [頁面](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/day/cq/wcm/api/Page.html) | [getResourcePage()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getResourcePage()) |
| [ValueMap](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/resource/ValueMap.html) | [getPageProperties()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getPageProperties()) |
| [ValueMap](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/resource/ValueMap.html) | [getProperties()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getProperties()) |
| [Designer](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/day/cq/wcm/api/designer/Designer.html) | [getDesigner()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getDesigner()) |
| [設計](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/day/cq/wcm/api/designer/Design.html) | [getCurrentDesign()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getCurrentDesign()) |
| [樣式](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/day/cq/wcm/api/designer/Style.html) | [getCurrentStyle()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getCurrentStyle()) |
| [元件](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/day/cq/wcm/api/components/Component.html) | [getComponent()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getComponent()) |
| [ValueMap](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/resource/ValueMap.html) | [getInDesign屬性()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse#getInheritedProperties.html()) |
| [資源](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/resource/Resource.html) | [getResource()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getResource()) |
| [ResourceSolver](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/resource/ResourceResolver.html) | [getResourceSolver()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getResourceResolver()) |
| [slingttpServletRequest](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/SlingHttpServletRequest.html) | [getRequest()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getRequest()) |
| [slingttpServletResponse](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/SlingHttpServletResponse.html) | [getResponse()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getResponse()) |
| [SlingScriptHelper](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/scripting/SlingScriptHelper.html) | [getSlingScriptHelper()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getSlingScriptHelper()) |

### getter方法 {#getter-methods}

在使用類別初始化後，就會執行HTL檔案。在此階段中，HTL通常會提取使用類別中各種成員變數的狀態，並將它們轉換為簡報。

若要從HTL檔案中存取這些值，您必須根據下列命名慣例，在使用類別 **中定義自訂getter方法**：

* 表單 **`getXyz`** 的方法會在HTL檔案中公開，名為 ***xyz***。

例如，在下列範例中，這些方法 **`getTitle`** 會 **`getDescription`** 出現在物件屬性 **`title`** 中，並 **`description`** 可在HTL檔案的內容中存取：

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

### data-sly-use屬性 {#data-sly-use-attribute}

**`data-sly-use`** 此屬性用於初始化HTL程式碼中的use-class。在我們的範例中， `data-sly-use` 屬性會宣告我們要使用類別 **`Info`**。我們只能使用本類別的本機名稱，因為我們使用本機安裝(放置Java來源檔案的資料夾與HTL檔案位於相同的檔案夾中)。如果我們使用了搭售功能，我們必須指定完整的合格類別名稱(請參閱 [「使用類別套件安裝](#LocalvsBundleJavaClass)」)。

### `/apps/my-example/component/info/info.html` {#apps-my-example-component-info-info-html-2}

```xml
<div data-sly-use.info="Info">
    <h1>${info.lowerCaseTitle}</h1>
    <p>${info.lowerCaseDescription}</p>
</div>
```

### 本機識別碼 {#local-identifier}

在HTL檔案中使用識別碼「**`info`****`data-sly-use.info`**(點後)」來識別類別。此識別碼的範圍在檔案宣告後即為全域。它不限於包含 `data-sly-use` 陳述式的元素。

### `/apps/my-example/component/info/info.html`{#apps-my-example-component-info-info-html-3}

```xml
<div data-sly-use.info="Info">
    <h1>${info.lowerCaseTitle}</h1>
    <p>${info.lowerCaseDescription}</p>
</div>
```

### 取得屬性 {#getting-properties}

接著識別碼 `info` 會用來存取物件屬性 **`title`** ，並 **`description`** 透過getter方法 `Info.getTitle` 公開 **`Info.getDescription`**。

### `/apps/my-example/component/info/info.html` {#apps-my-example-component-info-info-html-4}

```xml
<div data-sly-use.info="Info">
    <h1>${info.lowerCaseTitle}</h1>
    <p>${info.lowerCaseDescription}</p>
</div>
```

### 輸出 {#output}

現在，當我們存取時 **`/content/my-example.html`** ，將傳回下列HTML：

### `view-source:http://localhost:4502/content/my-example.html` {#view-source-http-localhost-content-my-example-html-1}

```xml
<div>
    <h1>my example</h1>
    <p>this is some example content.</p>
</div>
```

## 超越基本概念 {#beyond-the-basics}

在本節中，我們將進一步介紹以上簡單範例：

* 傳遞參數至使用類別。
* 套裝Java使用類別。
* 替代以外的替代項目 `WCMUsePojo`

### 傳遞參數 {#passing-parameters}

參數可在初始化時傳遞至使用類別。例如，我們可以這麼做：

### `/content/my-example/component/info/info.html` {#content-my-example-component-info-info-html}

```xml
<div data-sly-use.info="${'Info' @ text='Some text'}">
    <h1>${info.lowerCaseTitle}</h1>
    <p>${info.lowerCaseDescription}</p>
    <p>${info.upperCaseText}</p>
</div>
```

在此處，我們將傳遞呼叫的參數 **`text`**。使用類別，然後我們會擷取擷取的字串 `info.upperCaseText`並顯示結果。以下是調整的使用類別：

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

參數可透過方法 `WCMUsePojo` 存取

[ `<T> T get(String paramName, Class<T> type)`](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html)

在本公司案例中，

`get("text", String.class)`

字串接著會透過方法反轉及公開

**`getReverseText()`**

### 僅從資料範本範本傳遞參數 {#only-pass-parameters-from-data-sly-template}

上述範例在技術上是正確的，因此從HTL傳遞值以初始化使用類別並沒有意義，當相關的值可在HTL程式碼的執行上下文中使用時(或，也就是因為上述值是靜態的)。

原因是use-class永遠都能存取與HTL程式碼相同的執行上下文。這會產生最佳做法的匯入點：

>[!NOTE]
>
>將參數傳遞至use-class時，應僅在 **資料-sly範本** 檔案中使用該類別時，才會在呼叫另一個HTL檔案中的參數時完成。

例如，讓我們在現有的範例旁邊建立另一 `data-sly-template` 個檔案。我們會呼叫新檔案 `extra.html`。它包含呼叫的 **`data-sly-template`** 區塊 **`extra`**：

### `/apps/my-example/component/info/extra.html` {#apps-my-example-component-info-extra-html}

```xml
<template data-sly-template.extra="${@ text}"
          data-sly-use.extraHelper="${'ExtraHelper' @ text=text}">
  <p>${extraHelper.reversedText}</p>
</template>
```

範本 **`extra`**會採用單一參數 **`text`**。接著，它會以本機名稱 `ExtraHelper`**`extraHelper`** 初始化Java使用類別，並將範本參數的值傳遞 **`text`** 給use-class參數 **`text`**。

範本的主體會取得屬性 `extraHelper.reversedText` (在此範圍下，實際呼叫) `ExtraHelper.getReversedText()`並顯示該值。

我們也會改寫現有 **`info.html`** 的範本使用方式：

### `/apps/my-example/component/info/info.html` {#apps-my-example-component-info-info-html-5}

```xml
<div data-sly-use.info="Info" 
     data-sly-use.extra="extra.html">
    
  <h1>${info.lowerCaseTitle}</h1>
  <p>${info.lowerCaseDescription}</p>
    
  <div data-sly-call="${extra.extra @ text=properties.description}"></div>

</div>
```

檔案 `info.html` 現在包含兩 **`data-sly-use`** 個陳述式，原始檔案會匯入 **`Info`** Java使用類別，而另一個陳述式會在本機名稱下匯入範本檔案 `extra`。

請注意，我們可以將範本區塊放置 **`info.html`** 在檔案中，以避免第二 **`data-sly-use`**個範本，但另一個範本檔案較常用，而且更容易重復使用。

The **`Info`** class is employed as before, calling its getter methods **`getLowerCaseTitle()`** and `getLowerCaseDescription()` through their corresponding HTL properties `info.lowerCaseTitle` and **`info.lowerCaseDescription`**.

然後，我們 **`data-sly-call`** 執行範本 **`extra`** 並將其傳遞給 `properties.description` 參數作為參數 **`text`**。

Java使用類別 `Info.java` 會變更，以處理新的文字參數：

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

**`text`** 系統會擷取 **`get("text", String.class)`**參數，值會反轉，並透過getter提供HTL物件 `reversedText``getReversedText()`。

### 套裝Java類別 {#bundled-java-class}

透過套件使用類別，類別必須使用標準OSGi套件部署機制來編譯、封裝並部署在AEM中。與本機安裝相反的是，使用類別 **套件宣告** 應該命名為一般：

### `/apps/my-example/component/info/Info.java` {#apps-my-example-component-info-info-java-6}

```java
package org.example.app.components;
 
import com.adobe.cq.sightly.WCMUsePojo;
 
public class Info extends WCMUsePojo {
    ...
}
```

而且 `data-sly-use` ，此陳述式必須參照 *完整的合格類別名稱*，而不只是本端類別名稱：

### `/apps/my-example/component/info/info.html` {#apps-my-example-component-info-info-html-6}

```xml
<div data-sly-use.info="org.example.app.components.info.Info">
  <h1>${info.title}</h1>
  <p>${info.description}</p>
</div>
```

### 替代以外的替代項目 `WCMUsePojo`{#alternatives-to-wcmusepojo}

建立Java使用類別最常見的方法就是擴充 `WCMUsePojo`。不過，還有一些其他選項。若要瞭解這些變體，有助於瞭解HTL `data-sly-use` 陳述式在此套裝下的運作方式。

假設您有下列 `data-sly-use` 陳述式：

**`<div data-sly-use.`** `localName`**`="`** `UseClass`**`">`**

系統會處理陳述式，如下所示：

(1)

* 如果本機檔案 *useClass. java* 與HTL檔案位於相同目錄中，請嘗試編譯並載入該類別。如果成功，請至(2)。

* 否則，請將 *useClass解譯* 為完全合格的類別名稱，並嘗試從OSGi環境載入它。如果成功，請至(2)。
* 否則，請將 *useClass解譯* 為HTL或JavaScript檔案的路徑，並載入該檔案。如果成功的goto(4)。

(2)

* 嘗試調整目前 **`Resource`** 是否 *`UseClass.`* 成功，請前往(3)。

* 否則，請嘗試調整目前 **`Request`** 的內容 *`UseClass`*。如果成功，請前往(3)。

* 否則，請嘗試 *`UseClass`* 使用零引數建構函式進行個體化。如果成功，請前往(3)。

(3)

* 在HTL內，將新改編或建立的物件系結至名稱 *`localName`*。
* 如果 *`UseClass`* 實施 [`io.sightly.java.api.Use`](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/io/sightly/java/api/Use.html) ，則呼叫 `init` 方法，傳遞目前的執行內容(以 `javax.scripting.Bindings` 物件的形式)。

(4)

* 如果 *`UseClass`* 是包含A `data-sly-template`的HTL檔案路徑，請準備範本。

* 否則，如果 *`UseClass`* 是JavaScript使用類別的路徑，請準備使用類別(請參閱 [JavaScript Use-API](use-api-javascript.md))。

上述說明的幾個重點：

* 任何可自適應、 `Resource`自適應或 `Request`具有零引數建構函式的類別都可以是使用類別。類別不需要擴充或 `WCMUsePojo` 甚至建置 `Use`。

* 不過，如果使用類別 *，* 則會 `Use`自動呼叫其 **`init`** 方法，以配合目前的上下文，讓您將初始化程式碼放置在該內容視該上下文而定。

* 延伸 `WCMUsePojo` 的使用類別只是實施 **`Use`**的特殊個案。它提供便利的上下文方法，並自動呼叫其 **`activate`** 方法 `Use.init`。

### 直接實作介面使用 {#directly-implement-interface-use}

雖然建立使用類別最常見的方法是擴充， `WCMUsePojo`但也可以直接實作 `[io.sightly.java.api.Use](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/io/sightly/java/api/Use.html)`介面本身。

`Use` 介面只定義一種方法：

`[public void init(javax.script.Bindings bindings)](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/io/sightly/java/api/Use#init(javax.script.Bindings))`

**`init`** 在初始化類別時，會呼叫包含所有上下文物件和任何傳遞至使用類別之參數的 **`Bindings`** 物件。

所有其他功能(例如等同於 `WCMUsePojo.getProperties()`)都必須明確地使用 ` [javax.script.Bindings](http://docs.oracle.com/javase/7/docs/api/javax/script/Bindings.html)` 物件。例如：

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

當您想要使用現有類別的子類別做為使用類別時，要自己實施 `Use` 介面而不是擴充 `WCMUsePojo` 。

### 從資源中調整 {#adaptable-from-resource}

另一個選擇是使用自適應的輔助類別 **`org.apache.sling.api.resource.Resource`**。

假設您需要撰寫HTL指令碼，以顯示DAM資產的MIMI類型。在這種情況下，您知道當呼叫HTL指令碼時，它就會在包含 **`Resource`** nodeType的JCR **`Node`** 的環境中 **`dam:Asset`**。

您知道 **`dam:Asset`** 節點的結構如下：

### 儲存庫結構 {#repository-structure}

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

在這裡我們會顯示預設安裝AEM做為專案幾何圖庫的資產(JPEG影像)。資產被呼叫 **`jane_doe.jpg`** ，其mimame類型 **`image/jpeg`**為。

若要從HTL存取資產，您可以在陳述式中宣告 ` [com.day.cq.dam.api.Asset](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/granite/asset/api/Asset.html)` 為 **`data-sly-use`** 類別：然後使用取得 **`Asset`** 方法擷取所需資訊。例如：

### `mimetype.html` {#mimetype-html}

```xml
<div data-sly-use.asset="com.day.cq.dam.api.Asset">
  <p>${asset.mimeType}</p>
</div>
```

`data-sly-use` 此陳述式會指示HTL將目前 **`Resource`** 的內容調整為一個， **`Asset`** 並提供本機名稱 **`asset`**。然後呼叫使用HTL **`getMimeType`**`Asset` getter shorthand的方法： `asset.mimeType`。

### 從請求中調整 {#adaptable-from-request}

此外，您也可以使用 **` [org.apache.sling.api.SlingHttpServletRequest](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/SlingHttpServletRequest.html)`**

與上述使用類別相容的情況一樣， `Resource`可在 [`SlingHttpServletRequest`](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/SlingHttpServletRequest.html)`data-sly-use` 陳述式中指定使用類別的適應性。在執行時，目前的要求將會調整為指定的類別，而產生的物件將可在HTL中提供。
