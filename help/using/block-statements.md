---
title: HTL Block語句
seo-title: HTL Block語句
description: HTML範本語言(HTL)區塊陳述式是直接新增至現有HTML的自訂資料屬性。
seo-description: 'HTML範本語言(HTL)區塊陳述式是直接新增至現有HTML的自訂資料屬性。 '
uuid: 0624fb6e-6989-431b-abc-1138325393f1
contentOwner: 使用者
products: SG_EXPERIENCEMANAGER/HTL
topic-tags: html-template-language
content-type: 引用
discoiquuid: 58aa6ea8-1d45-4f6f-a77e-4819f593a19d
mwpw-migration-script-version: 2017-10-12T21 46 58.665-0400
translation-type: tm+mt
source-git-commit: afc29cbad83caeb549097da3fc33fd9147f1157a

---


# HTL Block語句 {#htl-block-statements}

HTML範本語言(HTL)區塊陳述式是直接新 `data` 增至現有HTML的自訂屬性。 這可讓原型靜態HTML頁面的註解更簡單且不顯眼，並將它轉換為可運作的動態範本，而不會中斷HTML程式碼的有效性。

## 暗元件 {#sly-element}

&lt;sly **** &gt;元素不會顯示在產生的HTML中，而且可以用來取代資料sly-unwrap。 &lt;sly&gt;元素的目的是使元素不輸出更加明顯。 如果您想要，您仍可使用資料隱藏的解封。

```xml
<sly data-sly-test.varone="${properties.yourProp}"/>
```

如同資料的隱藏式解除，請盡量減少此功能的使用。

## use {#use}

**`data-sly-use`**:初始化輔助對象（在JavaScript或Java中定義），並通過變數將其公開。

初始化JavaScript物件，其中來源檔案位於與範本相同的目錄中。 請注意，必須使用檔案名稱：

```xml
<div data-sly-use.nav="navigation.js">${nav.foo}</div>
```

初始化Java類，其中源檔案與模板位於同一目錄中。 請注意，必須使用類名，而不是使用檔案名：

```xml
        <div data-sly-use.nav="Navigation">${nav.foo}</div>
```

初始化Java類，該類作為OSGi包的一部分安裝。 請注意，必須使用其全限定類名：

```xml
        <div data-sly-use.nav="org.example.Navigation">${nav.foo}</div>
```

參數可以使用選項傳遞到初始 *化*。 通常，此功能僅應用於位於區塊內的HTL程 `data-sly-template` 式碼：

```xml
<div data-sly-use.nav="${'navigation.js' @parentPage=currentPage}">${nav.foo}</div>
```

初始化另一個HTL範本，然後可使用 **`data-sly-call`**:

```xml
<div data-sly-use.nav="navTemplate.html" data-sly-call="${nav.foo}"></div>
```

>[!NOTE]
>
>如需Use-API的詳細資訊，請參閱：
>
>* [Java Use-API](use-api-java.md)
>* [JavaScript Use-API](use-api-javascript.md)
>



## 取消包裝 {#unwrap}

**`data-sly-unwrap`**:從生成的標籤中刪除主機元素，同時保留其內容。 這可排除HTL呈現邏輯中需要但實際輸出中不需要的元素。

不過，應謹慎使用此陳述。 一般而言，最好將HTL標籤盡量靠近預期的輸出標籤。 換言之，新增HTL區塊陳述式時，請盡量試著只註解現有的HTML，而不要引入新元素。

例如，

```xml
<p data-sly-use.nav="navigation.js">Hello World</p>
```

將生產

```xml
<p>Hello World</p>
```

然而，

```xml
<p data-sly-use.nav="navigation.js" data-sly-unwrap>Hello World</p>
```

將生產

```xml
Hello World
```

您也可以有條件地解除包覆元素：

```xml
<div class="popup" data-sly-unwrap="${isPopup}">content</div>
```

## 文字 {#text}

**`data-sly-text`**:以指定的文字取代其主機元素的內容。

例如，

```xml
<p>${properties.jcr:description}</p>
```

等於

```xml
<p data-sly-text="${properties.jcr:description}">Lorem ipsum</p>
```

兩者都會將值顯示為 **`jcr:description`** 段落文字。 第二種方法的優點是，允許HTML的不顯眼註解，同時保留原始設計人員的靜態預留位置內容。

## 屬性 {#attribute}

**data-sly-attribute**:將屬性添加到主機元素。

例如，

```xml
<div title="${properties.jcr:title}"></div>
```

等於

```xml
<div title="Lorem Ipsum" data-sly-attribute.title="${properties.jcr:title}"></div>
```

兩者都會將屬 `title` 性設為的值 **`jcr:title`**。 第二種方法的優點是，允許HTML的不顯眼註解，同時保留原始設計人員的靜態預留位置內容。

屬性會由左至右解析，最右側的屬性例項(常值或定義透過 **`data-sly-attribute`**)優先於左側定義的相同屬性例項(定義字面或透過 **`data-sly-attribute`**)。

