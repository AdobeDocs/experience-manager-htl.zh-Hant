---
title: HTL區塊陳述式
seo-title: HTL區塊陳述式
description: HTML範本語言(HTL)區塊陳述式是直接新增至現有HTML的自訂資料屬性。
seo-description: HTML範本語言(HTL)區塊陳述式是直接新增至現有HTML的自訂資料屬性。
uuid: 0624fb6e-6989-431b-adic-1138325393f
contentOwner: 使用者
products: SG_ PERIENCENCENAGER/HTL
topic-tags: html-template-language
content-type: 引用
discoiquuid: 58aa6ea8-1d45-4f6f-a77 e-4819f593 a19 d
mwpw-migration-script-version: 2017-10-12T214658.665-0400
translation-type: tm+mt
source-git-commit: 7a94b0b010461b29d2b74c9c717e3b218d0ca5a8

---


# HTL區塊陳述式 {#htl-block-statements}

HTML範本語言(HTL)區塊陳述式是直接新增至現有HTML的自訂 `data` 屬性。這可讓靜態HTML網頁原型簡單而不顯眼地加上註解，將它轉換為功能強大的動態範本，而不會破壞HTML程式碼的有效性。

## sly元素 {#sly-element}

& **amp；t；smy& amp；gt；元素** 不會顯示在產生的HTML中，而是可以使用而非資料自動解除包裝。與amp的目標；t；smy& amp；gt；元素是為了更明顯地顯示元素。如果您希望您仍能使用資料自動解除包裝。

```xml
<sly data-sly-test.varone="${properties.yourProp}"/>
```

與資料自動解除包裝一樣，請盡量盡量減少這個使用量。

## use {#use}

**`data-sly-use`**：初始化輔助物件(以JavaScript或Java定義)並透過變數呈現它。

初始化JavaScript物件，來源檔案位於與範本相同的目錄中。請注意，必須使用檔案名稱：

```xml
<div data-sly-use.nav="navigation.js">${nav.foo}</div>
```

初始化Java類別，來源檔案位於與範本相同的目錄中。請注意，必須使用類別名稱，而不是檔案名稱：

```xml
        <div data-sly-use.nav="Navigation">${nav.foo}</div>
```

初始化Java類別，此類別會在OSGi套裝中安裝該類別。請注意，必須使用其完全合格的類別名稱：

```xml
        <div data-sly-use.nav="org.example.Navigation">${nav.foo}</div>
```

參數可使用 *選項傳遞至初始化*。一般而言，此功能只能用於 `data-sly-template` 位於區塊內的HTL程式碼：

```xml
<div data-sly-use.nav="${'navigation.js' @parentPage=currentPage}">${nav.foo}</div>
```

初始化另一個HTL範本，然後再使用 **`data-sly-call`**：

```xml
<div data-sly-use.nav="navTemplate.html" data-sly-call="${nav.foo}"></div>
```

>[!NOTE]
>
>如需使用API的詳細資訊，請參閱：
>
>* [Java Use-API](use-api-java.md)
>* [JavaScript使用API](use-api-javascript.md)
>



## 解除包裝 {#unwrap}

**`data-sly-unwrap`**：從產生的標記中移除主機元素，同時保留其內容。這允許排除HTL簡報邏輯中所需的元素，但實際輸出不需要。

不過，應謹慎使用此陳述式。一般而言，請盡量將HTL標記保持最接近預期輸出標記。換言之，新增HTL區塊陳述式時，盡量盡量為現有的HTML加上註解，而不需加入新元素。

例如，這個

```xml
<p data-sly-use.nav="navigation.js">Hello World</p>
```

將會產生

```xml
<p>Hello World</p>
```

但是，

```xml
<p data-sly-use.nav="navigation.js" data-sly-unwrap>Hello World</p>
```

將會產生

```xml
Hello World
```

您也可以有條件地解除包裝元素：

```xml
<div class="popup" data-sly-unwrap="${isPopup}">content</div>
```

## 文字 {#text}

**`data-sly-text`**：以指定文字取代其主機元素的內容。

例如，這個

```xml
<p>${properties.jcr:description}</p>
```

等同於

```xml
<p data-sly-text="${properties.jcr:description}">Lorem ipsum</p>
```

兩者都會顯示段落 **`jcr:description`** 文字的值。第二種方式就是允許HTML不顯眼的註解，並保留原始設計人員的靜態預留位置內容。

## attribute {#attribute}

