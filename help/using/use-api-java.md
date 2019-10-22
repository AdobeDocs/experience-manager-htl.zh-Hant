---
title: HTL Java Use-API
seo-title: HTL Java Use-API
description: 'HTML 範本語言 - HTL - Java Use-API 可讓 HTL 檔案存取自訂 Java 類別中的 Helper 方法。 '
seo-description: 'HTML 範本語言 - HTL - Java Use-API 可讓 HTL 檔案存取自訂 Java 類別中的 Helper 方法。 '
uuid: b340f8f7-a193-45c8-aa39-5c6e2c0194ea
contentOwner: 使用者
products: SG_EXPERIENCEMANAGER/HTL
topic-tags: html-template-language
content-type: 引用
discoiquuid: 126ebc9d-5f7b-47a4-aea2-c8840d34864c
mwpw-migration-script-version: 2017-10-12T21 46 58.665-0400
translation-type: tm+mt
source-git-commit: 6de5ed20e4463c0c2e804e24cb853336229a7c1f

---


# HTL Java Use-API{#htl-java-use-api}

HTML範本語言(HTL)Java Use-API可讓HTL檔案存取自訂Java類別中的輔助方法。 這可讓所有複雜的商業邏輯封裝在Java程式碼中，而HTL程式碼則僅處理直接標籤製作。

## 簡單範例 {#a-simple-example}

我們將從沒有use類 *別的* HTL元件開始。 它由一個檔案組成， `/apps/my-example/components/info.html`

### `/apps/my-example/component/info/info.html` {#apps-my-example-component-info-info-html}

```xml
<div>
    <h1>${properties.title}</h1>
    <p>${properties.description}</p>
</div>
```

我們也會為此元件新增一些內容，以便在以下網址進行演算 **`/content/my-example/`**:

### `http://localhost:4502/content/my-example.json` {#http-localhost-content-my-example-json}

```java
{
    "sling:resourceType": "my-example/component/info",
    "title": "My Example",
    "description": "This Is Some Example Content."
}
```

存取此內容時，會執行HTL檔案。 在HTL程式碼中，我們使用內容物 **`properties`** 件來存取目前資源並 `title` 顯 `description` 示它們。 輸出HTML將為：

### `view-source:http://localhost:4502/content/my-example.html` {#view-source-http-localhost-content-my-example-html}

```xml
<div>
    <h1>My Example</h1>
    <p>This Is Some Example Content.</p>
</div>
```

### 添加使用類 {#adding-a-use-class}

info **元件** （現狀）不需要use-class來執行其（非常簡單）功能。 不過，在某些情況下，您需要執行無法在HTL中完成的工作，因此您需要使用類別。 但請記住：

>[!NOTE]
>
>*僅當無法在HTL中單獨執行某項作業時，才應使用use-class。*

例如，假設您希望元件 `info` 顯示資源的 `title` 和 **`description`** 屬性，但全部以小寫表示。 由於HTL沒有用於小寫字串的方法，因此您需要use-class。 我們可以新增Java use-class並依下列方式變更，以 **`info.html`** 完成此作業：

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

### 本機與搭售Java類 {#local-vs-bundle-java-class}

Java use-class可以用兩種方式安裝：本 **機** 或 **搭售**。 *此示例使用本地安裝。*

在本機安裝中，Java來源檔案會放在HTL檔案旁，位於相同的儲存庫檔案夾中。 系統會根據需求自動編譯來源。 不需要個別編譯或封裝步驟。

