---
title: HTL Java Use-API
description: 'HTML 範本語言 - HTL - Java Use-API 可讓 HTL 檔案存取自訂 Java 類別中的 Helper 方法。 '
translation-type: tm+mt
source-git-commit: f7e46aaac2a4b51d7fa131ef46692ba6be58d878
workflow-type: tm+mt
source-wordcount: '2558'
ht-degree: 1%

---


# HTL Java Use-API {#htl-java-use-api}

HTML範本語言(HTL)Java Use-API可讓HTL檔案透過`data-sly-use`存取自訂Java類別中的輔助方法。 這可讓所有複雜的商業邏輯封裝在Java程式碼中，而HTL程式碼則僅處理直接標籤製作。

Java Use-API物件可以是簡單的POJO，由特定實作透過POJO的預設建構函式執行個體化。

Use-API POJO也可以使用下列簽名來公開名為init的公用方法：

```java
    /**
     * Initializes the Use bean.
     *
     * @param bindings All bindings available to the HTL scripts.
     **/
    public void init(javax.script.Bindings bindings);
```

`bindings`對應可包含物件，這些物件會提供目前執行的HTL指令碼，Use-API物件可用來處理該指令碼。

## 簡單範例{#a-simple-example}

我們將從沒有use-class的HTL元件開始。 它包含一個檔案`/apps/my-example/components/info.html`

### `/apps/my-example/component/info/info.html` {#apps-my-example-component-info-info-html}

```xml
<div>
    <h1>${properties.title}</h1>
    <p>${properties.description}</p>
</div>
```

我們也會為此元件新增一些內容，以便在`/content/my-example/`處演算：

### `http://<host>:<port>/content/my-example.json` {#http-localhost-content-my-example-json}

```java
{
    "sling:resourceType": "my-example/component/info",
    "title": "My Example",
    "description": "This Is Some Example Content."
}
```

存取此內容時，會執行HTL檔案。 在HTL程式碼中，我們使用內容物件`properties`來存取目前資源的`title`和`description`，並顯示它們。 輸出HTML將為：

### `view-source:http://<host>:<port>/content/my-example.html` {#view-source-http-localhost-content-my-example-html}

```xml
<div>
    <h1>My Example</h1>
    <p>This Is Some Example Content.</p>
</div>
```

### 添加使用類{#adding-a-use-class}

**info**&#x200B;元件如其現狀，不需要use-class來執行其（非常簡單）函式。 不過，在某些情況下，您需要執行無法在HTL中完成的工作，因此您需要使用類別。 但請記住：

>[!NOTE]
>
>僅當無法在HTL中單獨執行某項作業時，才應使用use-class。

例如，假設您希望`info`元件顯示資源的`title`和`description`屬性，但全部以小寫表示。 由於HTL沒有用於小寫字串的方法，因此您需要use-class。 我們可以通過添加Java use-class並按如下方式更改`info.html`來完成此操作：

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

在以下幾節中，我們將逐一介紹程式碼的不同部分。

### 本地與捆綁Java類{#local-vs-bundle-java-class}

Java use-class可以用兩種方式安裝：**local**&#x200B;或&#x200B;**bundle**。 此示例使用本地安裝。

在本機安裝中，Java來源檔案會放在HTL檔案旁，位於相同的儲存庫檔案夾中。 系統會根據需求自動編譯來源。 不需要個別編譯或封裝步驟。