**data-sly-attribute**：新增屬性至主機元素。

例如，這個

```xml
<div title="${properties.jcr:title}"></div>
```

等同於

```xml
<div title="Lorem Ipsum" data-sly-attribute.title="${properties.jcr:title}"></div>
```

兩者都會將 `title` 屬性設定 **`jcr:title`**為值。第二種方式就是允許HTML不顯眼的註解，並保留原始設計人員的靜態預留位置內容。

屬性會由左至右解析，其中屬性的大部分實例(常值或定義透過) **`data-sly-attribute`**優先權凌駕於定義為其左側之相同屬性(或透過 **`data-sly-attribute`**)的任何例項上。

請注意，值( **`literal`** 或透過 **`data-sly-attribute`**值設定) *會在* 最終標記中被移除。此規則的唯一例外是 *保留**常值* 空白字串的常值屬性。例如，

```xml
<div class="${''}" data-sly-attribute.id="${''}"></div>
```

製作、

```xml
<div></div>
```

但是，

```xml
<div class="" data-sly-attribute.id=""></div>
```

製作、

```xml
<div class=""></div>
```

若要設定多個屬性，請傳遞對應物件的索引鍵值配對，對應屬性及其值。例如，假設

```xml
attrMap = {
    title: "myTitle",
    class: "myClass",
    id: "myId"
}
```

然後,

```xml
<div data-sly-attribute="${attrMap}"></div>
```

製作、

```xml
<div title="myTitle" class="myClass" id="myId"></div>
```

## 元素 {#element}

**`data-sly-element`**：取代主機元素的元素名稱。

例如，

```xml
<h1 data-sly-element="${titleLevel}">text</h1>
```

取代 **`h1`****`titleLevel`**值。

基於安全理由， `data-sly-element` 只接受下列元素名稱：

```xml
a abbr address article aside b bdi bdo blockquote br caption cite code col colgroup
data dd del dfn div dl dt em figcaption figure footer h1 h2 h3 h4 h5 h6 header i ins
kbd li main mark nav ol p pre q rp rt ruby s samp section small span strong sub 
sup table tbody td tfoot th thead time tr u var wbr
```

若要設定其他元素，必須關閉XSS安全性( `@context='unsafe'`)。

## 測試 {#test}

**`data-sly-test`：** 有條件地移除主機元素及其內容。值會 `false` 移除元素；值會 `true` 保留元素。

例如 `p` ，元素及其內容只有在下列情況下才會呈現 `isShown``true`：

```xml
<p data-sly-test="${isShown}">text</p>
```

測試結果可以指派給稍後可使用的變數。這通常用於建構「if其他」邏輯，因為沒有明確的其他陳述式：

```xml
<p data-sly-test.abc="${a || b || c}">is true</p>
<p data-sly-test="${!abc}">or not</p>
```

變數一經設定，就有HTL檔案內的全域範圍。

以下是比較值的一些範例：

```xml
<div data-sly-test="${properties.jcr:title == 'test'}">TEST</div>
<div data-sly-test="${properties.jcr:title != 'test'}">NOT TEST</div>

<div data-sly-test="${properties['jcr:title'].length > 3}">Title is longer than 3</div>
<div data-sly-test="${properties['jcr:title'].length >= 0}">Title is longer or equal to zero </div> 

<div data-sly-test="${properties['jcr:title'].length > aemComponent.MAX_LENGTH}">
    Title is longer than the limit of ${aemComponent.MAX_LENGTH}
</div>
```

## repreate {#repeat}

有了data-sly-reprecyces，您可以根據指定的清單 *重復* 重復元素多次。

```xml
<div data-sly-repeat="${currentPage.listChildren}">${item.name}</div>
```

這與資料-sly清單相同，只不過您不需要容器元素。

下列範例顯示您也可以參考屬性的 *項目* ：

```xml
<div data-sly-repeat="${currentPage.listChildren}" data-sly-attribute.class="${item.name}">${item.name}</div>
```

## list {#list}

**`data-sly-list`**：重復提供物件中每個可列舉屬性的主機元素內容。

以下是簡單的回圈：

```xml
<dl data-sly-list="${currentPage.listChildren}">
    <dt>index: ${itemList.index}</dt>
    <dd>value: ${item.title}</dd>
</dl>
```

下列預設變數可在清單的範圍內使用：

**`item`**：重復的目前項目。

**`itemList`**：包含下列屬性的物件：

