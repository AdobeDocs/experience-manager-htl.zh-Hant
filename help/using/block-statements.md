---
title: HTL 區塊陳述式
description: HTML 範本語言 (HTL) 區塊陳述式是直接新增至現有 HTML 的自訂 data 屬性。
exl-id: a517dcef-ab7a-4d4c-a1a9-2e57aad034f7
source-git-commit: 89b9e89254f341e74f1a5a7b99735d2e69c8a91e
workflow-type: ht
source-wordcount: '1555'
ht-degree: 100%

---

# HTL 區塊陳述式 {#htl-block-statements}

HTML 範本語言 (HTL) 區塊陳述式是直接新增至現有 HTML 的自訂 `data` 屬性。 這樣能夠對原型靜態 HTML 頁面新增簡單且低調的註解，將其轉換為正常的動態範本，而不會破壞 HTML 程式碼的有效性。

## 區塊總覽 {#overview}

HTL 區塊外掛程式是由 HTML 元素上設定的 `data-sly-*` 屬性所定義。 元素可以有閉合標記或是自行閉合。 屬性可以有值 (可以是靜態字串或運算式)，或僅是布林值屬性 (沒有值)。

```xml
<tag data-sly-BLOCK></tag>                                 <!--/* A block is simply consists in a data-sly attribute set on an element. */-->
<tag data-sly-BLOCK/>                                      <!--/* Empty elements (without a closing tag) should have the trailing slash. */-->
<tag data-sly-BLOCK="string value"/>                       <!--/* A block statement usually has a value passed, but not necessarily. */-->
<tag data-sly-BLOCK="${expression}"/>                      <!--/* The passed value can be an expression as well. */-->
<tag data-sly-BLOCK="${@ myArg='foo'}"/>                   <!--/* Or a parametric expression with arguments. */-->
<tag data-sly-BLOCKONE="value" data-sly-BLOCKTWO="value"/> <!--/* Several block statements can be set on a same element. */-->
```

從產生的標記中移除所有評估過的 `data-sly-*` 屬性。

### 識別碼 {#identifiers}

區塊陳述式後面也可以緊接著識別碼：

```xml
<tag data-sly-BLOCK.IDENTIFIER="value"></tag>
```

區塊陳述式可透過各種方式使用此識別碼，以下是幾個範例：

```xml
<!--/* Example of statements that use the identifier to set a variable with their result: */-->
<div data-sly-use.navigation="MyNavigation">${navigation.title}</div>
<div data-sly-test.isEditMode="${wcmmode.edit}">${isEditMode}</div>
<div data-sly-list.child="${currentPage.listChildren}">${child.properties.jcr:title}</div>
<div data-sly-template.nav>Hello World</div>

<!--/* The attribute statement uses the identifier to know to which attribute it should apply it's value: */-->
<div data-sly-attribute.title="${properties.jcr:title}"></div> <!--/* This will create a title attribute */-->
```

最上層識別碼不區分大小寫 (因為這些識別碼可透過不區分大小寫的 HTML 屬性 (Attribute) 進行設定)，但其所有屬性 (Property) 都區分大小寫。

## 可用的區塊陳述式 {#available-block-statements}

有許多可用的區塊陳述式。 在相同元素上使用時，以下優先順序清單會定義如何評估區塊陳述式：

1. `data-sly-template`
1. `data-sly-set`, `data-sly-test`, `data-sly-use`
1. `data-sly-call`
1. `data-sly-text`
1. `data-sly-element`, `data-sly-include`, `data-sly-resource`
1. `data-sly-unwrap`
1. `data-sly-list`, `data-sly-repeat`
1. `data-sly-attribute`

當兩個區塊陳述式擁有相同優先順序時，其評估順序為從左到右。

### use {#use}

`data-sly-use` 會將 helper 物件 (定義在 JavaScript 或 Java 中) 初始化並透過變數將其公開。

初始化 JavaScript 物件，其中原始檔案位在與範本相同的目錄中。 請注意，必須使用檔案名稱：

```xml
<div data-sly-use.nav="navigation.js">${nav.foo}</div>
```

初始化 Java 類別，其中原始檔案位在與範本相同的目錄中。 請注意，必須使用類別名稱，而不是檔案名稱：

```xml
<div data-sly-use.nav="Navigation">${nav.foo}</div>
```

初始化 Java 類別，其中該類別會安裝成 OSGi 套件的一部分。 請注意，必須使用其完整類別名稱：

```xml
<div data-sly-use.nav="org.example.Navigation">${nav.foo}</div>
```

可以使用選項將參數傳遞給初始化作業。 通常這項功能只能由位在 `data-sly-template` 區塊中的 HTL 程式碼使用：

```xml
<div data-sly-use.nav="${'navigation.js' @parentPage=currentPage}">${nav.foo}</div>
```