請注意，在最終標注 **`literal`** 中，將刪 **`data-sly-attribute`**&#x200B;除其值評估為空 *字串的屬性* （或通過設定）。 此規則的一個例外是，將 *常值* 屬性設為常 *值空字串* 的屬性會保留。 例如，

```xml
<div class="${''}" data-sly-attribute.id="${''}"></div>
```

生產、

```xml
<div></div>
```

但是，

```xml
<div class="" data-sly-attribute.id=""></div>
```

生產、

```xml
<div class=""></div>
```

要設定多個屬性，請傳遞對應於屬性及其值的映射對象保持鍵值對。 例如，假設

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

生產、

```xml
<div title="myTitle" class="myClass" id="myId"></div>
```

## 元素 {#element}

**`data-sly-element`**:替換主機元素的元素名稱。

例如，

```xml
<h1 data-sly-element="${titleLevel}">text</h1>
```

以 **`h1`** 值替換 **`titleLevel`**。

出於安全原因， `data-sly-element` 僅接受以下元素名稱：

```xml
a abbr address article aside b bdi bdo blockquote br caption cite code col colgroup
data dd del dfn div dl dt em figcaption figure footer h1 h2 h3 h4 h5 h6 header i ins
kbd li main mark nav ol p pre q rp rt ruby s samp section small span strong sub 
sup table tbody td tfoot th thead time tr u var wbr
```

若要設定其他元素，XSS安全性必須關閉( `@context='unsafe'`)。

## 測試 {#test}

**`data-sly-test`** :有條件地移除主機元素及其內容。 的值會 `false` 移除元素；值保留 `true` 元素。

例如，元素 `p` 及其內容只會在下列情況下轉 `isShown` 譯 `true`:

```xml
<p data-sly-test="${isShown}">text</p>
```

測試結果可指派給變數，以供稍後使用。 這通常用於構造"if else"邏輯，因為沒有明確的else語句：

```xml
<p data-sly-test.abc="${a || b || c}">is true</p>
<p data-sly-test="${!abc}">or not</p>
```

變數一旦設定，就會在HTL檔案中具有全域範圍。

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

## repeat {#repeat}

透過資料密切重複，您 *可以根據指定的清單* ，重複多個元素。

```xml
<div data-sly-repeat="${currentPage.listChildren}">${item.name}</div>
```

這與資料隱藏清單的運作方式相同，但您不需要容器元素。

以下示例說明您也可以參考 *屬性* :

```xml
<div data-sly-repeat="${currentPage.listChildren}" data-sly-attribute.class="${item.name}">${item.name}</div>
```

## list {#list}

**`data-sly-list`**:對提供的對象中的每個可枚舉屬性重複主機元素的內容。

以下是一個簡單的循環：

```xml
<dl data-sly-list="${currentPage.listChildren}">
    <dt>index: ${itemList.index}</dt>
    <dd>value: ${item.title}</dd>
</dl>
```

下列預設變數可在清單範圍中使用：

**`item`**:小版本中的當前項。

**`itemList`**:保存以下屬性的對象：

**`index`**:零計數器( `0..length-1`)。

**`count`**:單一計數器( `1..length`)。

`first`:如果 `true` 當前項目是第一個項目，

**`middle`**:如果 `true` 當前項目既不是第一個項目，也不是最後一個項目。

**`last`**:如果 `true` 當前項目是最後一個項目，

**`odd`**:如 `true` 果 `index` 奇數。

**`even`**:如 `true` 果 `index` 扯平了。

在語句中定義標 `data-sly-list` 識符可以更名和 **`itemList`** 變 `item` 量。 **`item`** 將變為**** `<variable>`**，並 **`itemList`** 將變為 **`*<variable>*List`**。

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

**`data-sly-resource`**:包含透過吊索解析度和轉換程式轉換指示資源的結果。

簡單的資源包括：

```xml
<article data-sly-resource="path/to/resource"></article>
```

選項允許許多其他變體：

控制資源路徑：

```xml
<article data-sly-resource="${ @ path='path/to/resource'}"></article>
<article data-sly-resource="${'resource' @ prependPath='my/path'}"></article>
<article data-sly-resource="${'my/path' @ appendPath='resource'}"></article>
```

新增（或取代）選擇器：

```xml
<article data-sly-resource="${'path/to/resource' @ selectors='selector'}"></article>
```

添加、替換或刪除多個選擇器：

```xml
<article data-sly-resource="${'path/to/resource' @ selectors=['s1', 's2']}"></article>
```

將選取器新增至現有的選取器：

```xml
<article data-sly-resource="${'path/to/resource' @ addSelectors='selector'}"></article>
```

從現有選擇器中刪除一些選擇器：

```xml
<article data-sly-resource="${'path/to/resource' @ removeSelectors='selector1'}"></article>
```

移除所有選擇器：

```xml
<article data-sly-resource="${'path/to/resource' @ removeSelectors}"></article>
```

覆蓋資源的資源類型：

```xml
<article data-sly-resource="${'path/to/resource' @ resourceType='my/resource/type'}"></article>
```

更改WCM模式：