**`index`**：零型計數器( `0..length-1`)。

**`count`**：一個計數器( `1..length`)。

`first`： `true` 如果目前項目是第一個項目。

**`middle`**： `true` 如果目前項目不是第一個項目或最後一個項目。

**`last`**： `true` 如果目前項目是最後一個項目。

**`odd`**： `true` If `index` is nunder.

**`even`**： `true``index` 如果是。

在 `data-sly-list` 陳述式上定義識別碼可讓您重新命名 **`itemList`** 和 `item` 變數。**`item`** 將變為*** `<variable>`*** **`itemList`****`*<variable>*List`**，將成為您的最佳選擇。

```xml
<dl data-sly-list.child="${currentPage.listChildren}">
    <dt>index: ${childList.index}</dt>
    <dd>value: ${child.title}</dd>
</dl>
```

您也可以動態存取屬性：

```xml
<dl data-sly-list.child="${myObj}">
    <dt>key: ${child}</dt>
    <dd>value: ${myObj[child]}</dd>
</dl>
```

## 資源 {#resource}

**`data-sly-resource`**：包含透過傾斜解析度和演算程序呈現指定資源的結果。

簡單的資源包括：

```xml
<article data-sly-resource="path/to/resource"></article>
```

選項允許多種其他變體：

控制資源路徑：

```xml
<article data-sly-resource="${ @ path='path/to/resource'}"></article>
<article data-sly-resource="${'resource' @ prependPath='my/path'}"></article>
<article data-sly-resource="${'my/path' @ appendPath='resource'}"></article>
```

新增(或取代)選擇器：

```xml
<article data-sly-resource="${'path/to/resource' @ selectors='selector'}"></article>
```

新增、取代或移除多個選擇器：

```xml
<article data-sly-resource="${'path/to/resource' @ selectors=['s1', 's2']}"></article>
```

將選擇器新增至現有的選擇器：

```xml
<article data-sly-resource="${'path/to/resource' @ addSelectors='selector'}"></article>
```

移除現有的選擇器：

```xml
<article data-sly-resource="${'path/to/resource' @ removeSelectors='selector1'}"></article>
```

移除所有選擇器：

```xml
<article data-sly-resource="${'path/to/resource' @ removeSelectors}"></article>
```

覆寫資源的資源類型：

```xml
<article data-sly-resource="${'path/to/resource' @ resourceType='my/resource/type'}"></article>
```

變更WCM模式：

```xml
<article data-sly-resource="${'path/to/resource' @ wcmmode='disabled'}"></article>
```

根據預設，AEM裝飾標記會停用，而designationTagName選項則允許將它們傳回，而CsClassName會將類別新增至該元素。

```xml
<article data-sly-resource="${'path/to/resource' @ decorationTagName='span',
cssClassName='className'}"></article>
```

