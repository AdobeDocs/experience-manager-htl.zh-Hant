---
title: HTL 區塊陳述式
description: HTML範本語言(HTL)區塊陳述式是直接新增至現有HTML的自訂資料屬性。
translation-type: tm+mt
source-git-commit: f7e46aaac2a4b51d7fa131ef46692ba6be58d878
workflow-type: tm+mt
source-wordcount: '1555'
ht-degree: 1%

---


# HTL 區塊陳述式 {#htl-block-statements}

HTML範本語言(HTL)區塊陳述式是自訂`data`屬性，直接新增至現有的HTML。 這可讓原型靜態HTML頁面的註解更簡單且不顯眼，並將它轉換為可運作的動態範本，而不會中斷HTML程式碼的有效性。

## 塊概述{#overview}

HTL區塊外掛程式是由HTML元素上設定的`data-sly-*`屬性所定義。 元素可以有結束標籤或是自行結束。 屬性可以有值（可以是靜態字串或運算式），或只是布林屬性（不含值）。

```xml
<tag data-sly-BLOCK></tag>                                 <!--/* A block is simply consists in a data-sly attribute set on an element. */-->
<tag data-sly-BLOCK/>                                      <!--/* Empty elements (without a closing tag) should have the trailing slash. */-->
<tag data-sly-BLOCK="string value"/>                       <!--/* A block statement usually has a value passed, but not necessarily. */-->
<tag data-sly-BLOCK="${expression}"/>                      <!--/* The passed value can be an expression as well. */-->
<tag data-sly-BLOCK="${@ myArg='foo'}"/>                   <!--/* Or a parametric expression with arguments. */-->
<tag data-sly-BLOCKONE="value" data-sly-BLOCKTWO="value"/> <!--/* Several block statements can be set on a same element. */-->
```

所有評估的`data-sly-*`屬性都從生成的標籤中刪除。

### 標識符{#identifiers}

塊語句後面還可以有標識符：

```xml
<tag data-sly-BLOCK.IDENTIFIER="value"></tag>
```

此標識符可由block語句以各種方式使用，以下是一些示例：

```xml
<!--/* Example of statements that use the identifier to set a variable with their result: */-->
<div data-sly-use.navigation="MyNavigation">${navigation.title}</div>
<div data-sly-test.isEditMode="${wcmmode.edit}">${isEditMode}</div>
<div data-sly-list.child="${currentPage.listChildren}">${child.properties.jcr:title}</div>
<div data-sly-template.nav>Hello World</div>

<!--/* The attribute statement uses the identifier to know to which attribute it should apply it's value: */-->
<div data-sly-attribute.title="${properties.jcr:title}"></div> <!--/* This will create a title attribute */-->
```

頂層識別碼不區分大小寫（因為它們可以透過不區分大小寫的HTML屬性來設定），但是其所有屬性都區分大小寫。

## 可用塊語句{#available-block-statements}

有許多可用的塊語句。 在同一元素上使用時，以下優先順序清單定義了塊語句的評估方式：

1. `data-sly-template`
1. `data-sly-set`,  `data-sly-test`  `data-sly-use`
1. `data-sly-call`
1. `data-sly-text`
1. `data-sly-element`,  `data-sly-include`  `data-sly-resource`
1. `data-sly-unwrap`
1. `data-sly-list`, `data-sly-repeat`
1. `data-sly-attribute`

當兩個塊語句具有相同的優先順序時，其評估順序從左到右。

### 使用{#use}

`data-sly-use` 初始化幫助程式對象（在JavaScript或Java中定義），並通過變數將其公開。

初始化JavaScript物件，其中來源檔案位於與範本相同的目錄中。 請注意，必須使用檔案名稱：

```xml
<div data-sly-use.nav="navigation.js">${nav.foo}</div>
```

初始化Java類，其中源檔案與模板位於同一目錄中。 請注意，必須使用類名，而不是檔案名：

```xml
<div data-sly-use.nav="Navigation">${nav.foo}</div>
```

初始化Java類，該類作為OSGi包的一部分安裝。 請注意，必須使用其全限定類名：

```xml
<div data-sly-use.nav="org.example.Navigation">${nav.foo}</div>
```

參數可以使用選項傳遞至初始化。 一般而言，此功能僅應用於位於`data-sly-template`區塊內的HTL程式碼：

```xml
<div data-sly-use.nav="${'navigation.js' @parentPage=currentPage}">${nav.foo}</div>
```

