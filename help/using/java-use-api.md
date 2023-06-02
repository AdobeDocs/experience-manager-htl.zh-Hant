---
title: HTL Java Use-API
description: HTL Java Use-API 讓 HTL 檔案能夠存取自訂 Java 類別中的 helper 方法。
exl-id: 9a9a2bf8-d178-4460-a3ec-cbefcfc09959
source-git-commit: 88edbd2fd66de960460df5928a3b42846d32066b
workflow-type: ht
source-wordcount: '1510'
ht-degree: 100%

---


# HTL Java Use-API {#htl-java-use-api}

HTL Java Use-API 讓 HTL 檔案能夠存取自訂 Java 類別中的 helper 方法。

## 使用案例 {#use-case}

HTL Java Use-API 讓 HTL 檔案能夠透過 `data-sly-use` 存取自訂 Java 類別中的 helper 方法。 如此可讓所有複雜的商業邏輯都封裝在 Java 程式碼中，而 HTL 程式碼只需處理直接標記的產生。

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

本範例說明 Use-API 的使用情況。

>[!NOTE]
>
>這個範例經過簡化，以利單純說明其用途。 在生產環境中，建議使用 [Sling 模型](https://sling.apache.org/documentation/bundles/models.html)。

我們從稱為 `info` 且沒有 use 類別的 HTL 元件開始。 它是由單一檔案 `/apps/my-example/components/info.html` 所組成

```xml
<div>
    <h1>${properties.title}</h1>
    <p>${properties.description}</p>
</div>
```

我們也會新增此元件的一些內容，以便在 `/content/my-example/` 中呈現：

```xml
{
    "sling:resourceType": "my-example/component/info",
    "title": "My Example",
    "description": "This Is Some Example Content."
}
```

當存取此內容時，就會執行 HTL 檔案。 在 HTL 程式碼中，我們會使用設定語法物件 `properties` 來存取目前資源的 `title` 和 `description` 並顯示出來。 輸出檔案 `/content/my-example.html` 會是：

```html
<div>
    <h1>My Example</h1>
    <p>This Is Some Example Content.</p>
</div>
```

### 新增 Use 類別 {#adding-a-use-class}

`info` 元件目前不需要 use 類別來執行其簡單的函數。但在某些情況下，您需要做一些在 HTL 中無法完成的事，所以需要 use 類別。 但請牢記以下事項：

>[!NOTE]
>
>只有當某件事無法單獨在 HTL 中完成時，才應該使用 use 類別。

例如，假設您希望 `info` 元件能顯示資源的 `title` 和 `description` 屬性，但全都以小寫字母顯示。 由於 HTL 沒有方法可顯示小寫字串，所以您需要 use 類別。 我們可以新增 Java use 類別並變更 `/apps/my-example/component/info/info.html` 來達成此目的，如下所示：

```xml
<div data-sly-use.info="Info">
    <h1>${info.lowerCaseTitle}</h1>
    <p>${info.lowerCaseDescription}</p>
</div>
```

此外，我們也會建立 `/apps/my-example/component/info/Info.java`。

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

如需詳細資訊，請參閱 [`com.adobe.cq.sightly.WCMUsePojo` 的 Javadocs](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/com/adobe/cq/sightly/WCMUsePojo.html)。

現在我們來逐一解說程式碼各個部分。

### 本機與套件 Java 類別 {#local-vs-bundle-java-class}

Java 的 use 類別有兩種安裝方法：

* **本機** - 在本機安裝中，Java 來源檔案與 HTL 檔案一起放在相同的存放庫資料夾中。來源會自動隨需編譯。 不需要個別的編譯或封裝步驟。
* **套件** - 在套件安裝中，您必須使用標準 AEM 套件部署機制在 OSGi 套件中編譯及部署 Java 類別 (請參閱「[套件式 Java 類別](#bundled-java-class)」小節)。

要知道什麼時候使用哪種方法，請記得以下兩個重點：

* 當 use 類別為此處所討論的元件所專屬時，建議使用&#x200B;**本機 Java use 類別**。
* 當 Java 程式碼實作可從多個 HTL 元件存取的服務時，建議使用&#x200B;**套件 Java use 類別**。

此範例使用本機安裝。

### Java 套件是存放庫路徑 {#java-package-is-repository-path}

當使用本機安裝時，use 類別的封裝名稱必須符合存放庫資料夾位置的名稱，而且路徑中的所有連字號都替換為封裝名稱中的底線。

在此情況下，`Info.java` 位於 `/apps/my-example/components/info`，所以封裝為 `apps.my_example.components.info`：

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

雖然有許多方法可以將 Java 類別與 HTL 合併 (請參閱「[`WCMUsePojo`](#alternatives-to-wcmusepojo) 的替代方案」小節)，最簡單的方法卻是擴充 `WCMUsePojo` 類別。例如我們的範例 `/apps/my-example/component/info/Info.java`：

```java
package apps.my_example.components.info;

import com.adobe.cq.sightly.WCMUsePojo;

public class Info extends WCMUsePojo

    ...
}
```

### 初始化類別 {#initializing-the-class}

從 `WCMUsePojo` 擴充 use 類別時，覆寫 `activate` 方法便可以執行初始化，在這個情況下該方法是指 `/apps/my-example/component/info/Info.java`

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

通常 [activate](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/com/adobe/cq/sightly/WCMUsePojo.html) 方法是用來根據目前上下文 (例如目前的請求和資源) 預先計算及儲存 (在成員變數中) HTL 程式碼中所需的值。

`WCMUsePojo` 類別允許存取 HTL 檔案中可用的同一組設定語法物件 (請參閱文件「[全域物件](global-objects.md)」)。

在擴充 `WCMUsePojo` 的類別中，要使用名稱存取設定語法物件時，可使用

[`<T> T get(String name, Class<T> type)`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/com/adobe/cq/sightly/WCMUsePojo.html)

或者，您也可以利用本表格所列的適當的便利方法，直接存取常用的設定語法物件。

| 物件 | 便利方法 |
|---|---|
| [`PageManager`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/com/day/cq/wcm/api/PageManager.html) | [`getPageManager()`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/com/adobe/cq/sightly/WCMUsePojo.html#getPageManager--) |
| [`Page`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/com/day/cq/wcm/api/Page.html) | [`getCurrentPage()`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/com/adobe/cq/sightly/WCMUsePojo.html#getCurrentPage--) |
| [`Page`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/com/day/cq/wcm/api/Page.html) | [`getResourcePage()`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/com/adobe/cq/sightly/WCMUsePojo.html#getResourcePage--) |
| [`ValueMap`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/org/apache/sling/api/resource/ValueMap.html) | [`getPageProperties()`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/com/adobe/cq/sightly/WCMUsePojo.html#getPageProperties--) |
| [`ValueMap`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/org/apache/sling/api/resource/ValueMap.html) | [`getProperties()`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/com/adobe/cq/sightly/WCMUsePojo.html#getProperties--) |
| [`Designer`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/com/day/cq/wcm/api/designer/Designer.html) | [`getDesigner()`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/com/adobe/cq/sightly/WCMUsePojo.html#getDesigner--) |
| [`Design`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/com/day/cq/wcm/api/designer/Design.html) | [`getCurrentDesign()`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/com/adobe/cq/sightly/WCMUsePojo.html#getCurrentDesign--) |
| [`Style`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/com/day/cq/wcm/api/designer/Style.html) | [`getCurrentStyle()`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/com/adobe/cq/sightly/WCMUsePojo.html#getCurrentStyle--) |
| [`Component`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/com/day/cq/wcm/api/components/Component.html) | [`getComponent()`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/com/adobe/cq/sightly/WCMUsePojo.html#getComponent--) |
| [`ValueMap`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/org/apache/sling/api/resource/ValueMap.html) | [`getInheritedProperties()`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/com/adobe/cq/sightly/WCMUsePojo.html#getInheritedPageProperties--) |
| [`Resource`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/org/apache/sling/api/resource/Resource.html) | [`getResource()`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/com/adobe/cq/sightly/WCMUsePojo.html#getResource--) |
| [`ResourceResolver`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/org/apache/sling/api/resource/ResourceResolver.html) | [`getResourceResolver()`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/com/adobe/cq/sightly/WCMUsePojo.html#getResourceResolver--) |
| [`SlingHttpServletRequest`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/org/apache/sling/api/SlingHttpServletRequest.html) | [`getRequest()`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/com/adobe/cq/sightly/WCMUsePojo.html#getRequest--) |
| [`SlingHttpServletResponse`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/org/apache/sling/api/SlingHttpServletResponse.html) | [`getResponse()`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/com/adobe/cq/sightly/WCMUsePojo.html#getResponse--) |
| [`SlingScriptHelper`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/org/apache/sling/api/scripting/SlingScriptHelper.html) | [`getSlingScriptHelper()`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/com/adobe/cq/sightly/WCMUsePojo.html#getSlingScriptHelper--) |

### Getter 方法 {#getter-methods}

在初始化 use 類別後，就會執行 HTL 檔案。 在這個階段中，HTL 通常會拉入 use 類別的各種成員變數的狀態，並加以呈現。

若要允許從 HTL 檔案內部存取這些值，您必須根據以下命名慣例在 use 類別中定義自訂 getter 方法：

* 一個採用 `getXyz` 形式的方法，會在 HTL 檔案中公開一個稱為 `xyz` 的物件屬性。

在以下範例檔案 `/apps/my-example/component/info/Info.java` 中，形成物件屬性 `title` 和 `description` 的 `getTitle` 和 `getDescription` 方法，變成可以在 HTL 檔案的設定語法中存取。

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

請注意這個 `/apps/my-example/component/info/info.html` 範例的使用情況。

```xml
<div data-sly-use.info="Info">
    <h1>${info.lowerCaseTitle}</h1>
    <p>${info.lowerCaseDescription}</p>
</div>
```

### 本機識別碼 {#local-identifier}

`info` 識別碼 (`data-sly-use.info` 中的點後面的代碼) 會在 HTL 檔案中用來識別類別。 在宣告此識別碼後，其範圍在檔案中為全域， 不限於包含 `data-sly-use` 陳述式的元素。

請注意這個 `/apps/my-example/component/info/info.html` 範例的使用情況。

```xml
<div data-sly-use.info="Info">
    <h1>${info.lowerCaseTitle}</h1>
    <p>${info.lowerCaseDescription}</p>
</div>
```

### 取得屬性 {#getting-properties}

接著會使用識別碼 `info` 來存取之前透過 getter 方法 `Info.getTitle` 和 `Info.getDescription` 所公開的物件屬性 `title` 和 `description`。

請注意這個 `/apps/my-example/component/info/info.html` 範例的使用情況。

```xml
<div data-sly-use.info="Info">
    <h1>${info.lowerCaseTitle}</h1>
    <p>${info.lowerCaseDescription}</p>
</div>
```

### 輸出 {#output}

現在，當我們存取 `/content/my-example.html` 時，它會傳回以下 `/content/my-example.html` 檔案。

```xml
<div>
    <h1>my example</h1>
    <p>this is some example content.</p>
</div>
```

>[!NOTE]
>
>這個範例經過簡化，以利單純說明其用途。在生產環境中，建議使用 [Sling 模型](https://sling.apache.org/documentation/bundles/models.html)。

## 超越基本知識 {#beyond-the-basics}

本節進一步介紹更多功能，比之前敘述的簡單範例更加深入。

* 傳遞參數給 use 類別
* 套件式 Java use 類別

### 傳遞參數 {#passing-parameters}

在初始化之後，可以將參數傳遞給 use 類別。

如需詳細資訊，請參閱 Sling [HTL Scripting Engine 文件](https://sling.apache.org/documentation/bundles/scripting/scripting-htl.html#passing-parameters-to-java-use-objects)。

### 套件式 Java 類別 {#bundled-java-class}

使用套件式 use 類別時，必須使用標準 OSGi 套件部署機制在 AEM 中編譯、封裝及部署該類別。與本機安裝不同，use 類別套件宣告應該以一般方式命名，如同此 `/apps/my-example/component/info/Info.java` 範例。

```java
package org.example.app.components;

import com.adobe.cq.sightly.WCMUsePojo;

public class Info extends WCMUsePojo {
    ...
}
```

而且 `data-sly-use` 陳述式必須參照完整類別名稱，而不只是本機類別名稱，如同此 `/apps/my-example/component/info/info.html` 範例。

```xml
<div data-sly-use.info="org.example.app.components.info.Info">
  <h1>${info.title}</h1>
  <p>${info.description}</p>
</div>
```