在套件安裝中，Java類別必須使用標準AEM套件部署機制，編譯並部署在OSGi套件中（請參閱[套件Java類別](#bundled-java-class)）。

>[!NOTE]
>
>當use-class是相關元件專屬時，建議使用&#x200B;**本機Java use-class**。
>
>當Java代碼實施從多個HTL元件訪問的服務時，建議使用&#x200B;**捆綁Java use-class**。

### Java包是儲存庫路徑{#java-package-is-repository-path}

使用本地安裝時，use-class的軟體包名稱必須與儲存庫資料夾位置的軟體包名稱匹配，路徑中的任何連字元都由軟體包名稱中的下划線替換。

在這種情況下，`Info.java`位於`/apps/my-example/components/info` ，因此軟體包是`apps.my_example.components.info` :

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
>在AEM開發中，建議在儲存庫項目名稱中使用連字型大小。 但是，連字型大小在Java包名稱中是非法的。 因此，儲存庫路徑中的&#x200B;**所有連字元都必須轉換為軟體包名稱**&#x200B;中的下划線。

### 擴展`WCMUsePojo` {#extending-wcmusepojo}

雖然有許多將Java類與HTL結合的方法（請參閱`WCMUsePojo`的替代方案），但最簡單的方式是擴充`WCMUsePojo`類別：

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

通常，[activate](https://helpx.adobe.com/experience-manager/6-5/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUsePojo.html)方法會用於根據目前的上下文（例如，目前的請求和資源）預先計算並儲存（在成員變數中）HTL程式碼中所需的值。

`WCMUsePojo`類提供對HTL檔案中可用的同一組上下文對象的訪問（請參閱[全局對象](global-objects.md)）。

在擴展`WCMUsePojo`的類中，可通過名稱訪問上下文對象，使用

[`<T> T get(String name, Class<T> type)`](https://helpx.adobe.com/experience-manager/6-5/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUsePojo.html)

或者，常用的上下文物件可由適當的&#x200B;**便利性方法**&#x200B;直接存取：

|  |  |
|---|---|
| [PageManager](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/day/cq/wcm/api/PageManager.html) | [getPageManager()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getPageManager()) |
| [頁面](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/day/cq/wcm/api/Page.html) | [getCurrentPage()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getCurrentPage()) |
| [頁面](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/day/cq/wcm/api/Page.html) | [getResourcePage()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getResourcePage()) |
| [ValueMap](https://helpx.adobe.com/experience-manager/6-5/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/resource/ValueMap.html) | [getPageProperties()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getPageProperties()) |
| [ValueMap](https://helpx.adobe.com/experience-manager/6-5/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/resource/ValueMap.html) | [getProperties()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getProperties()) |
| [設計人員](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/day/cq/wcm/api/designer/Designer.html) | [getDesigner()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getDesigner()) |
| [設計](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/day/cq/wcm/api/designer/Design.html) | [getCurrentDesign()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getCurrentDesign()) |
| [樣式](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/day/cq/wcm/api/designer/Style.html) | [getCurrentStyle()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getCurrentStyle()) |
| [元件](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/day/cq/wcm/api/components/Component.html) | [getComponent()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getComponent()) |
| [ValueMap](https://helpx.adobe.com/experience-manager/6-5/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/resource/ValueMap.html) | [getInheritedProperties()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getInheritedProperties) |
| [資源](https://helpx.adobe.com/experience-manager/6-5/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/resource/Resource.html) | [getResource()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getResource()) |
| [資源解析器](https://helpx.adobe.com/experience-manager/6-5/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/resource/ResourceResolver.html) | [getResourceResolver()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getResourceResolver()) |
| [SlingHttpServletRequest](https://helpx.adobe.com/experience-manager/6-5/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/SlingHttpServletRequest.html) | [getRequest()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getRequest()) |
| [SlingHttpServletResponse](https://helpx.adobe.com/experience-manager/6-5/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/SlingHttpServletResponse.html) | [getResponse()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getResponse()) |
| [SlingScriptHelper](https://helpx.adobe.com/experience-manager/6-5/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/scripting/SlingScriptHelper.html) | [getSlingScriptHelper()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getSlingScriptHelper()) |

### Getter方法{#getter-methods}

在use-class初始化後，就會執行HTL檔案。 在此階段中，HTL通常會拉入use-class的各個成員變數的狀態，並將它們轉譯為簡報。

若要從HTL檔案中存取這些值，您必鬚根據下列命名慣例，在use-class中定義自訂getter方法：

* `getXyz`格式的方法將在HTL檔案中公開名為`xyz`的對象屬性。

在以下示例中，`getTitle`和`getDescription`方法使對象屬性`title`和`description`在HTL檔案的上下文中變為可訪問：

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

### data-slyuse屬性{#data-sly-use-attribute}

`data-sly-use`屬性用於初始化HTL程式碼中的use-class。 在我們的示例中， `data-sly-use`屬性聲明我們要使用類`Info`。 我們只能使用類的本地名稱，因為我們使用的是本地安裝（將Java源檔案放在與HTL檔案相同的資料夾中）。 如果我們使用套件安裝，我們必須指定完全限定的類別名稱。

### `/apps/my-example/component/info/info.html` {#apps-my-example-component-info-info-html-2}

```xml
<div data-sly-use.info="Info">
    <h1>${info.lowerCaseTitle}</h1>
    <p>${info.lowerCaseDescription}</p>
</div>
```

### 本機識別碼{#local-identifier}

識別碼`info`（位於`data-sly-use.info`中的點後）用於HTL檔案中以識別類別。 此標識符的範圍在檔案中是全局的，在聲明後。 它不限於包含`data-sly-use`語句的元素。

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

## Beyond The Basics {#beyond-the-basics}

在本節中，我們將介紹一些超越上述簡單範例的其他功能：

* 將參數傳遞至use-class。
* 搭售的Java使用類別。
* `WCMUsePojo`的替代方案

### 傳遞參數{#passing-parameters}

參數可在初始化時傳遞至use-class。 例如，我們可以做這樣的事：

### `/content/my-example/component/info/info.html` {#content-my-example-component-info-info-html}

```xml
<div data-sly-use.info="${'Info' @ text='Some text'}">
    <h1>${info.lowerCaseTitle}</h1>
    <p>${info.lowerCaseDescription}</p>
    <p>${info.upperCaseText}</p>
</div>
```

我們在此傳遞名為`text`的參數。 use-class接著將我們擷取的字串加上上，並顯示結果與`info.upperCaseText`。 以下是已調整的使用類別：

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

此參數是透過`WCMUsePojo`方法[`<T> T get(String paramName, Class<T> type)`](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html)存取

就我們而言，聲明是：

`get("text", String.class)`

然後，該字串會通過以下方法被反轉和公開：

`getReverseText()`

### 僅從data-sly-template {#only-pass-parameters-from-data-sly-template}傳遞參數

雖然上述範例在技術上是正確的，但是當相關值在HTL程式碼的執行內容中可用時，從HTL傳遞值來初始化使用類別實際上並不有意義（或者，實際上，如上所述，該值是靜態的）。

原因是use-class將永遠擁有與HTL程式碼相同執行內容的存取權。 這就引出了最佳實踐的匯入點：

>[!NOTE]
>
>只有當use-class用於`data-sly-template`檔案時，才應將參數傳入use-class，而此檔案本身是從另一個HTL檔案呼叫，且需要傳入參數。

例如，讓我們在現有範例旁建立個別的`data-sly-template`檔案。 我們將調用新檔案`extra.html`。 它包含一個名為`extra`的`data-sly-template`塊：

### `/apps/my-example/component/info/extra.html` {#apps-my-example-component-info-extra-html}

```xml
<template data-sly-template.extra="${@ text}"
          data-sly-use.extraHelper="${'ExtraHelper' @ text=text}">
  <p>${extraHelper.reversedText}</p>
</template>
```

範本`extra`採用單一參數`text`。 然後，它以本地名稱`extraHelper`初始化Java use-class `ExtraHelper`，並將模板參數`text`的值作為use-class參數`text`傳遞給它。

範本的內文會取得屬性`extraHelper.reversedText`（在外罩下方，實際呼叫`ExtraHelper.getReversedText()`），並顯示該值。

我們也會調整現有的`info.html`，以使用此新範本：

### `/apps/my-example/component/info/info.html` {#apps-my-example-component-info-info-html-5}

```xml
<div data-sly-use.info="Info"
     data-sly-use.extra="extra.html">

  <h1>${info.lowerCaseTitle}</h1>
  <p>${info.lowerCaseDescription}</p>

  <div data-sly-call="${extra.extra @ text=properties.description}"></div>

</div>
```

檔案`info.html`現在包含兩個`data-sly-use`語句，其中原始語句導入`Info` Java use-class，新語句導入模板檔案，其本地名稱為`extra`。

請注意，我們可將範本區塊放在`info.html`檔案中，以避免第二個`data-sly-use`，但是個別範本檔案較常見，而且可重複使用。

`Info`類與以前一樣，通過其相應的HTL屬性`info.lowerCaseTitle`和`info.lowerCaseDescription`調用其getter方法`getLowerCaseTitle()`和`getLowerCaseDescription()`。

然後，我們對模板`extra`執行`data-sly-call`，並將值`properties.description`作為參數`text`傳遞。

Java use-class `Info.java`已更改為處理新文本參數：

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

使用`get("text", String.class)`檢索`text`參數，該值會反轉，並通過getter `getReversedText()`作為HTL對象`reversedText`使用。

### 捆綁的Java類{#bundled-java-class}

使用Bundle use-class，必須使用標準OSGi Bundle部署機制，在AEM中編譯、封裝和部署類別。 與本地安裝不同，use-class **package declaration**&#x200B;應正常命名：

### `/apps/my-example/component/info/Info.java` {#apps-my-example-component-info-info-java-6}

```java
package org.example.app.components;

import com.adobe.cq.sightly.WCMUsePojo;

public class Info extends WCMUsePojo {
    ...
}
```

和， `data-sly-use`語句必須引用完全限定的類名，而不只是本地類名：

### `/apps/my-example/component/info/info.html` {#apps-my-example-component-info-info-html-6}

```xml
<div data-sly-use.info="org.example.app.components.info.Info">
  <h1>${info.title}</h1>
  <p>${info.description}</p>
</div>
```

### `WCMUsePojo` {#alternatives-to-wcmusepojo}的替代方案

建立Java use-class的最常見方法是擴展`WCMUsePojo`。 但是，還有許多其他選擇。 要瞭解這些變體，瞭解HTL `data-sly-use`陳述式在引擎蓋下的運作方式會有所幫助。

假設您有下列`data-sly-use`陳述式：

**`<div data-sly-use.`** `localName`**`="`**`UseClass`**`">`**

系統將按以下方式處理語句：

(1)

* 如果與HTL檔案位於同一目錄中存在本地檔案`UseClass.java`，請嘗試編譯並載入該類。 如果成功轉到(2)。
* 否則，請將`UseClass`解譯為完全限定的類名，並嘗試從OSGi環境載入它。 如果成功轉到(2)。
* 否則，請將`UseClass`解譯為HTL或JavaScript檔案的路徑，並載入該檔案。 如果goto(4)成功，

(2)

* 嘗試將當前`Resource`調整為`UseClass`。 如果成功，請轉至(3)。
* 否則，請嘗試將當前`Request`調整為`UseClass`。 如果成功，請轉至(3)。
* 否則，請嘗試使用零參數建構函式實例化`UseClass`。 如果成功，請轉至(3)。

(3)

* 在HTL中，將新調整或建立的物件系結至名稱`localName`。
* 如果`UseClass`實作[`io.sightly.java.api.Use`](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/io/sightly/java/api/Use.html)，則調用`init`方法，傳遞當前執行上下文（以`javax.scripting.Bindings`對象的形式）。

(4)

* 如果`UseClass`是包含`data-sly-template`之HTL檔案的路徑，請準備範本。
* 否則，如果`UseClass`是JavaScript use-class的路徑，請準備use-class（請參閱[JavaScript Use-API](use-api-javascript.md)）。

上述說明的幾個要點：

* 任何可從`Resource`調整、可從`Request`調整，或具有零參數建構函式的類別都可以是use-class。 類不必擴展`WCMUsePojo`甚至實現`Use`。
* 不過，如果use-class *does*&#x200B;實作`Use`，則其`init`方法會自動與目前的內容一起呼叫，讓您將初始化程式碼放在該內容上。
* 擴展`WCMUsePojo`的use-class只是實施`Use`的特殊情況。 它提供了方便的上下文方法，並自動從`Use.init`調用其`activate`方法。

### 直接實施介面使用{#directly-implement-interface-use}

雖然建立use-class的最常見方式是擴展`WCMUsePojo` ，但也可以直接實施[`io.sightly.java.api.Use`](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/io/sightly/java/api/Use.html)介面本身。

`Use`介面僅定義一種方法：

[`public void init(javax.script.Bindings bindings)`](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/io/sightly/java/api/Use#init(javax.script.Bindings))

對於具有`Bindings`對象的類的初始化，將調用`init`方法，該對象保存所有上下文對象和傳遞到use-class中的任何參數。

必須使用[`javax.script.Bindings`](http://docs.oracle.com/javase/7/docs/api/javax/script/Bindings.html)物件明確實作所有其他功能（例如`WCMUsePojo.getProperties()`的等值）。 例如：

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

當您希望將現有類的子類用作use-class時，您可以自行實施`Use`介面，而不是擴展`WCMUsePojo`。

### 從資源{#adaptable-from-resource}調整

另一個選項是使用從`org.apache.sling.api.resource.Resource`可調整的輔助類。

假設您需要編寫HTL指令碼，以顯示DAM資產的中型。 在本例中，您知道在呼叫HTL指令檔時，它會位於`Resource`的內容中，該內容會以nodetype `dam:Asset`包住JCR `Node`。

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

在這裡，我們會顯示AEM預設安裝隨附的資產（JPEG影像），做為範例專案geometrixx的一部分。 資產稱為`jane_doe.jpg`，其mimetype為`image/jpeg`。

若要從HTL中存取資產，您可以將[`com.day.cq.dam.api.Asset`](https://helpx.adobe.com/experience-manager/6-5/sites/developing/using/reference-materials/javadoc/com/adobe/granite/asset/api/Asset.html)宣告為`data-sly-use`陳述式中的類別，然後使用`Asset`的get方法來擷取所需資訊。 例如：

### `mimetype.html` {#mimetype-html}

```xml
<div data-sly-use.asset="com.day.cq.dam.api.Asset">
  <p>${asset.mimeType}</p>
</div>
```

`data-sly-use`陳述式會指示HTL將目前的`Resource`調整為`Asset`，並為它指定本機名稱`asset`。 然後，它使用HTL getter shorthand呼叫`Asset`的`getMimeType`方法：`asset.mimeType`。

### 從請求{#adaptable-from-request}調整

還可以將任何從[`org.apache.sling.api.SlingHttpServletRequest`](https://helpx.adobe.com/experience-manager/6-5/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/SlingHttpServletRequest.html)可適應的類作為使用類使用

與上述從`Resource`適應的use-class一樣，可在`data-sly-use`語句中指定從[`SlingHttpServletRequest`](https://helpx.adobe.com/experience-manager/6-5/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/SlingHttpServletRequest.html)適應的use-class。 執行時，目前的要求將會適應給定的類別，而產生的物件將可在HTL中使用。