初始化另一個HTL範本，然後可使用`data-sly-call`呼叫：

```xml
<div data-sly-use.nav="navTemplate.html" data-sly-call="${nav.foo}"></div>
```

>[!NOTE]
>
>如需Use-API的詳細資訊，請參閱：
>
>* [Java Use-API](use-api-java.md)
>* [JavaScript Use-API](use-api-javascript.md)


#### 資源{#data-sly-use-with-resources}的資料密集使用

這可讓使用`data-sly-use`直接在HTL中取得資源，而不需要編寫程式碼即可取得。

例如：

```xml
<div data-sly-use.product=“/etc/commerce/product/12345”>
  ${ product.title }
</div>
```

>[!TIP]
>
>另請參閱[Path not Always Required.](#path-not-required)一節

### 取消換行{#unwrap}

`data-sly-unwrap` 從生成的標籤中移除主機元素，同時保留其內容。這可排除HTL呈現邏輯中需要但實際輸出中不需要的元素。

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

### set {#set}

`data-sly-set` 定義具有預先定義值的新標識符。

```xml
<span data-sly-set.profile="${user.profile}">Hello, ${profile.firstName} ${profile.lastName}!</span>
<a class="profile-link" href="${profile.url}">Edit your profile</a>
```

### 文字 {#text}

`data-sly-text` 將其主機元素的內容替換為指定的文本。

例如，

```xml
<p>${properties.jcr:description}</p>
```

等於

```xml
<p data-sly-text="${properties.jcr:description}">Lorem ipsum</p>
```

兩者都會將`jcr:description`值顯示為段落文字。 第二種方法的優點是，允許HTML的不顯眼註解，同時保留原始設計人員的靜態預留位置內容。

### 屬性{#attribute}

`data-sly-attribute` 將屬性添加到主機元素。

例如，

```xml
<div title="${properties.jcr:title}"></div>
```

等於

```xml
<div title="Lorem Ipsum" data-sly-attribute.title="${properties.jcr:title}"></div>
```

兩者都會將`title`屬性設為`jcr:title`的值。 第二種方法的優點是，允許HTML的不顯眼註解，同時保留原始設計人員的靜態預留位置內容。

屬性由左至右解析，最右側的屬性例項（常值或透過`data-sly-attribute`定義）優先於其左側定義的相同屬性例項（逐字定義或透過`data-sly-attribute`定義）。

請注意，在最終標注中，將刪除其值評估為空字串的屬性（`literal`或通過`data-sly-attribute`設定）。 此規則的一個例外是，將常值屬性設為常值空字串時，將會保留。 例如，

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

### 元素 {#element}

`data-sly-element` 替換主機元素的元素名稱。

例如，

```xml
<h1 data-sly-element="${titleLevel}">text</h1>
```

將`h1`替換為`titleLevel`值。

出於安全原因，`data-sly-element`僅接受以下元素名稱：

```xml
a abbr address article aside b bdi bdo blockquote br caption cite code col colgroup
data dd del dfn div dl dt em figcaption figure footer h1 h2 h3 h4 h5 h6 header i ins
kbd li main mark nav ol p pre q rp rt ruby s samp section small span strong sub
sup table tbody td tfoot th thead time tr u var wbr
```

要設定其他元素，必須關閉XSS安全性(`@context='unsafe'`)。

### 測試 {#test}

`data-sly-test` 有條件地移除主機元素及其內容。值`false`會移除元素；值`true`保留元素。

例如，`p`元素及其內容只有在`isShown`為`true`時才會呈現：

```xml
<p data-sly-test="${isShown}">text</p>
```

測試結果可指派給變數，以供稍後使用。 這通常用於構造&quot;if else&quot;邏輯，因為沒有明確的else語句：

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

### 重複{#repeat}

使用`data-sly-repeat`，您可以根據指定的清單重複多個元素。

```xml
<div data-sly-repeat="${currentPage.listChildren}">${item.name}</div>
```

其運作方式與`data-sly-list`相同，但您不需要容器元素。

以下示例顯示，您也可以參考&#x200B;*item*&#x200B;的屬性：

```xml
<div data-sly-repeat="${currentPage.listChildren}" data-sly-attribute.class="${item.name}">${item.name}</div>
```

### 清單{#list}

`data-sly-list` 為提供的對象中的每個可枚舉屬性重複主機元素的內容。

以下是一個簡單的循環：

```xml
<dl data-sly-list="${currentPage.listChildren}">
    <dt>index: ${itemList.index}</dt>
    <dd>value: ${item.title}</dd>
</dl>
```

下列預設變數可在清單範圍中使用：

* `item`:小版本中的當前項。
* `itemList`:保存以下屬性的對象：
* `index`:零計數器( `0..length-1`)。
* `count`:單一計數器( `1..length`)。
* `first`: `true` 如果當前項目是第一個項目。
* `middle`: `true` 如果當前項目既不是第一個項目，也不是最後一個項目。
* `last`: `true` 如果當前項目是最後一個項目。
* `odd`: `true` 如果 `index` 是奇數。
* `even`: `true` 如 `index` 果扯平。

在`data-sly-list`語句中定義標識符可以更名`itemList`和`item`變數。 `item` 會變 `<variable>` 成 `itemList` 會變 `<variable>List`成。

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

### 資源 {#resource}

`data-sly-resource` 包含透過吊索解析度和轉換程式轉換指示資源的結果。

簡單的資源包括：

```xml
<article data-sly-resource="path/to/resource"></article>
```

#### 路徑不總是必需{#path-not-required}

請注意，如果您已擁有資源，則不需要使用具有`data-sly-resource`的路徑。 如果您已擁有資源，則可直接使用。

例如，以下是正確的。

```xml
<sly data-sly-resource="${resource.path @ decorationTagName='div'}"></sly>
```

但是，以下也完全可以接受。

```xml
<sly data-sly-resource="${resource @ decorationTagName='div'}"></sly>
```

基於以下原因，建議盡可能直接使用資源。

* 如果您已經擁有資源，則使用路徑重新解析是額外的不必要工作。
* 當您已擁有資源時，使用路徑可能會引入非預期的結果，因為Sling資源可以包裝或是合成，而不是在指定路徑上提供。

#### 選項 {#resource-options}

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
>AEM提供清晰簡單的邏輯，可控制包住內含元素的裝飾標籤。 如需詳細資訊，請參閱開發元件檔案中的[裝飾標籤](https://docs.adobe.com/content/help/en/experience-manager-65/developing/components/decoration-tag.html)。

### 加入 {#include}

`data-sly-include` 以指示的HTML範本檔案（HTL、JSP、ESP等）產生的標籤取代主機元素的內容當其對應的範本引擎處理時。 內含檔案的呈現內容將不包含目前的HTL內容（包含檔案的呈現內容）;因此，要包含HTL檔案，必須在包含的檔案中重複當前`data-sly-use`（在這種情況下，通常最好使用`data-sly-template`和`data-sly-call`）

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

### Request-attributes {#request-attributes}

在`data-sly-include`和`data-sly-resource`中，您可以傳遞`requestAttributes`，以便在接收HTL-script中使用它們。

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

例如，透過Sling-Model，您可以使用指定`requestAttributes`的值。

在此範例中，版面會透過Use-class的Map插入：

```xml
@Model(adaptables=SlingHttpServletRequest.class)
public class ProductSettings {
  @Inject @Optional @Default(values="empty")
  public String layout;

}
```

### 範本與呼叫{#template-call}

範本區塊可像函式呼叫一樣使用：在其聲明中，它們可以得到參數，然後在調用參數時可以傳遞這些參數。 它們也允許遞回。

`data-sly-template` 定義範本。HTL不會輸出主機元素及其內容

`data-sly-call` 調用使用資料漏洞模板定義的模板。所呼叫範本的內容（可選地參數化）會取代呼叫的主機元素的內容。

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

位於不同檔案中的範本可以使用`data-sly-use`初始化。 請注意，在此情況下，`data-sly-use`和`data-sly-call`也可以放置在相同的元素上：

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

## Sly元素{#sly-element}

`<sly>` HTML標籤可用來移除目前的元素，只顯示其子項。 其功能與`data-sly-unwrap`區塊元素類似：

```xml
<!--/* This will display only the output of the 'header' resource, without the wrapping <sly> tag */-->
<sly data-sly-resource="./header"></sly>
```

雖然不是有效的HTML 5標籤，但`<sly>`標籤可使用`data-sly-unwrap`在最終輸出中顯示：

```xml
<sly data-sly-unwrap="${false}"></sly> <!--/* outputs: <sly></sly> */-->
```

`<sly>`元素的目標是使元素不輸出更加明顯。 如果您想要，您仍可使用`data-sly-unwrap`。

與`data-sly-unwrap`一樣，請盡量減少此功能的使用。
