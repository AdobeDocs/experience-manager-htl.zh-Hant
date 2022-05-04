---
title: HTL Java Use-API
description: HTML 範本語言 (HTL) Java Use-API 讓 HTL 檔案能夠存取自訂 Java 類別中的 helper 方法。
exl-id: 9a9a2bf8-d178-4460-a3ec-cbefcfc09959
source-git-commit: 8e70ee4921a7ea071ab7e06947824c371f4013d8
workflow-type: ht
source-wordcount: '2558'
ht-degree: 100%

---

# HTL Java Use-API {#htl-java-use-api}

HTML 範本語言 (HTL) Java Use-API 讓 HTL 檔案能夠透過 `data-sly-use` 存取自訂 Java 類別中的 helper 方法。 如此可讓所有複雜的商業邏輯都封裝在 Java 程式碼中，而 HTL 程式碼只需處理直接標記的產生。

Java Use-API 物件可以是簡單 POJO，由特定實作透過 POJO 的預設建構函式具現化。

Use-API POJO 也可以透過以下簽章公開 public 方法 (稱為 init)：

```java
    /**
     * Initializes the Use bean.
     *
     * @param bindings All bindings available to the HTL scripts.
     **/
    public void init(javax.script.Bindings bindings);
```

`bindings` 對應可以包含一些物件，這些物件為目前執行的 HTL 指令碼提供上下文，以供 Use-API 物件處理之用。

## 簡單範例 {#a-simple-example}

我們先來看一個沒有 use 類別的 HTL 元件。 它是由單一檔案 `/apps/my-example/components/info.html` 所組成

### `/apps/my-example/component/info/info.html` {#apps-my-example-component-info-info-html}

```xml
<div>
    <h1>${properties.title}</h1>
    <p>${properties.description}</p>
</div>
```

我們也會新增此元件的一些內容，以便在 `/content/my-example/` 中呈現：

### `http://<host>:<port>/content/my-example.json` {#http-localhost-content-my-example-json}

```java
{
    "sling:resourceType": "my-example/component/info",
    "title": "My Example",
    "description": "This Is Some Example Content."
}
```

當存取此內容時，就會執行 HTL 檔案。 在 HTL 程式碼中，我們會使用上下文物件 `properties` 來存取目前資源的 `title` 和 `description` 並加以顯示。 輸出 HTML 將會是：

### `view-source:http://<host>:<port>/content/my-example.html` {#view-source-http-localhost-content-my-example-html}

```xml
<div>
    <h1>My Example</h1>
    <p>This Is Some Example Content.</p>
</div>
```

### 新增 use 類別 {#adding-a-use-class}

目前 **info** 元件不需要 use 類別來執行其 (非常簡單的) 函數。 但在某些情況下，您需要做一些在 HTL 中無法完成的事，所以需要 use 類別。 但請牢記以下事項：

>[!NOTE]
>
>只有當某件事無法單獨在 HTL 中完成時，才應該使用 use 類別。

例如，假設您希望 `info` 元件能顯示資源的 `title` 和 `description` 屬性，但全都以小寫字母顯示。 由於 HTL 沒有方法可顯示小寫字串，所以您需要 use 類別。 我們可以新增 Java use 類別並變更 `info.html` 來達成此目的，如下所示：

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

在以下幾節中，我們將逐步說明此程式碼的不同部分。

### 本機與套件 Java 類別的比較 {#local-vs-bundle-java-class}

Java use 類別有兩種安裝方法：**本機**&#x200B;或&#x200B;**套件**。 此範例使用本機安裝。

在本機安裝中，Java 來源檔案與 HTL 檔案一起放在相同的存放庫資料夾中。 來源會自動隨需編譯。 不需要個別的編譯或封裝步驟。