>[!NOTE]
>
>AEM提供清楚簡單的邏輯，可控制包含元素的裝飾標籤。如需詳細資訊，請參閱 [開發元件說明文件中的裝飾標籤](https://helpx.adobe.com/experience-manager/6-4/sites/developing/using/decoration-tag.html) 。

## include {#include}

**`data-sly-include`**：將主機元素的內容取代為由指定HTML範本檔案(HTL、JSP、ESP等)產生的標記。的值。*隨附檔案* 的演算內容不會包含目前的HTL上下文( *包括檔案)*；因此，若要納入HTL檔案，目前 **`data-sly-use`** 在包含的檔案中必須重復此項目(在這種情況下，通常較適合 **`data-sly-template`** 使用和 `data-sly-call`)

簡單包括：

```xml
<section data-sly-include="path/to/template.html"></section>
```

JSP也可以包含相同的方式：

```xml
<section data-sly-include="path/to/template.jsp"></section>
```

選項可讓您控制檔案路徑：

```xml
<section data-sly-include="${ @ file='path/to/template.html'}"></section>
<section data-sly-include="${'template.html' @ prependPath='my/path'}"></section>
<section data-sly-include="${'my/path' @ appendPath='template.html'}"></section>
```

您也可以變更WCM模式：

```xml
<section data-sly-include="${'template.html' @ wcmmode='disabled'}"></section>
```

## 範本與呼叫 {#template-call}

`data-sly-template`：定義範本。主機元素及其內容不會由HTL輸出

`data-sly-call`：呼叫使用資料ly範本定義的範本。名為範本的內容(選擇性的參數化)會取代呼叫的主機元素內容。

定義靜態範本，然後呼叫它：

```xml
<template data-sly-template.one>blah</template>
<div data-sly-call="${one}"></div>
```

定義動態範本，然後使用參數呼叫它：

```xml
<template data-sly-template.two="${ @ title}"><h1>${title}</h1></template>
<div data-sly-call="${two @ title=properties.jcr:title}"></div>
```

位於不同檔案中的範本可以初始化 `data-sly-use`。請注意，在此情況 `data-sly-use` 下， `data-sly-call` 也可以放置在同一個元素上：

```xml
<div data-sly-use.lib="templateLib.html">
    <div data-sly-call="${lib.one}"></div>
    <div data-sly-call="${lib.two @ title=properties.jcr:title}"></div>
</div>
```

支援範本循環：

```xml
<template data-sly-template.nav="${ @ page}">
    <ul data-sly-list="${page.listChildren}">
        <li>
            <div class="title">${item.title}</div>
            <div data-sly-call="${nav @ page=item}" data-sly-unwrap></div>
        </li>
    </ul>
</template>
<div data-sly-call="${nav @ page=currentPage}" data-sly-unwrap></div>
```

## i18n和地區設定物件 {#i-n-and-locale-objects}

當您使用i18n和HTL時，現在也可以傳入自訂地區設定物件。

```xml
${'Hello World' @ i18n, locale=request.locale}
```

## URL操作 {#url-manipulation}

可使用新的url控制項。

請參閱下列的使用範例：

新增html延伸模組至路徑。

```xml
<a href="${item.path @ extension = 'html'}">${item.name}</a>
```

將html擴充功能和選擇器加入路徑中。

```xml
<a href="${item.path @ extension = 'html', selectors='products'}">${item.name}</a>
```

將html擴充功能和片段(# value)新增至路徑。

```xml
<a href="${item.path @ extension = 'html', fragment=item.name}">${item.name}</a>
```

## AEM6.3支援的HTL功能 {#htl-features-supported-in-aem}

Adobe Experience Manager(AEM)6.3支援下列新的HTL功能：

### 數字/日期格式 {#number-date-formatting}

AEM6.3支援原生格式的數字和日期格式，毋需編寫自訂程式碼。這也支援時區和地區設定。

下列範例顯示先指定格式，然後使用需要格式化的值：

```xml
<h2>${ 'dd-MMMM-yyyy hh:mm:ss' @
           format=currentPage.lastModified,
           timezone='PST',
           locale='fr'}</h2>

<h2>${ '#.00' @ format=300}</h2>
```

>[!NOTE]
>
>有關您可使用之格式的完整詳細資訊，請參閱 [HTL規格](https://github.com/Adobe-Marketing-Cloud/htl-spec/blob/master/SPECIFICATION.md)。

### 資料與資源的緊密運用 {#data-sly-use-with-resources}

這可讓HTL直接在HTL中取得資料，毋需編寫程式碼才能取得資源。

例如：

```xml
<div data-sly-use.product=“/etc/commerce/product/12345”>
  ${ product.title }
</div>
```

### 要求屬性 {#request-attributes}

在 *data-sly-include* 和 *data-sly-resources中* ，您現在可以傳遞 *requestAttributes* ，以便在接收HTL指令碼中使用它們。

這可讓您將參數正確傳入指令碼或元件中。

```xml
<sly data-sly-use.settings="com.adobe.examples.htl.core.hashmap.Settings" 
        data-sly-include="${ 'productdetails.html' @ requestAttributes=settings.settings}" />
```

「設定」類別的Java程式碼，會使用地圖來傳遞RequestAttributes：

```xml
public class Settings extends WCMUsePojo {

  // used to pass is requestAttributes to data-sly-resource
  public Map<String, Object> settings = new HashMap<String, Object>();

  @Override
  public void activate() throws Exception {
    settings.put("layout", "flex");
  }
}
```

例如，透過Sling-Model，您可以使用指定的requestAttributes值。

在此範例中， *會* 從「使用」類別透過「地圖」插入版面：

```xml
@Model(adaptables=SlingHttpServletRequest.class)
public class ProductSettings {
  @Inject @Optional @Default(values="empty")
  public String layout;

}
```

### 修正@ extension {#fix-for-extension}

@ extension適用於AEM6.3中的所有藍本，之前您可能會像 *www.adobe.com.html* 一樣，也會檢查是否新增或未新增副檔名。

```xml
${ link @ extension = 'html' }
```