在套件安裝中，Java類別必須使用標準AEM套件部署機制，在OSGi套件中編譯並部署(請參 [閱套件Java類別](#bundled-java-class))。

>[!NOTE]
>
>當 **use-class是相關元件專屬時** ，建議使用本機Java use-class。
>
>當 **Java程式碼實作從多個HTL元件存取的服務時** ，建議使用套裝Java use-class。

### Java包是儲存庫路徑 {#java-package-is-repository-path}

使用本地安裝時，use-class的軟體包名稱必須與儲存庫資料夾位置的軟體包名稱匹配，路徑中的任何連字元都由軟體包名稱中的下划線替換。

在本例中， **`Info.java`** 位於的 **`/apps/my-example/components/info`** 位置是，因此包 **`apps.my_example.components.info`** :

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
>在AEM開發中，建議在儲存庫項目名稱中使用連字型大小。 但是，連字型大小在Java包名稱中是非法的。 因此，儲存庫 **路徑中的所有連字元都必須轉換為軟體包名稱中的下划線**。

### 擴充功 `WCMUsePojo` 能 {#extending-wcmusepojo}

雖然有許多將Java類別與HTL結合的方式(請參閱替代 `WCMUsePojo`)，但最簡單的方式是擴充類 `WCMUsePojo` 別：

#### `/apps/my-example/component/info/Info.java` {#apps-my-example-component-info-info-java-2}

```java
package apps.my_example.components.info;
 
import com.adobe.cq.sightly.WCMUsePojo;
 
public class Info extends WCMUsePojo {
    
    ...
}
```

### 初始化類 {#initializing-the-class}

從擴展use-class時，通過 **`WCMUsePojo`**&#x200B;覆蓋方法來執行初 **`activate`** 始化：

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

通常， [activate](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html) 方法會用來根據目前的上下文（例如目前的請求和資源），預先計算並儲存（在成員變數中）HTL程式碼中所需的值。

類 `WCMUsePojo` 別可讓您存取HTL檔案中可用的相同上下文物件集(請參閱 [全域物件](global-objects.md))。

在擴充的類中， **`WCMUsePojo`**** 可使用

[`<T> T get(String name, Class<T> type)`](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html)

或者，常用的上下文物件可透過適當的方便方法 **直接存取**:

|  |  |
|---|---|
| [PageManager](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/day/cq/wcm/api/PageManager.html) | [getPageManager()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getPageManager()) |
| [頁面](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/day/cq/wcm/api/Page.html) | [getCurrentPage()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getCurrentPage()) |
| [頁面](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/day/cq/wcm/api/Page.html) | [getResourcePage()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getResourcePage()) |
| [ValueMap](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/resource/ValueMap.html) | [getPageProperties()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getPageProperties()) |
| [ValueMap](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/resource/ValueMap.html) | [getProperties()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getProperties()) |
| [設計人員](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/day/cq/wcm/api/designer/Designer.html) | [getDesigner()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getDesigner()) |
| [設計](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/day/cq/wcm/api/designer/Design.html) | [getCurrentDesign()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getCurrentDesign()) |
| [樣式](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/day/cq/wcm/api/designer/Style.html) | [getCurrentStyle()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getCurrentStyle()) |
| [元件](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/day/cq/wcm/api/components/Component.html) | [getComponent()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getComponent()) |
| [ValueMap](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/resource/ValueMap.html) | [getInheritedProperties()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse#getInheritedProperties.html()) |
| [資源](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/resource/Resource.html) | [getResource()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getResource()) |
| [資源解析器](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/resource/ResourceResolver.html) | [getResourceResolver()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getResourceResolver()) |
| [SlingHttpServletRequest](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/SlingHttpServletRequest.html) | [getRequest()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getRequest()) |
| [SlingHttpServletResponse](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/SlingHttpServletResponse.html) | [getResponse()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getResponse()) |
| [SlingScriptHelper](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/scripting/SlingScriptHelper.html) | [getSlingScriptHelper()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getSlingScriptHelper()) |

### Getter方法 {#getter-methods}

在use-class初始化後，就會執行HTL檔案。 在此階段中，HTL通常會拉入use-class的各個成員變數的狀態，並將它們轉譯為簡報。

若要從HTL檔案中存取這些值，您必鬚根據下列命名慣例，在use-class中定 **義自訂getter方法**:

* 表單的方法 **`getXyz`** 將在HTL檔案中公開名為xyz的物件屬 ***性***。

例如，在下例中，這些方 **`getTitle`** 法 **`getDescription`** 和結果導致對象屬 **`title`** 性，並可 **`description`** 在HTL檔案的上下文中訪問：

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

### data-slyuse屬性 {#data-sly-use-attribute}

此 **`data-sly-use`** 屬性用於初始化HTL程式碼中的use-class。 在我們的示例中， `data-sly-use` 屬性聲明我們要使用類 **`Info`**。 我們只能使用類的本地名稱，因為我們使用的是本地安裝（將Java源檔案放在與HTL檔案相同的資料夾中）。 如果我們使用的是套件安裝，我們必須指定完全限定的類別名稱(請參 [閱使用類別套裝安裝](#LocalvsBundleJavaClass))。

### `/apps/my-example/component/info/info.html` {#apps-my-example-component-info-info-html-2}

```xml
<div data-sly-use.info="Info">
    <h1>${info.lowerCaseTitle}</h1>
    <p>${info.lowerCaseDescription}</p>
</div>
```

### 本機識別碼 {#local-identifier}

識別碼'**`info`**'(在中的點之後 **`data-sly-use.info`**)會用於HTL檔案中，以識別類別。 此標識符的範圍在檔案中是全局的，在聲明後。 它不限於包含語句的元 `data-sly-use` 素。

### `/apps/my-example/component/info/info.html`{#apps-my-example-component-info-info-html-3}

```xml
<div data-sly-use.info="Info">
    <h1>${info.lowerCaseTitle}</h1>
    <p>${info.lowerCaseDescription}</p>
</div>
```

### 取得屬性 {#getting-properties}

然後，該 `info` 標識符用於訪問通過getter方 **`title`** 法 **`description`** 和公開的對象屬性 `Info.getTitle` 和 **`Info.getDescription`**。

### `/apps/my-example/component/info/info.html` {#apps-my-example-component-info-info-html-4}

```xml
<div data-sly-use.info="Info">
    <h1>${info.lowerCaseTitle}</h1>
    <p>${info.lowerCaseDescription}</p>
</div>
```

### 輸出 {#output}

現在，當我們存取時， **`/content/my-example.html`** 它會傳回下列HTML:

### `view-source:http://localhost:4502/content/my-example.html` {#view-source-http-localhost-content-my-example-html-1}

```xml
<div>
    <h1>my example</h1>
    <p>this is some example content.</p>
</div>
```

## 超越基本概念 {#beyond-the-basics}

在本節中，我們將介紹一些超越上述簡單示例的其他功能：

* 將參數傳遞至use-class。
* 搭售的Java使用類別。
* 替代項目 `WCMUsePojo`

### 傳遞參數 {#passing-parameters}

參數可在初始化時傳遞至use-class。 例如，我們可以做這樣的事：

### `/content/my-example/component/info/info.html` {#content-my-example-component-info-info-html}

```xml
<div data-sly-use.info="${'Info' @ text='Some text'}">
    <h1>${info.lowerCaseTitle}</h1>
    <p>${info.lowerCaseDescription}</p>
    <p>${info.upperCaseText}</p>
</div>
```

這裡我們傳遞的參數稱為 **`text`**。 use-class接著將我們擷取的字串加上上，並顯示結果與 `info.upperCaseText`。 以下是已調整的使用類別：

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

參數是透過方法存 `WCMUsePojo` 取

[ `<T> T get(String paramName, Class<T> type)`](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html)

就我們而言，這份聲明

`get("text", String.class)`

然後，該字串被反轉並通過該方法公開

**`getReverseText()`**

### 僅從資料密碼模板傳遞參數 {#only-pass-parameters-from-data-sly-template}

雖然上述範例在技術上是正確的，但是當相關值在HTL程式碼的執行內容中可用時，從HTL傳遞值來初始化使用類別實際上並不有意義（或者，實際上，如上所述，該值是靜態的）。

原因是use-class將永遠擁有與HTL程式碼相同執行內容的存取權。 這就引出了最佳實踐的匯入點：

>[!NOTE]
>
>只有當use-class用於 **data-sly-template** file中時，才應將參數傳遞至use-class，而此檔案本身是從另一個HTL檔案呼叫，且參數需要傳遞。

例如，讓我們在現有範例旁 `data-sly-template` 建立個別的檔案。 我們將調用新檔案 `extra.html`。 它包含一個 **`data-sly-template`** 名為 **`extra`**:

### `/apps/my-example/component/info/extra.html` {#apps-my-example-component-info-extra-html}

```xml
<template data-sly-template.extra="${@ text}"
          data-sly-use.extraHelper="${'ExtraHelper' @ text=text}">
  <p>${extraHelper.reversedText}</p>
</template>
```

範本 **`extra`**&#x200B;會取用單一參數 **`text`**。 然後，它以本地名 `ExtraHelper` 稱初始化Java use-class **`extraHelper`** ，並將模板參數的值 **`text`** 作為use-class參數傳遞給它 **`text`**。

範本的內文會取得屬性( `extraHelper.reversedText` 在外罩下方實際呼叫)並顯 `ExtraHelper.getReversedText()`示該值。

我們也會調整現有 **`info.html`** 的範本，以使用此新範本：

### `/apps/my-example/component/info/info.html` {#apps-my-example-component-info-info-html-5}

```xml
<div data-sly-use.info="Info" 
     data-sly-use.extra="extra.html">
    
  <h1>${info.lowerCaseTitle}</h1>
  <p>${info.lowerCaseDescription}</p>
    
  <div data-sly-call="${extra.extra @ text=properties.description}"></div>

</div>
```

檔案現 `info.html` 在包含兩 **`data-sly-use`** 個陳述式，其中原始陳述式會匯入 **`Info`** Java use-class，新陳述式會以本機名稱匯入範本檔案 `extra`。

請注意，我們可以將範本區塊放在檔案中，以避 **`info.html`** 免第二個範本區塊 **`data-sly-use`**，但是個別範本檔案比較常見，而且可重複使用。

類 **`Info`** 和以前一樣使用，調用其getter方法 **`getLowerCaseTitle()`** , `getLowerCaseDescription()` 並通過其相應的HTL屬性 `info.lowerCaseTitle` 和 **`info.lowerCaseDescription`**。

然後，我們對模 **`data-sly-call`** 板執行 **`extra`** ，並將值作為 `properties.description` 參數傳遞 **`text`**。

Java use-class已變 `Info.java` 更，可處理新文字參數：

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

參 **`text`** 數會擷取， **`get("text", String.class)`**&#x200B;值會反轉，並透過getter `reversedText` 做為HTL物件使用 `getReversedText()`。

### 捆綁的Java類 {#bundled-java-class}

使用Bundle use-class，必須使用標準OSGi Bundle部署機制，在AEM中編譯、封裝和部署類別。 與本地安裝不同， use-class包聲明 **應正常命** 名：

### `/apps/my-example/component/info/Info.java` {#apps-my-example-component-info-info-java-6}

```java
package org.example.app.components;
 
import com.adobe.cq.sightly.WCMUsePojo;
 
public class Info extends WCMUsePojo {
    ...
}
```

而且，語 `data-sly-use` 句必須引用完 *全限定的類名*，而不是僅引用本地類名：

### `/apps/my-example/component/info/info.html` {#apps-my-example-component-info-info-html-6}

```xml
<div data-sly-use.info="org.example.app.components.info.Info">
  <h1>${info.title}</h1>
  <p>${info.description}</p>
</div>
```

### 替代方 `WCMUsePojo` 案 {#alternatives-to-wcmusepojo}

建立Java使用類別最常見的方式是擴充 `WCMUsePojo`。 但是，還有許多其他選擇。 要瞭解這些變體，瞭解HTL陳述式在外罩 `data-sly-use` 下的運作方式會有所幫助。

假設您有下列陳 `data-sly-use` 述式：

**`<div data-sly-use.`** `localName`**`="`** `UseClass`**`">`**

系統將按以下方式處理語句：

(1)

* 如果與HTL檔案位於同一 *目錄中存在本地檔案UseClass.java* ，請嘗試編譯並載入該類。 如果成功轉到(2)。

* 否則，請 *將UseClass* 解譯為完全限定的類名，並嘗試從OSGi環境載入它。 如果成功轉到(2)。
* 否則，請 *將UseClass* 解譯為HTL或JavaScript檔案的路徑，並載入該檔案。 如果goto(4)成功，

(2)

* 請嘗試將目前版本 **`Resource`** 調整為 *`UseClass.`* 如果成功，請轉至(3)。

* 否則，請嘗試將電流調 **`Request`** 整為 *`UseClass`*。 如果成功，請轉至(3)。

* 否則，請嘗試使用零 *`UseClass`* 引數建構函式實例化。 如果成功，請轉至(3)。

(3)

* 在HTL中，將新調整或建立的物件系結至名稱 *`localName`*。
* 如果 *`UseClass`* 實 [ 現，則調用 `io.sightly.java.api.Use`](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/io/sightly/java/api/Use.html) 方法，傳遞當前執行上下文(以對象的 `init``javax.scripting.Bindings` 形式)。

(4)

* 如果 *`UseClass`* 是包含HTL檔案的路徑，請 `data-sly-template`準備範本。

* 否則， *`UseClass`* 如果是JavaScript use-class的路徑，請準備use-class(請參 [閱JavaScript Use-API](use-api-javascript.md))。

上述說明的幾個要點：

* 任何可從中調整、可 `Resource`從中調整或 `Request`具有零引數建構函式的類別都可以是use-class。 類不必擴展甚至 `WCMUsePojo` 實現 `Use`。

* 不過，如果use-class實 *作* ，則其方法會自動與目前的內容 `Use`**`init`** 一起呼叫，讓您將初始化程式碼放置在該內容上。

* 擴充的使用類別 `WCMUsePojo` 只是實作的特例 **`Use`**。 它提供了方便的上下文方法，並 **`activate`** 且自動調用其方法 `Use.init`。

### 直接實作介面使用 {#directly-implement-interface-use}

雖然建立使用類別最常見的方式是擴充， `WCMUsePojo`但您也可以直接實作介 `[io.sightly.java.api.Use](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/io/sightly/java/api/Use.html)`面本身。

該接 `Use` 口僅定義一種方法：

`[public void init(javax.script.Bindings bindings)](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/io/sightly/java/api/Use#init(javax.script.Bindings))`

在 **`init`** 初始化類時，將調用該方法，該類具有 **`Bindings`** 包含所有上下文對象和傳遞到use-class中的任何參數的對象。

所有其他功能(如等同的 `WCMUsePojo.getProperties()`)都必須使用物件明確實 ` [javax.script.Bindings](http://docs.oracle.com/javase/7/docs/api/javax/script/Bindings.html)` 施。 例如：

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

要自行實作介面而 `Use` 不是擴展介面， `WCMUsePojo` 主要的案例是您希望使用現有類的子類作為use-class。

### 從資源調整 {#adaptable-from-resource}

另一個選項是使用可自適應的輔助類 **`org.apache.sling.api.resource.Resource`**。

假設您需要編寫HTL指令碼，以顯示DAM資產的中型。 在本例中，您知道在呼叫HTL指令檔時，該指令檔會位於包含 **`Resource`** nodetype的JCR的 **`Node`** 內容內 **`dam:Asset`**。

您知道節點 **`dam:Asset`** 有這樣的結構：

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

在這裡，我們會顯示AEM預設安裝隨附的資產（JPEG影像），做為範例專案geometrixx的一部分。 資產稱為， **`jane_doe.jpg`** 其mimetype為 **`image/jpeg`**。

若要從HTL中存取資產，您可以在 ` [com.day.cq.dam.api.Asset](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/granite/asset/api/Asset.html)` 陳述式中宣告為類 **`data-sly-use`** 別：然後使用get方法來 **`Asset`** 檢索所需資訊。 例如：

### `mimetype.html` {#mimetype-html}

```xml
<div data-sly-use.asset="com.day.cq.dam.api.Asset">
  <p>${asset.mimeType}</p>
</div>
```

此語 `data-sly-use` 句會指示HTL將目前版本改 **`Resource`** 變為 **`Asset`** 本機名稱，並為其命名 **`asset`**。 然後，它會呼叫 **`getMimeType`** 使用HTL getter `Asset` 速記的方法： `asset.mimeType`。

### 自請求調整 {#adaptable-from-request}

您也可以將任何可從 **` [org.apache.sling.api.SlingHttpServletRequest](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/SlingHttpServletRequest.html)`**

與上述適用於的use類的情況一樣， `Resource`可在語句中指定 [`SlingHttpServletRequest`](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/SlingHttpServletRequest.html) 適用於的use類 `data-sly-use` 。 執行時，目前的要求將會適應給定的類別，而產生的物件將可在HTL中使用。