在套件安裝中，必須使用標準 AEM 套件部署機制在 OSGi 套件中編譯及部署 Java 類別 (請參閱[套件式 Java 類別](#bundled-java-class))。

>[!NOTE]
>
>當 use 類別為此處所討論的元件所專屬時，建議使用&#x200B;**本機 Java use 類別**。
>
>當 Java 程式碼實作可從多個 HTL 元件存取的服務時，建議使用&#x200B;**套件 Java use 類別**。

### Java 封裝為存放庫路徑 {#java-package-is-repository-path}

當使用本機安裝時，use 類別的封裝名稱必須符合存放庫資料夾位置的名稱，而且路徑中的所有連字號都替換為封裝名稱中的底線。

在此情況下，`Info.java` 位於 `/apps/my-example/components/info`，所以封裝為 `apps.my_example.components.info`：

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
>AEM 開發中的建議做法是在存放庫項目的名稱中使用連字號。 不過，在 Java 封裝名稱中使用連字號是不合法的。 因此，**存放庫路徑中的所有連字號都必須轉換成封裝名稱中的底線**。

### 擴充 `WCMUsePojo` {#extending-wcmusepojo}

雖然有許多方法可以將 Java 類別與 HTL (請參閱「`WCMUsePojo` 的替代方案」) 合併，最簡單的方法是擴充 `WCMUsePojo` 類別：

#### `/apps/my-example/component/info/Info.java` {#apps-my-example-component-info-info-java-2}

```java
package apps.my_example.components.info;

import com.adobe.cq.sightly.WCMUsePojo;

public class Info extends WCMUsePojo

    ...
}
```

### 初始化類別 {#initializing-the-class}

從 `WCMUsePojo` 擴充 use 類別時，會藉由覆寫 `activate` 方法來執行初始化作業：

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

通常 [activate](https://helpx.adobe.com/tw/experience-manager/6-5/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUsePojo.html) 方法是用來根據目前上下文 (例如目前的請求和資源) 預先計算及儲存 (在成員變數中) HTL 程式碼中所需的值。

`WCMUsePojo` 類別會提供與 HTL 檔案中相同的一組上下文物件的存取權 (請參閱[全域物件](global-objects.md))。

在擴充 `WCMUsePojo` 的類別中，可以使用名稱存取上下文物件

[`<T> T get(String name, Class<T> type)`](https://helpx.adobe.com/tw/experience-manager/6-5/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUsePojo.html)

另外，也可以透過適當的&#x200B;**便利方法**&#x200B;直接存取常用的上下文物件：

|  |  |
|---|---|
| [PageManager](https://helpx.adobe.com/tw/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/day/cq/wcm/api/PageManager.html) | [getPageManager()](https://helpx.adobe.com/tw/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getPageManager()) |
| [Page](https://helpx.adobe.com/tw/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/day/cq/wcm/api/Page.html) | [getCurrentPage()](https://helpx.adobe.com/tw/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getCurrentPage()) |
| [Page](https://helpx.adobe.com/tw/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/day/cq/wcm/api/Page.html) | [getResourcePage()](https://helpx.adobe.com/tw/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getResourcePage()) |
| [ValueMap](https://helpx.adobe.com/tw/experience-manager/6-5/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/resource/ValueMap.html) | [getPageProperties()](https://helpx.adobe.com/tw/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getPageProperties()) |
| [ValueMap](https://helpx.adobe.com/tw/experience-manager/6-5/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/resource/ValueMap.html) | [getProperties()](https://helpx.adobe.com/tw/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getProperties()) |
| [Designer](https://helpx.adobe.com/tw/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/day/cq/wcm/api/designer/Designer.html) | [getDesigner()](https://helpx.adobe.com/tw/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getDesigner()) |
| [Design](https://helpx.adobe.com/tw/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/day/cq/wcm/api/designer/Design.html) | [getCurrentDesign()](https://helpx.adobe.com/tw/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getCurrentDesign()) |
| [Style](https://helpx.adobe.com/tw/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/day/cq/wcm/api/designer/Style.html) | [getCurrentStyle()](https://helpx.adobe.com/tw/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getCurrentStyle()) |
| [Component](https://helpx.adobe.com/tw/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/day/cq/wcm/api/components/Component.html) | [getComponent()](https://helpx.adobe.com/tw/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getComponent()) |
| [ValueMap](https://helpx.adobe.com/tw/experience-manager/6-5/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/resource/ValueMap.html) | [getInheritedProperties()](https://helpx.adobe.com/tw/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getInheritedProperties) |
| [Resource](https://helpx.adobe.com/tw/experience-manager/6-5/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/resource/Resource.html) | [getResource()](https://helpx.adobe.com/tw/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getResource()) |
| [ResourceResolver](https://helpx.adobe.com/tw/experience-manager/6-5/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/resource/ResourceResolver.html) | [getResourceResolver()](https://helpx.adobe.com/tw/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getResourceResolver()) |
| [SlingHttpServletRequest](https://helpx.adobe.com/tw/experience-manager/6-5/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/SlingHttpServletRequest.html) | [getRequest()](https://helpx.adobe.com/tw/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getRequest()) |
| [SlingHttpServletResponse](https://helpx.adobe.com/tw/experience-manager/6-5/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/SlingHttpServletResponse.html) | [getResponse()](https://helpx.adobe.com/tw/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getResponse()) |
| [SlingScriptHelper](https://helpx.adobe.com/tw/experience-manager/6-5/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/scripting/SlingScriptHelper.html) | [getSlingScriptHelper()](https://helpx.adobe.com/tw/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getSlingScriptHelper()) |

### Getter 方法 {#getter-methods}

在初始化 use 類別後，就會執行 HTL 檔案。 在這個階段中，HTL 通常會拉入 use 類別的各種成員變數的狀態，並加以呈現。

若要從 HTL 檔案提供對這些值的存取權，您必須根據以下命名慣例在 use 類別中定義自訂 getter 方法：

* `getXyz` 形式的方法將會在 HTL 檔案中公開一個稱為 `xyz` 的物件屬性。

在以下範例中，`getTitle` 和 `getDescription` 方法會導致 `title` 和 `description` 物件屬性可以在 HTL 檔案的上下文中存取：

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

### data-sly-use 屬性 {#data-sly-use-attribute}

`data-sly-use` 屬性是用來初始化 HTL 程式碼中的 use 類別。 在我們的範例中，`data-sly-use` 屬性會宣告我們想要使用 `Info` 類別。 我們可以僅使用類別的本機名稱，因為我們正在使用本機安裝 (已將 Java 來源檔案放在與 HTL 檔案相同的資料夾中)。 如果我們之前是使用套件安裝，就必須指定完整類別名稱。

### `/apps/my-example/component/info/info.html` {#apps-my-example-component-info-info-html-2}

```xml
<div data-sly-use.info="Info">
    <h1>${info.lowerCaseTitle}</h1>
    <p>${info.lowerCaseDescription}</p>
</div>
```

### 本機識別碼 {#local-identifier}

`info` 識別碼 (`data-sly-use.info` 中的點後面的代碼) 會在 HTL 檔案中用來識別類別。 在宣告此識別碼後，其範圍在檔案中為全域， 不限於包含 `data-sly-use` 陳述式的元素。

### `/apps/my-example/component/info/info.html`{#apps-my-example-component-info-info-html-3}

```xml
<div data-sly-use.info="Info">
    <h1>${info.lowerCaseTitle}</h1>
    <p>${info.lowerCaseDescription}</p>
</div>
```

### 取得屬性 {#getting-properties}

然後會使用 `info` 識別碼來存取之前透過 getter 方法 `Info.getTitle` 和 `Info.getDescription` 所公開的物件屬性 `title` 和 `description`。

### `/apps/my-example/component/info/info.html` {#apps-my-example-component-info-info-html-4}

```xml
<div data-sly-use.info="Info">
    <h1>${info.lowerCaseTitle}</h1>
    <p>${info.lowerCaseDescription}</p>
</div>
```

### 輸出 {#output}

現在，當我們存取 `/content/my-example.html` 時，它會傳回以下 HTML：

### `view-source:http://<host>:<port>/content/my-example.html` {#view-source-http-localhost-content-my-example-html-1}

```xml
<div>
    <h1>my example</h1>
    <p>this is some example content.</p>
</div>
```

## 基本知識以外 {#beyond-the-basics}

在本節中，我們將介紹一些進一步的功能，這些功能超越上面的簡單範例：

* 傳遞參數給 use 類別。
* 套件式 Java use 類別。
* `WCMUsePojo` 的替代方案

### 傳遞參數 {#passing-parameters}

在初始化之後，可以將參數傳遞給 use 類別。 例如，我們可以執行類似以下的操作：

### `/content/my-example/component/info/info.html` {#content-my-example-component-info-info-html}

```xml
<div data-sly-use.info="${'Info' @ text='Some text'}">
    <h1>${info.lowerCaseTitle}</h1>
    <p>${info.lowerCaseDescription}</p>
    <p>${info.upperCaseText}</p>
</div>
```

我們在這裡傳遞一個稱為 `text` 的參數。 然後 use 類別透過 `info.upperCaseText` 將我們擷取的字串變成大寫，並顯示結果。 以下是調整過的 use 類別：

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

透過以下 `WCMUsePojo` 方法存取此參數：[`<T> T get(String paramName, Class<T> type)`](https://helpx.adobe.com/tw/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html)

在我們的案例中，陳述式為：

`get("text", String.class)`

然後透過以下方法反轉及公開字串：

`getReverseText()`

### 只傳遞來自 data-sly-template 的參數 {#only-pass-parameters-from-data-sly-template}

雖然上述範例在技術上是正確的，但實際上從 HTL 傳遞值來初始化 use 類別並沒有太大意義，因為這裡所討論的值是在 HTL 程式碼的執行上下文中所提供 (或更進一步地說，此值是靜態值，如上所述)。

原因是因為 use 類別總是可以存取與 HTL 程式碼相同的執行上下文。 這會帶來最佳實務的重要意義：

>[!NOTE]
>
>只有在以下情況下才應該將參數傳遞給 use 類別：使用 use 類別的 `data-sly-template` 檔案本身透過需要傳遞的參數從其他 HTL 檔案呼叫。

例如，讓我們在現有範例旁邊建立個別 `data-sly-template` 檔案。 我們將新檔案稱為 `extra.html`。 此檔案包含稱為 `extra` 的 `data-sly-template` 區塊：

### `/apps/my-example/component/info/extra.html` {#apps-my-example-component-info-extra-html}

```xml
<template data-sly-template.extra="${@ text}"
          data-sly-use.extraHelper="${'ExtraHelper' @ text=text}">
  <p>${extraHelper.reversedText}</p>
</template>
```

`extra` 範本使用單一參數 `text`。 然後它會使用本機名稱 `extraHelper` 初始化 Java use 類別 `ExtraHelper`，向其傳遞範本參數 `text` 的值作為 use 類別參數 `text`。

範本的主體會取得屬性 `extraHelper.reversedText` (實際上會在幕後呼叫 `ExtraHelper.getReversedText()`) 並顯示該值。

我們也會改寫現有的 `info.html` 來使用這個新範本：

### `/apps/my-example/component/info/info.html` {#apps-my-example-component-info-info-html-5}

```xml
<div data-sly-use.info="Info"
     data-sly-use.extra="extra.html">

  <h1>${info.lowerCaseTitle}</h1>
  <p>${info.lowerCaseDescription}</p>

  <div data-sly-call="${extra.extra @ text=properties.description}"></div>

</div>
```

`info.html` 檔案現在包含兩個 `data-sly-use` 陳述式，原始陳述式會匯入 `Info` Java use 類別，新陳述式會以本機名稱 `extra` 匯入範本檔案。

請注意，我們原本可以將範本區塊放在 `info.html` 檔案內以避免第二個 `data-sly-use`，但個別範本檔案較為常見，而且比較可以重複使用。

`Info` 類別的使用跟之前一樣，透過其對應的 HTL 屬性 `info.lowerCaseTitle` 和 `info.lowerCaseDescription` 呼叫其 getter 方法 `getLowerCaseTitle()` 和 `getLowerCaseDescription()`。

然後我們對 `extra` 範本執行 `data-sly-call`，對其傳遞 `properties.description` 值當作 `text` 參數。

Java use 類別 `Info.java` 會變更，以便處理新的 text 參數：

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

透過 `get("text", String.class)` 擷取 `text` 參數，透過 getter `getReversedText()` 反轉值並當作 HTL 物件 `reversedText` 來使用。

### 套件式 Java 類別 {#bundled-java-class}

使用套件 use 類別時，必須使用標準 OSGi 套件部署機制在 AEM 中編譯、封裝及部署該類別。 相較於本機安裝，use 類別&#x200B;**封裝宣告**&#x200B;應該正常命名：

### `/apps/my-example/component/info/Info.java` {#apps-my-example-component-info-info-java-6}

```java
package org.example.app.components;

import com.adobe.cq.sightly.WCMUsePojo;

public class Info extends WCMUsePojo {
    ...
}
```

而且 `data-sly-use` 陳述式必須參照完整類別名稱，而不只是本機類別名稱：

### `/apps/my-example/component/info/info.html` {#apps-my-example-component-info-info-html-6}

```xml
<div data-sly-use.info="org.example.app.components.info.Info">
  <h1>${info.title}</h1>
  <p>${info.description}</p>
</div>
```

### `WCMUsePojo` 的替代方案 {#alternatives-to-wcmusepojo}

建立 Java use 類別的最常見方式是擴充 `WCMUsePojo`。 不過，還有許多其他選項。 若要了解這些變體，先了解 HTL `data-sly-use` 陳述式在幕後運作的方式會很有用。

假設您有以下 `data-sly-use` 陳述式：

**`<div data-sly-use.`** `localName`**`="`**`UseClass`**`">`**

系統會依以下方式處理陳述式：

(1)

* 如果在與 HTL 檔案相同的目錄中有本機檔案 `UseClass.java` 存在，請嘗試編譯並載入該類別。 如果成功，則前往 (2)。
* 否則，將 `UseClass` 解譯為完整類別名稱，並嘗試從 OSGi 環境載入它。 如果成功，則前往 (2)。
* 否則，將 `UseClass` 解譯為 HTL 或 JavaScript 檔案的路徑，並載入該檔案。 如果成功，則前往 (4)。

(2)

* 嘗試改寫目前的 `Resource` 來配合 `UseClass`。 如果成功，則前往 (3)。
* 否則，嘗試改寫目前的 `Request` 來配合 `UseClass`。 如果成功，則前往 (3)。
* 否則，嘗試使用零引數建構函式具現化 `UseClass`。 如果成功，則前往 (3)。

(3)

* 在 HTL 中，將最新改寫過或建立的物件與 `localName` 名稱綁定。
* 如果 `UseClass` 實作 [`io.sightly.java.api.Use`](https://helpx.adobe.com/tw/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/io/sightly/java/api/Use.html)，則呼叫 `init` 方法，傳遞目前的執行上下文 (採用 `javax.scripting.Bindings` 物件的形式)。

(4)

* 如果 `UseClass` 是包含 `data-sly-template` 的 HTL 檔案的路徑，請準備該範本。
* 否則，當 `UseClass` 是 JavaScript use 類別的路徑時，請準備 use 類別 (請參閱 [JavaScript Use-API](use-api-javascript.md))。

關於以上說明的幾個要點：

* 任何改寫自 `Resource` 或 `Request` 或者具有零引數建構函式的類別都可以是 use 類別。 該類別不必擴充 `WCMUsePojo`，甚至也不必實作 `Use`。
* 不過，如果 use 類別&#x200B;*確實有*&#x200B;實作 `Use`，則會在目前上下文中自動呼叫其 `init` 方法，好讓您可以將與該上下文相依的初始化程式碼放在那裡。
* 擴充 `WCMUsePojo` 的 use 類別是實作 `Use` 的一個特例。 它會提供方便的上下文方法，而且會從 `Use.init` 自動呼叫其 `activate` 方法。

### 直接實作介面 Use {#directly-implement-interface-use}

雖然建立 use 類別的最常見方法是擴充 `WCMUsePojo`，但也可以直接實作 [`io.sightly.java.api.Use`](https://helpx.adobe.com/tw/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/io/sightly/java/api/Use.html) 介面本身。

`Use` 介面只會定義一個方法：

[`public void init(javax.script.Bindings bindings)`](https://helpx.adobe.com/tw/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/io/sightly/java/api/Use#init(javax.script.Bindings))

在使用 `Bindings` 物件初始化類別時，將會呼叫 `init` 方法，該物件會保存所有上下文物件以及傳入 use 類別中的所有參數。

所有其他功能 (例如 `WCMUsePojo.getProperties()` 的同等功能) 都必須使用 [`javax.script.Bindings`](http://docs.oracle.com/javase/7/docs/api/javax/script/Bindings.html) 物件明確地實作。 例如：

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

自行實作 `Use` 介面而不是擴充 `WCMUsePojo` 的主要情況是，當您想要使用已存在類別的子類別當作 use 類別時。

### 從資源改寫 {#adaptable-from-resource}

使用 helper 類別的另一個選項是從 `org.apache.sling.api.resource.Resource` 改寫。

假設您需要撰寫 HTL 指令碼來顯示 DAM 資產的 Mime 類型。 在此情況下，您知道在呼叫您的 HTL 指令碼時，它會在 `Resource` 的上下文中，該資源打包的 JCR `Node` 具有節點類型 `dam:Asset`。

您知道 `dam:Asset` 節點的結構如下所示：

### 存放庫結構 {#repository-structure}

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

我們在這裡顯示該資產 (一張 JPEG 影像)，它隨附在預設 AEM 安裝中，當作範例專案 geometrixx 的一部分。 該資產稱為 `jane_doe.jpg`，其 Mime 類型為 `image/jpeg`。

若要從 HTL 存取資產，您可以將 [`com.day.cq.dam.api.Asset`](https://helpx.adobe.com/tw/experience-manager/6-5/sites/developing/using/reference-materials/javadoc/com/adobe/granite/asset/api/Asset.html) 宣告為 `data-sly-use` 陳述式中的類別，然後使用 `Asset` 的 get 方法擷取所需的資訊。 例如：

### `mimetype.html` {#mimetype-html}

```xml
<div data-sly-use.asset="com.day.cq.dam.api.Asset">
  <p>${asset.mimeType}</p>
</div>
```

`data-sly-use` 陳述式會指示 HTL 改寫目前的 `Resource` 來配合 `Asset`，並為它提供本機名稱 `asset`。 然後它會使用 HTL getter 速記 `asset.mimeType` 來呼叫 `Asset` 的 `getMimeType` 方法。

### 從請求改寫 {#adaptable-from-request}

也可以將任何可從 [`org.apache.sling.api.SlingHttpServletRequest`](https://helpx.adobe.com/tw/experience-manager/6-5/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/SlingHttpServletRequest.html) 改寫的類別當作 use 類別使用

與上述可從 `Resource` 改寫的 use 類別的使用情況一樣，可以在 `data-sly-use` 陳述式中指定可從 [`SlingHttpServletRequest`](https://helpx.adobe.com/tw/experience-manager/6-5/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/SlingHttpServletRequest.html) 改寫的 use 類別。 在執行後，將會改寫目前的請求來配合給定的類別，而且產生的物件將可以在 HTL 中使用。