```xml
<article data-sly-resource="${'path/to/resource' @ wcmmode='disabled'}"></article>
```

依預設，AEM裝飾標籤會停用、centoringTagName選項可讓它們返回，而cssClassName則會將類別新增至該元素。

```xml
<article data-sly-resource="${'path/to/resource' @ decorationTagName='span',
cssClassName='className'}"></article>
```

>[!NOTE]
>
>AEM提供清晰簡單的邏輯，可控制包住內含元素的裝飾標籤。 如需詳細資訊，請 [參閱開發元件檔案中](https://helpx.adobe.com/experience-manager/6-4/sites/developing/using/decoration-tag.html) 「裝飾標籤」。

## include {#include}

**`data-sly-include`**:以指示的HTML範本檔案（HTL、JSP、ESP等）產生的標籤取代主機元素的內容當其對應的範本引擎處理時。 包含檔案的轉 *譯內容* ，將不包含目前的HTL內容(包含檔案的 *內容*);因此，若要包含HTL檔案，必 **`data-sly-use`** 須在包含的檔案中重複目前檔案(在這種情況下，通常最好使用 **`data-sly-template`** 和 `data-sly-call`)

簡單的包括：

```xml
<section data-sly-include="path/to/template.html"></section>
```

JSP的加入方式相同：

```xml
<section data-sly-include="path/to/template.jsp"></section>
```

選項可讓您控制檔案的路徑：

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

`data-sly-template`:定義範本。 HTL不會輸出主機元素及其內容

`data-sly-call`:呼叫以資料透明範本定義的範本。 所呼叫範本的內容（可選地參數化）會取代呼叫的主機元素的內容。

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

位於不同檔案中的範本可以初始化為 `data-sly-use`。 請注意，在此情 `data-sly-use` 況下 `data-sly-call` ，也可以放在相同元素上：

```xml
<div data-sly-use.lib="templateLib.html">
    <div data-sly-call="${lib.one}"></div>
    <div data-sly-call="${lib.two @ title=properties.jcr:title}"></div>
</div>
```

支援範本遞回：

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

## i18n和地區物件 {#i-n-and-locale-objects}

當您使用i18n和HTL時，現在也可以傳入自訂的地區設定物件。

```xml
${'Hello World' @ i18n, locale=request.locale}
```

## URL操縱 {#url-manipulation}

有新的URL操縱集可供使用。

請參閱下列使用範例：

將html副檔名新增至路徑。

```xml
<a href="${item.path @ extension = 'html'}">${item.name}</a>
```

將html副檔名和選擇器新增至路徑。

```xml
<a href="${item.path @ extension = 'html', selectors='products'}">${item.name}</a>
```

將html副檔名和片段(#value)新增至路徑。

```xml
<a href="${item.path @ extension = 'html', fragment=item.name}">${item.name}</a>
```

## AEM 6.3支援的HTL功能 {#htl-features-supported-in-aem}

Adobe Experience Manager(AEM)6.3支援下列新的HTL功能：

### 數字／日期格式 {#number-date-formatting}

AEM 6.3支援數字和日期的原生格式，毋需編寫自訂程式碼。 這也支援時區和地區設定。

下列範例顯示，先指定格式，再指定需要格式的值：

```xml
<h2>${ 'dd-MMMM-yyyy hh:mm:ss' @
           format=currentPage.lastModified,
           timezone='PST',
           locale='fr'}</h2>

<h2>${ '#.00' @ format=300}</h2>
```

>[!NOTE]
>
>有關可使用格式的完整詳細資訊，請參閱 [HTL規格](https://github.com/Adobe-Marketing-Cloud/htl-spec/blob/master/SPECIFICATION.md)。

### 資源的資料密集使用 {#data-sly-use-with-resources}

這可讓HTL直接透過資料密集使用取得資源，而不需編寫程式碼即可取得資源。

例如：

```xml
<div data-sly-use.product=“/etc/commerce/product/12345”>
  ${ product.title }
</div>
```

### 請求屬性 {#request-attributes}

在 *data-scly* -include ** 和data-scly-resources中 *，您現在可以傳遞requestAttributes* ，以便在接收的HTL-script中使用它們。

這可讓您將參數正確傳入指令碼或元件。

```xml
<sly data-sly-use.settings="com.adobe.examples.htl.core.hashmap.Settings" 
        data-sly-include="${ 'productdetails.html' @ requestAttributes=settings.settings}" />
```

Settings類別的Java程式碼，Map會用來傳入requestAttributes:

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

例如，透過Sling-Model，您可以使用指定requestAttributes的值。

在此範例中， *版面配置* 會透過Use-class的Map插入：

```xml
@Model(adaptables=SlingHttpServletRequest.class)
public class ProductSettings {
  @Inject @Optional @Default(values="empty")
  public String layout;

}
```

### @extension的修正 {#fix-for-extension}

@extension可在AEM 6.3的所有藍本中運作，之後您就可以產生如 *www.adobe.com.html* ，並檢查是否要新增擴充功能。

```xml
${ link @ extension = 'html' }
```
