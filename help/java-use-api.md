---
title: HTL Java Use-API
description: HTL Java Use-API使HTL檔案能夠訪問自定義Java類中的幫助程式方法。
exl-id: 9a9a2bf8-d178-4460-a3ec-cbefcfc09959
source-git-commit: da2375a2390f0254dba9745d9f4970e88788e5d5
workflow-type: tm+mt
source-wordcount: '1518'
ht-degree: 39%

---


# HTL Java Use-API {#htl-java-use-api}

HTL Java Use-API使HTL檔案能夠訪問自定義Java類中的幫助程式方法。

## 使用案例 {#use-case}

HTL Java Use-API使HTL檔案能夠通過以下方式訪問自定義Java類中的幫助程式方法： `data-sly-use`。 如此可讓所有複雜的商業邏輯都封裝在 Java 程式碼中，而 HTL 程式碼只需處理直接標記的產生。

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

此示例說明了Use-API的用法。

>[!NOTE]
>
>為了簡化此示例，以便簡單說明其使用。 在生產環境中，建議使用 [吊帶模型。](https://sling.apache.org/documentation/bundles/models.html)

我們從HTL元件開始 `info`沒有用類。 它是由單一檔案 `/apps/my-example/components/info.html` 所組成

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

當存取此內容時，就會執行 HTL 檔案。 在HTL代碼中，我們使用上下文對象 `properties` 訪問當前資源 `title` 和 `description` 並顯示。 輸出檔案 `/content/my-example.html` 將：

```html
<div>
    <h1>My Example</h1>
    <p>This Is Some Example Content.</p>
</div>
```

### 新增 use 類別 {#adding-a-use-class}

的 `info` 元件本身不需要use類來執行其簡單功能。 但在某些情況下，您需要做一些在 HTL 中無法完成的事，所以需要 use 類別。 但請牢記以下事項：

>[!NOTE]
>
>只有當某件事無法單獨在 HTL 中完成時，才應該使用 use 類別。

例如，假設您希望 `info` 元件能顯示資源的 `title` 和 `description` 屬性，但全都以小寫字母顯示。 由於 HTL 沒有方法可顯示小寫字串，所以您需要 use 類別。 我們可以通過添加Java use-class和更改 `/apps/my-example/component/info/info.html` 如下：

```xml
<div data-sly-use.info="Info">
    <h1>${info.lowerCaseTitle}</h1>
    <p>${info.lowerCaseDescription}</p>
</div>
```

此外，我們還建立 `/apps/my-example/component/info/Info.java`。

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

請參閱 [Javadoc `com.adobe.cq.sightly.WCMUsePojo`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/com/adobe/cq/sightly/WCMUsePojo.html) 的子菜單。

現在，我們來看一下代碼的不同部分。

### 本機與套件 Java 類別的比較 {#local-vs-bundle-java-class}

Java use-class可以通過兩種方式安裝：

* **本地**  — 在本地安裝中，Java源檔案與HTL檔案並置，位於同一儲存庫資料夾中。 來源會自動隨需編譯。 不需要個別的編譯或封裝步驟。
* **捆綁**  — 在捆綁包安裝中，必須使用標準捆綁包部署機制在OSGi捆綁包內編譯和部AEM署Java類(請參見 [捆綁的Java類](#bundled-java-class))。

要知道何時使用哪種方法，請記住以下兩點：

* 當 use 類別為此處所討論的元件所專屬時，建議使用&#x200B;**本機 Java use 類別**。
* 當 Java 程式碼實作可從多個 HTL 元件存取的服務時，建議使用&#x200B;**套件 Java use 類別**。

此範例使用本機安裝。

### Java包是儲存庫路徑 {#java-package-is-repository-path}

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

雖然有多種方法將Java類與HTL合併（請參見一節） [替代 `WCMUsePojo`](#alternatives-to-wcmusepojo))，最簡單的是 `WCMUsePojo` 類。 例如 `/apps/my-example/component/info/Info.java`:

```java
package apps.my_example.components.info;

import com.adobe.cq.sightly.WCMUsePojo;

public class Info extends WCMUsePojo

    ...
}
```

### 初始化類 {#initializing-the-class}

當use-class從 `WCMUsePojo`，初始化通過覆蓋 `activate` 方法在本例中 `/apps/my-example/component/info/Info.java`

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

的 `WCMUsePojo` 類提供對HTL檔案中可用的同一組上下文對象的訪問(請參見文檔 [全局對象。](global-objects.md))

在擴充 `WCMUsePojo` 的類別中，可以使用名稱存取上下文物件

[`<T> T get(String name, Class<T> type)`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/com/adobe/cq/sightly/WCMUsePojo.html)

或者，常用上下文對象可以通過下表中列出的適當方便方法直接訪問。

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

### Getter方法 {#getter-methods}

在初始化 use 類別後，就會執行 HTL 檔案。 在這個階段中，HTL 通常會拉入 use 類別的各種成員變數的狀態，並加以呈現。

要從HTL檔案中訪問這些值，必鬚根據以下命名約定在use-class中定義自定義getter方法：

* `getXyz` 形式的方法將會在 HTL 檔案中公開一個稱為 `xyz` 的物件屬性。

在以下示例檔案中 `/apps/my-example/component/info/Info.java`，方法 `getTitle` 和 `getDescription` 導致對象屬性 `title` 和 `description` 在HTL檔案的上下文中可訪問。

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

### 資料漏用屬性 {#data-sly-use-attribute}

`data-sly-use` 屬性是用來初始化 HTL 程式碼中的 use 類別。 在我們的範例中，`data-sly-use` 屬性會宣告我們想要使用 `Info` 類別。 我們可以僅使用類別的本機名稱，因為我們正在使用本機安裝 (已將 Java 來源檔案放在與 HTL 檔案相同的資料夾中)。 如果我們之前是使用套件安裝，就必須指定完整類別名稱。

請注意此中的用法 `/apps/my-example/component/info/info.html` 示例。

```xml
<div data-sly-use.info="Info">
    <h1>${info.lowerCaseTitle}</h1>
    <p>${info.lowerCaseDescription}</p>
</div>
```

### 本地標識符 {#local-identifier}

`info` 識別碼 (`data-sly-use.info` 中的點後面的代碼) 會在 HTL 檔案中用來識別類別。 在宣告此識別碼後，其範圍在檔案中為全域， 不限於包含 `data-sly-use` 陳述式的元素。

請注意此中的用法 `/apps/my-example/component/info/info.html` 示例。

```xml
<div data-sly-use.info="Info">
    <h1>${info.lowerCaseTitle}</h1>
    <p>${info.lowerCaseDescription}</p>
</div>
```

### 獲取屬性 {#getting-properties}

然後會使用 `info` 識別碼來存取之前透過 getter 方法 `Info.getTitle` 和 `Info.getDescription` 所公開的物件屬性 `title` 和 `description`。

請注意此中的用法 `/apps/my-example/component/info/info.html` 示例。

```xml
<div data-sly-use.info="Info">
    <h1>${info.lowerCaseTitle}</h1>
    <p>${info.lowerCaseDescription}</p>
</div>
```

### 輸出 {#output}

現在，當我們 `/content/my-example.html` 它將返回以下 `/content/my-example.html` 的子菜單。

```xml
<div>
    <h1>my example</h1>
    <p>this is some example content.</p>
</div>
```

>[!NOTE]
>
>為了簡單說明其使用，對本例進行了簡化。 在生產環境中，建議使用 [吊帶模型。](https://sling.apache.org/documentation/bundles/models.html)

## 超越基礎 {#beyond-the-basics}

本節介紹了一些超出前面介紹的簡單示例的其他功能。

* 將參數傳遞到use-class
* 捆綁的Java使用類

### 傳遞參數 {#passing-parameters}

在初始化之後，可以將參數傳遞給 use 類別。 例如，我們可以執行類似以下的操作：

有關詳細資訊，請參閱Sling [HTL指令碼引擎文檔。](https://sling.apache.org/documentation/bundles/scripting/scripting-htl.html#passing-parameters-to-java-use-objects)

### 套件式 Java 類別 {#bundled-java-class}

使用捆綁的use-class ，必須使用標準OSGi捆綁包部署機AEM制編譯、打包和部署類。 與本地安裝不同， use-class軟體包聲明的命名通常應與本例中的名稱相同 `/apps/my-example/component/info/Info.java` 示例。

```java
package org.example.app.components;

import com.adobe.cq.sightly.WCMUsePojo;

public class Info extends WCMUsePojo {
    ...
}
```

還有 `data-sly-use` 語句必須引用完全限定的類名，而不是僅引用本地類名，如 `/apps/my-example/component/info/info.html` 示例。

```xml
<div data-sly-use.info="org.example.app.components.info.Info">
  <h1>${info.title}</h1>
  <p>${info.description}</p>
</div>
```