初始化另一個 HTL 範本，然後可以使用 `data-sly-call` 呼叫此範本：

```xml
<div data-sly-use.nav="navTemplate.html" data-sly-call="${nav.foo}"></div>
```

>[!NOTE]
>
>如需有關 Use-API 的詳細資訊，請參閱：
>
>* [Java Use-API](use-api-java.md)
>* [JavaScript Use-API](use-api-javascript.md)


#### 使用 data-sly-use 取得資源 {#data-sly-use-with-resources}

如此可讓您使用 `data-sly-use` 直接在 HTL 中取得資源，而不需要撰寫程式碼來取得。

例如：

```xml
<div data-sly-use.product=“/etc/commerce/product/12345”>
  ${ product.title }
</div>
```

>[!TIP]
>
>也請參閱[路徑非永遠必要](#path-not-required)一節

### unwrap {#unwrap}

`data-sly-unwrap` 會從產生的標記中移除主機元素，同時保留其內容。 如此可讓您排除 HTL 表示邏輯中所需但在實際輸出中不需要的元素。

不過，應謹慎使用此陳述式。 通常最好讓 HTL 標記盡可能接近預期的輸出標記。 換句話說，在新增 HTL 區塊陳述式時，盡可能只在現有的 HTML 中加註，而不要引進新元素。

例如，這個

```xml
<p data-sly-use.nav="navigation.js">Hello World</p>
```

將會產生

```xml
<p>Hello World</p>
```

而這個

```xml
<p data-sly-use.nav="navigation.js" data-sly-unwrap>Hello World</p>
```

將會產生

```xml
Hello World
```

也可以有條件地將元素解除包裝：

```xml
<div class="popup" data-sly-unwrap="${isPopup}">content</div>
```

### set {#set}

`data-sly-set` 會使用預先定義的值來定義新的識別碼。

```xml
<span data-sly-set.profile="${user.profile}">Hello, ${profile.firstName} ${profile.lastName}!</span>
<a class="profile-link" href="${profile.url}">Edit your profile</a>
```

### text {#text}

`data-sly-text` 會將其主機元素的內容替換為指定的文字。

例如，這個

```xml
<p>${properties.jcr:description}</p>
```

相當於

```xml
<p data-sly-text="${properties.jcr:description}">Lorem ipsum</p>
```

兩者都會將 `jcr:description` 的值顯示為段落文字。 第二個方法的優點是，它允許對 HTML 加上低調的註解，同時保留原始設計者的靜態預留位置內容。

### attribute {#attribute}

`data-sly-attribute` 會將屬性新增到主機元素。

例如，這個

```xml
<div title="${properties.jcr:title}"></div>
```

相當於

```xml
<div title="Lorem Ipsum" data-sly-attribute.title="${properties.jcr:title}"></div>
```

兩者都會將 `title` 屬性設定為 `jcr:title` 的值。 第二個方法的優點是，它允許對 HTML 加上低調的註解，同時保留原始設計者的靜態預留位置內容。

屬性的解析順序是從左到右，而屬性最右邊的執行個體 (常值或透過 `data-sly-attribute` 進行定義) 會優先於其左邊定義的相同屬性的任何執行個體 (定義為常值或透過 `data-sly-attribute` 進行定義)。

請注意，如果屬性 (`literal` 或透過 `data-sly-attribute` 所設定) 值評估為空字串，則會從最終標記中移除該屬性。 此規則的唯一例外是，將會保留設定為常值空字串的常值屬性。 例如，

```xml
<div class="${''}" data-sly-attribute.id="${''}"></div>
```

會產生

```xml
<div></div>
```

但是

```xml
<div class="" data-sly-attribute.id=""></div>
```

會產生

```xml
<div class=""></div>
```

若要設定多個屬性，請傳遞一個對應物件，其中包含與屬性和其值相對應的機碼值組。 例如，假設

```xml
attrMap = {
    title: "myTitle",
    class: "myClass",
    id: "myId"
}
```

然後

```xml
<div data-sly-attribute="${attrMap}"></div>
```

會產生

```xml
<div title="myTitle" class="myClass" id="myId"></div>
```

### 元素 {#element}

`data-sly-element` 會取代主機元素的元素名稱。

例如，

```xml
<h1 data-sly-element="${titleLevel}">text</h1>
```

將 `h1` 替換為 `titleLevel` 的值。

基於安全理由，`data-sly-element` 只接受以下元素名稱：

```xml
a abbr address article aside b bdi bdo blockquote br caption cite code col colgroup
data dd del dfn div dl dt em figcaption figure footer h1 h2 h3 h4 h5 h6 header i ins
kbd li main mark nav ol p pre q rp rt ruby s samp section small span strong sub
sup table tbody td tfoot th thead time tr u var wbr
```

若要設定其他元素，必須關閉 XSS 安全性 (`@context='unsafe'`)。

### test {#test}

`data-sly-test` 會有條件地移除主機元素及其內容。 `false` 的值會移除元素；`true` 則會保留元素。

例如，只有當 `isShown` 為 `true` 時，才會呈現 `p` 元素及其內容：

```xml
<p data-sly-test="${isShown}">text</p>
```

可將測試的結果指派給可於稍後使用的變數。 這通常會用來建構「if else」邏輯，因為沒有明確的 else 陳述式：

```xml
<p data-sly-test.abc="${a || b || c}">is true</p>
<p data-sly-test="${!abc}">or not</p>
```

變數在設定後，會在 HTL 檔案中擁有全域範圍。

以下是對值進行比較的幾個範例：

```xml
<div data-sly-test="${properties.jcr:title == 'test'}">TEST</div>
<div data-sly-test="${properties.jcr:title != 'test'}">NOT TEST</div>

<div data-sly-test="${properties['jcr:title'].length > 3}">Title is longer than 3</div>
<div data-sly-test="${properties['jcr:title'].length >= 0}">Title is longer or equal to zero </div>

<div data-sly-test="${properties['jcr:title'].length > aemComponent.MAX_LENGTH}">
    Title is longer than the limit of ${aemComponent.MAX_LENGTH}
</div>
```

### repeat {#repeat}

使用 `data-sly-repeat` 時，您可以根據指定的清單多次重複某個元素。

```xml
<div data-sly-repeat="${currentPage.listChildren}">${item.name}</div>
```

其運作方式與 `data-sly-list` 相同，唯一例外是您不需要容器元素。

以下範例說明您也可以參照屬性的 *item*：

```xml
<div data-sly-repeat="${currentPage.listChildren}" data-sly-attribute.class="${item.name}">${item.name}</div>
```

### list {#list}

`data-sly-list` 會針對提供的物件中的每個可列舉的屬性重複主機元素的內容。

以下是簡單迴圈：

```xml
<dl data-sly-list="${currentPage.listChildren}">
    <dt>index: ${itemList.index}</dt>
    <dd>value: ${item.title}</dd>
</dl>
```

清單範圍內可使用以下預設變數：

* `item`：反覆運算中的目前項目。
* `itemList`：保存以下屬性的物件：
* `index`：以零為主的計數器 (`0..length-1`)。
* `count`：以一為主的計數器 (`1..length`)。
* `first`：`true` 表示目前的項目是第一個項目。
* `middle`：`true` 表示目前的項目不是第一個也不是最後一個項目。
* `last`：`true` 表示目前的項目是最後一個項目。
* `odd`：`true` 表示 `index` 是奇數。
* `even`：`true` 表示 `index` 是偶數。

在 `data-sly-list` 陳述式上定義識別碼可讓您重新命名 `itemList` 和 `item` 變數。 `item` 將成為 `<variable>`，`itemList` 則會成為 `<variable>List`。

```xml
<dl data-sly-list.child="${currentPage.listChildren}">
    <dt>index: ${childList.index}</dt>
    <dd>value: ${child.title}</dd>
</dl>
```

您也可以動態地存取屬性：

```xml
<dl data-sly-list.child="${myObj}">
    <dt>key: ${child}</dt>
    <dd>value: ${myObj[child]}</dd>
</dl>
```

### resource {#resource}

`data-sly-resource` 包含透過 sling 解析和呈現過程呈現指示的資源的結果。

簡單資源包括：

```xml
<article data-sly-resource="path/to/resource"></article>
```

#### 路徑非永遠必要 {#path-not-required}

請注意，如果您已經有資源，不需要搭配 `data-sly-resource` 使用路徑。 如果您已經有資源，可以直接使用它。

例如，以下是正確的。

```xml
<sly data-sly-resource="${resource.path @ decorationTagName='div'}"></sly>
```

不過，以下也是完全可以接受的。

```xml
<sly data-sly-resource="${resource @ decorationTagName='div'}"></sly>
```

基於以下理由，建議您盡可能直接使用資源。

* 如果您已經有資源，使用路徑重新解析是多餘且不必要的工作。
* 當您已經有資源時，使用路徑可能會產生非預期結果，因為 Sling 資源可以被包裝或合成，且不在給定路徑中提供。

#### 選項 {#resource-options}

選項可使用許多其他變體：

操控資源的路徑：

```xml
<article data-sly-resource="${ @ path='path/to/resource'}"></article>
<article data-sly-resource="${'resource' @ prependPath='my/path'}"></article>
<article data-sly-resource="${'my/path' @ appendPath='resource'}"></article>
```

新增 (或取代) 選擇器：

```xml
<article data-sly-resource="${'path/to/resource' @ selectors='selector'}"></article>
```

新增、取代或移除多個選擇器：

```xml
<article data-sly-resource="${'path/to/resource' @ selectors=['s1', 's2']}"></article>
```

在現有選擇器中新增選擇器：

```xml
<article data-sly-resource="${'path/to/resource' @ addSelectors='selector'}"></article>
```

從現有選擇器中移除一些選擇器：

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

變更 WCM 模式：

```xml
<article data-sly-resource="${'path/to/resource' @ wcmmode='disabled'}"></article>
```

預設會停用 AEM 裝飾標記，decorationTagName 選項可讓您重新啟用這些標記，而 cssClassName 則可在該元素中新增類別。

```xml
<article data-sly-resource="${'path/to/resource' @ decorationTagName='span',
cssClassName='className'}"></article>
```

>[!NOTE]
>
>AEM 提供簡單清晰的邏輯，可控制包住所含元素的裝飾標記。 如需詳細資訊，請參閱開發元件文件中的[裝飾標記](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/implementing/developing/full-stack/components-templates/decoration-tag.html?lang=zh-Hant)。

### include {#include}

`data-sly-include` 在由其對應的範本引擎進行處理時，會將主機元素的內容替換為指示的 HTML 範本檔案 (HTL、JSP、ESP 等) 所產生的標記。 所含檔案的呈現上下文將不會包含目前 HTL 上下文 (包含檔案的上下文)；因此，若要包含 HTL 檔案，必須在所含檔案中重複目前的 `data-sly-use` (在這種情況下，通常最好使用 `data-sly-template` 和 `data-sly-call`)

簡單的包含：

```xml
<section data-sly-include="path/to/template.html"></section>
```

可透過相同方式包含 JSP：

```xml
<section data-sly-include="path/to/template.jsp"></section>
```

選項可讓您操控檔案的路徑：

```xml
<section data-sly-include="${ @ file='path/to/template.html'}"></section>
<section data-sly-include="${'template.html' @ prependPath='my/path'}"></section>
<section data-sly-include="${'my/path' @ appendPath='template.html'}"></section>
```

您也可以變更 WCM 模式：

```xml
<section data-sly-include="${'template.html' @ wcmmode='disabled'}"></section>
```

### Request-attributes {#request-attributes}

在 `data-sly-include` 和 `data-sly-resource` 中，您可以傳遞 `requestAttributes` 以便在接收 HTL 指令碼中使用它們。

如此可讓您正確地將參數傳入指令碼或元件中。

```xml
<sly data-sly-use.settings="com.adobe.examples.htl.core.hashmap.Settings"
        data-sly-include="${ 'productdetails.html' @ requestAttributes=settings.settings}" />
```

Settings 類別的 Java 程式碼，Map 是用來傳入 requestAttributes：

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

例如，透過 Sling 模型，您可以取用指定的 `requestAttributes` 的值。

在此範例中，layout 是透過 Use 類別中的 Map 所插入：

```xml
@Model(adaptables=SlingHttpServletRequest.class)
public class ProductSettings {
  @Inject @Optional @Default(values="empty")
  public String layout;

}
```

### template 和 call {#template-call}

template 區塊可以像函數呼叫一樣使用：在其宣告中，它們可以取得參數，然後可以在呼叫它們時傳遞這些參數。 它們也允許遞迴。

`data-sly-template` 會定義範本。 HTL 不會輸出主機元素及其內容

`data-sly-call` 會呼叫使用 data-sly-template 所定義的範本。 所呼叫之範本的內容 (可選擇加以參數化) 會取代呼叫的主機元素的內容。

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

位於另一個檔案的範本可使用 `data-sly-use` 進行初始化。 請注意，在此情況下，`data-sly-use` 和 `data-sly-call` 也可能會置於相同元素上：

```xml
<div data-sly-use.lib="templateLib.html">
    <div data-sly-call="${lib.one}"></div>
    <div data-sly-call="${lib.two @ title=properties.jcr:title}"></div>
</div>
```

支援範本遞迴：

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

## sly 元素 {#sly-element}

`<sly>` HTML 標記可用來移除目前的元素，以便只顯示其子系。 其功能類似於 `data-sly-unwrap` 區塊元素：

```xml
<!--/* This will display only the output of the 'header' resource, without the wrapping <sly> tag */-->
<sly data-sly-resource="./header"></sly>
```

雖然 `<sly>` 標記不是有效的 HTML 5 標記，但可以使用 `data-sly-unwrap` 將其顯示在最終輸出內：

```xml
<sly data-sly-unwrap="${false}"></sly> <!--/* outputs: <sly></sly> */-->
```

`<sly>` 元素的目標是為了更清楚地表示該元素不是輸出。 如果您要的話，還是可以使用 `data-sly-unwrap`。

就像 `data-sly-unwrap` 一樣，請盡量少用它。
