---
title: HTL快速入門
seo-title: HTL快速入門
description: AEM支援的HTL將JSP取代為AEM中偏好的伺服器端範本系統。
seo-description: HTML範本語言- HTL-由Adobe Experience Manager支援，將JSP取代為AEM中偏好的伺服器端範本系統。
uuid: 4a7d6748-8cdf-4280-a85 d-6c5319 abf487
content-type: 引用
topic-tags: 簡介
discoiquuid: 3fc2ca75-0d68-489d-bd1 c-1d4 d4 fd730 c61 a
mwpw-migration-script-version: 2017-10-12T214658.665-0400
translation-type: tm+mt
source-git-commit: 271c355ae56e16e309853b02b8ef09f2ff971a2e

---


# HTL快速入門 {#getting-started-with-htl}

Adobe Experience Manager(AEM)支援的HTML範本語言(HTL)將JSP(JavaServer Pages)取代為AEM中偏好的伺服器端範本系統。

>[!NOTE]
>
>若要執行此頁面上提供的大部分範例，可使用稱為「閱讀Eval Print Loop [](https://github.com/Adobe-Marketing-Cloud/aem-htl-repl) 」的即時執行環境。
>
>AEM社群已產生一系列有關使用HTL的 [文章、影片和網路研討會](related-community-articles.md) 。

## HTL over JSP {#htl-over-jsp}

建議新的AEM專案使用HTML範本語言，因為它可提供多項優點與JSP相比。不過，若是現有的專案，只有在未來數年內仍能維持現有JSP時，移轉才有意義。

但是移轉至HTL並不一定完全可行，因為以HTL撰寫的元件與以JSP或ESP撰寫的元件相容。這表示現有的專案無法使用HTL for新元件，同時仍保有現有元件的JSP。

即使在相同元件內，HTL檔案也可以與JSP和ESP搭配使用。下列範例顯示 **如何** 從JSP檔案加入HTL檔案， **以及第行** 如何從HTL檔案中包含JSP檔案：

```xml
<cq:include script="template.html"/>
<sly data-sly-include="template.jsp"/>
```

### 常見問題{#frequently-asked-questions}

開始使用HTML範本語言之前，先回答一些與JSP與HTL主題相關的問題。

**HTL是否有任何JSP限制？**與JSP相比，HTL並沒有真正的限制，因為使用HTL時，JSP也能實現。但是，在某些方面，HTL的設計比JSP更嚴格，而且可以在單一JSP檔案中實現所有功能，可能需要將它分開至Java類別或JavaScript檔案才能在HTL中實現。但是一般而言，要確保邏輯與標記之間有良好的差異。

**HTL是否支援JSP標籤庫？**否，但如 [「載入用戶端資料庫](getting-started.md#loading-client-libraries) 」區段中所示， [範本與呼叫](block-statements.md#template-call) 陳述式提供類似模式。

**AEM專案可以延伸HTL功能嗎？****否，但如 [「載入用戶端資料庫](getting-started.md#loading-client-libraries) 」區段中所示， [範本與呼叫](block-statements.md#template-call) 陳述式提供類似模式。
不行，他們無法做到。HTL具備強大的擴充機制，可重復使用邏輯- [API](getting-started.md#use-api-for-accessing-logic) -和標記( [範本與呼叫](block-statements.md#template-call) 陳述式)，可用來模組化專案程式碼。

**HTL優於JSP的主要優點為何？**安全性和專案效率是主要優點，詳細說明請參閱 [「概述](overview.md)」。

**JSP最終會消失嗎？**在目前的日期中，沒有這些線上計劃。

## HTL的基本概念 {#fundamental-concepts-of-htl}

HTML範本語言使用運算式語言，將內容片段插入產生的標記，以及HTML資料屬性，以便在標記區塊上定義陳述式(如條件或重復)。當HTL編譯為Java Servlet時，運算式和HTL資料屬性都會完全伺服器端評估，而產生的HTML中不會顯示任何內容。

### 區塊和運算式 {#blocks-and-expressions}

以下是第一個範例，可能包含在 **`template.html`** 檔案中：

```xml
<h1 data-sly-test="${properties.jcr:title}">
    ${properties.jcr:title}
</h1>
```

可區分兩種不同語法：

* **[區塊陳述式](block-statements.md)**以有條件地顯示 **&
amp；t；h& amp；gt；** 元素，則會使用 `[data-sly-test](block-statements.md#test)` HTML資料屬性。HTL提供多個屬性，可附加行為至任何HTML元素，並加上前置詞 `data-sly`。

* **[運算式語言](expression-language.md)**HTL運算式會以字元 `${` 分隔 `}`。在執行時期，會評估這些運算式，並將其值插入傳出HTML串流中。

上述兩頁提供語法可用的詳細功能清單。

### SY元素 {#the-sly-element}

>[!NOTE]
>
>SY元素已隨附於AEM6.1或HTL1.1。
>
>在此之前，必須改用 `[data-sly-unwrap](block-statements.md)` 屬性。

HTL的一個主要概念是，有可能重復使用現有的HTML元素來定義區塊陳述式，以避免插入額外的分隔字元來定義陳述式開始和結束的位置。此標記不顯眼的標記可將靜態HTML轉換為功能正常的動態範本，其優點在於無法破壞HTML程式碼的有效性，因此仍能正確顯示，即使靜態檔案亦然。

不過，有時不會在必須插入區塊陳述式的確切位置上建立現有元素。對於這種情況，可以插入特殊的SY元素，在執行附加的區塊陳述式並據以顯示其內容時，自動從輸出移除。

因此，下列範例：

```xml
<sly data-sly-test="${properties.jcr:title && properties.jcr:description}">
    <h1>${properties.jcr:title}</h1>
    <p>${properties.jcr:description}</p>
</sly>
```

將會輸出如下HTML，但只有在兩者皆有，以及 **`jcr:title`** 定義 **`jcr:decription`** 的屬性均為空白時，才會輸出：

```xml
<h1>MY TITLE</h1>
<p>MY DESCRIPTION</p>
```

但要謹慎處理的一個問題，則是只有在沒有現有元素時，才會使用SY元素來表示區塊陳述式，因為SY元素會降低語言提供的值，以便在動態HTML變動態時不更改靜態HTML。

例如，如果先前的範例已包裝在DIV元素內，則新增的SY元素會具侮辱性：

```xml
<div>
    <sly data-sly-test="${properties.jcr:title && properties.jcr:description}">
        <h1>${properties.jcr:title}</h1>
        <p>${properties.jcr:description}</p>
    </sly>
</div>
```

而DIV元素可能已與條件加上註解：

```xml
<div data-sly-test="${properties.jcr:title && properties.jcr:description}">
    <h1>${properties.jcr:title}</h1>
    <p>${properties.jcr:description}</p>
</div>
```

### HTL注釋 {#htl-comments}

下列範例顯示 **第行** 的HTL註解， **以及第行** 的HTML備注：

```xml
<!--/* An HTL Comment */-->
<!-- An HTML Comment -->
```

HTL註解是HTML注釋，其他JavaScript類似語法。整個HTL注釋及任何內的任何內容都會被處理器完全忽略，並從輸出中移除。

但會傳遞標準HTML留言的內容，並評估注釋中的運算式。

HTML留言不能包含HTL注釋，反之亦然。

### 特殊內容 {#special-contexts}

為了能夠充分運用HTL，務必要瞭解它以HTML語法為基礎的效果。

### 元素和屬性名稱 {#element-and-attribute-names}

運算式只能放置在HTML文字或屬性值中，但不能在元素名稱或屬性名稱內，或者不是有效的HTML。為了動態設定元素名稱， [`data-sly-element`](block-statements.md#element) 可在所需元素上使用陳述式，並動態設定屬性名稱，甚至一次設定多個屬性，則可使用 [`data-sly-attribute`](block-statements.md#attribute) 陳述式。

```xml
<h1 data-sly-element="${myElementName}" data-sly-attribute="${myAttributeMap}">...</h1>
```

### 不含區塊陳述式的內容 {#contexts-without-block-statements}

HTL使用資料屬性來定義區塊陳述式，因此無法在下列上下文中定義此區塊陳述式，而且僅能使用運算式：

* HTML注釋
* 指令碼元素
* 樣式元素

原因在於內容的內容是文字而非HTML，而包含HTML元素則會被視為簡單的字元資料。因此，沒有真正的HTML元素，也無法執行 **`data-sly`** 屬性。

這聽起來可能很棒，但是它是需要的，因為HTML範本語言不應被濫用來產生不HTML的輸出。以下 [「存取邏輯」區段的「存取邏輯](getting-started.md#use-api-for-accessing-logic) 」區段會說明如何從範本呼叫其他邏輯，如果需要為這些內容準備複雜輸出，則可加以使用。例如，從後端傳送資料至前端指令碼的簡單方式，就是擁有元件的邏輯產生JSON字串，然後使用簡單的HTL運算式將它放置在資料屬性中。

下列範例說明HTML註解的行為，但在指令碼或樣式元素中，會觀察相同的行為：

```xml
<!--
    The title is: ${properties.jcr:title}
    <h1 data-sly-test="${properties.jcr:title}">${properties.jcr:title}</h1>
-->
```

會輸出如下HTML：

```xml
<!--
    The title is: MY TITLE
    <h1 data-sly-test="MY TITLE">MY TITLE</h1>
-->
```

### 需要明確的內容 {#explicit-contexts-required}

如下面 [「自動內容感知逸出](getting-started.md#automatic-context-aware-escaping) 」一節所述，HTL的一個目標是透過自動套用內容感知逸出至所有運算式，降低引入跨網站指令碼(XSS)弱點的風險。HTL可自動偵測放置在HTML標記內的運算式內容，但不會分析內嵌JavaScript或CSS的語法，因此需由開發人員明確指定要套用至該運算式的確切上下文。

由於無法在XSS弱點中套用正確的逸出結果，因此HTL會在尚未宣告內容時移除指令碼和樣式內容中的所有運算式輸出。

以下是如何在指令碼和樣式內設定運算式內容的範例：

```xml
<script> var trackingID = "${myTrackingID @ context='scriptString'}"; </script>
<style> a { font-family: "${myFont @ context='styleString'}"; } </style>
```

如需如何控制逸出的詳細資訊，請參閱 [「運算式語言顯示內容](expression-language.md#display-context) 」區段。

### 特殊內容的解除限制 {#lifting-limitations-of-special-contexts}

在需要略過指令碼、樣式和留言內容限制的特殊情況下，可以將其內容隔離在個別HTL檔案中。位於其檔案中的所有內容將會被HTL解讀為一般HTML片段，以避免可能被納入其中的內容。

如需範例，請參閱 [「使用用戶端範本](getting-started.md#working-with-client-side-templates) 」區段。

>[!CAUTION]
>
>此項技術可引入跨網站指令碼(XSS)弱點，如果使用此弱點，應謹慎研究。實作與依賴此做法相比，通常是更好的方式。

## HTL的一般功能 {#general-capabilities-of-htl}

本節會快速逐步解說HTML範本語言的一般功能。

### 使用API存取邏輯 {#use-api-for-accessing-logic}

請考慮下列範例：

```xml
<p data-sly-use.logic="logic.js">${logic.title}</p>
```

下 `logic.js` 列為伺服器端已執行的JavaScript檔案：

```
use(function () {
    return {
        title: currentPage.getTitle().substring(0, 10) + "..."
    };
});
```

由於HTML範本語言不允許在標記內混合程式碼，所以它提供使用API擴充機制，輕鬆從範本執行程式碼。

上述範例使用伺服器端執行的JavaScript將標題縮短至10個字元，但也可能使用Java程式碼，提供完全符合資格的Java類別名稱。通常，商業邏輯應該以Java建立，但是當元件需要某些特定檢視的變更時，Java API提供的部分特定檢視會方便使用某些伺服器端執行的JavaScript執行該動作。

有關下列章節的更多資訊：

* [資料詳盡使用陳述式中的章節](block-statements.md#use) 說明可使用該陳述式完成的一切。
* [「使用API」頁面](use-api.md) 提供一些資訊，以協助您在Java中或以JavaScript編寫邏輯。
* 並詳細說明如何編寫邏輯、 [JavaScript Use-API](use-api-javascript.md) 和 [Java Use-API](use-api-java.md) 頁面應說明。

### 自動內容感知逸出 {#automatic-context-aware-escaping}

請考慮下列範例：

```xml
<p data-sly-use.logic="logic.js">
    <a href="${logic.link}" title="${logic.title}">
        ${logic.text}
    </a>
</p>
```

在大部分的範本語言中，此範例可能會造成跨網站指令碼(XSS)弱點，因為即使所有變數自動被HTML逸出，屬性 `href` 仍必須特別具有URL逸出。這種省略是最常見的錯誤之一，因為它很容易被遺忘，而且很難自動辨別。

為協助這種情況，HTML範本語言會自動逸出每個變數，並將其放置在所放置的內容中。這是因為HTL瞭解HTML語法的事實。

假設下列 `logic.js` 檔案：

```
use(function () {
    return {
        link:  "#my link's safe",
        title: "my title's safe",
        text:  "my text's safe"
    };
});
```

初始範例將會產生下列輸出：

```xml
<p>
    <a href="#my%20link%27s%20safe" title="my title&#39;s safe">
        my text&#39;s safe
    </a>
</p>
```

Notice how the two attribute got escaped differently, because HTL knows that `href` and `src` attributes must be escaped for URI context. 此外，如果URI開始， **`javascript:`**屬性會完全移除，除非內容明確變更為其他內容。

如需如何控制逸出的詳細資訊，請參閱 [「運算式語言顯示內容](expression-language.md#display-context) 」區段。

### 自動移除空白屬性 {#automatic-removal-of-empty-attributes}

請考慮下列範例：

```xml
<p class="${properties.class}">some text</p>
```

如果 `class` 屬性的值為空白，HTML範本語言會自動從輸出中移除整個 `class` 屬性。

同樣地，這是可能的，因為HTL瞭解HTML語法，因此只有當動態值不是空值時，才有條件地顯示屬性。因為它可避免新增條件區塊的屬性區塊，因此標記無效且無法讀取。

此外，運算式中放置的變數類型重要：

* **String:**
   * **非空白：** 將字串設為屬性值。
   * **空白：** 完全移除屬性。

* **數字：** 將值設為屬性值。

* **布林函數:**
   * **true：** 顯示沒有值的屬性(作為布林HTML屬性
   * **false：** 完全移除屬性。

以下範例說明布林運算式如何允許控制布林HTML屬性：

```xml
<input type="checkbox" checked="${properties.isChecked}"/>
```

對於設定屬性， [`data-sly-attribute`](block-statements.md#attribute) 此陳述式也很有用。

## 使用HTL的常用模式 {#common-patterns-with-htl}

本節提供一些常見案例，以及如何使用HTML範本語言來最佳解決它們。

### 載入用戶端程式庫 {#loading-client-libraries}

在HTL中，用戶端程式庫會透過AEM提供的Helper範本載入，可透過AEM提供 [`data-sly-use`](block-statements.md#use)。此檔案提供三個範本，可透過下列方式呼叫 [`data-sly-call`](block-statements.md#template-call)：

* **`css`** - 僅載入參考用戶端程式庫的CSS檔案。
* **`js`** - 僅載入參考用戶端程式庫的JavaScript檔案。
* **`all`** - 載入參考用戶端程式庫的所有檔案(CSS和JavaScript)。

每個help範本都預期 **`categories`** 一個選項可參照所需的用戶端程式庫。該選項可以是字串值的陣列，或包含逗號分隔值清單的字串。

以下為兩個簡短範例：

### 一次完整載入多個用戶端程式庫 {#loading-multiple-client-libraries-fully-at-once}

```xml
<sly data-sly-use.clientlib="/libs/granite/sightly/templates/clientlib.html"
     data-sly-call="${clientlib.all @ categories=['myCategory1', 'myCategory2']}"/>
```

### 參照頁面不同區段中的用戶端庫 {#referencing-a-client-library-in-different-sections-of-a-page}

```xml
<!doctype html>
<html data-sly-use.clientlib="/libs/granite/sightly/templates/clientlib.html">
    <head>
        <!-- HTML meta-data -->
        <sly data-sly-call="${clientlib.css @ categories='myCategory'}"/>
    </head>
    <body>
        <!-- page content -->
        <sly data-sly-call="${clientlib.js @ categories='myCategory'}"/>
    </body>
</html>
```

在上述第二個範例中，如果HTML **`head`** 和 **`body`** 元素放置在不同的檔案中， **`clientlib.html`** 則必須將範本載入每個需要該範本的檔案中。

[範本與呼叫](block-statements.md#template-call) 陳述式上的章節提供如何宣告和呼叫此範本的詳細資訊。

### 將資料傳遞至用戶端 {#passing-data-to-the-client}

要將資料傳遞給一般客戶，但使用HTL更多，則是使用資料屬性。

以下範例說明邏輯(也可能以Java編寫)可用來非常便利地序列化至JSON的物件，將其傳遞給用戶端，然後很容易放入資料屬性中：

```xml
<!--/* template.html file: */-->
<div data-sly-use.logic="logic.js" data-json="${logic.json}">...</div>
```

```
/* logic.js file: */
use(function () {
    var myData = {
        str: "foo",
        arr: [1, 2, 3]
    };

    return {
        json: JSON.stringify(myData)
    };
});
```

從那裡，很容易想像客戶端JavaScript如何存取該屬性並剖析JSON。例如，對應的JavaScript會放置在用戶端程式庫中：

```
var elements = document.querySelectorAll("[data-json]");
for (var i = 0; i < elements.length; i++) {
    var obj = JSON.parse(elements[i].dataset.json);
    //console.log(obj);
}
```

### 使用用戶端範本 {#working-with-client-side-templates}

特殊內容的 [「解除特殊內容](getting-started.md#lifting-limitations-of-special-contexts) 」區段中說明的一個特殊案例，就是要編寫位於 **指令碼** 元素內的用戶端範本(例如手寫列)。在這種情況下可安全使用此技巧的原因，是 **因為指令碼** 元素不包含JavaScript，而是更進一步的HTML元素。以下是如何運作的範例：

```xml
<!--/* template.html file: */-->
<script id="entry-section" type="text/template"
    data-sly-include="entry-section.html"></script>

<!--/* entry-section.html file: */-->
<div class="entry-section">
    <h2 data-sly-test="${properties.entrySectionTitle}">
        ${properties.entrySectionTitle}
    </h2>
    {{#each entry}}<h3><a href="{{link}}">{{title}}</a></h3>{{/each}}
</div>
```

如上所述，將包含在 **`script`** 元素中的標記可以包含HTL區塊陳述式，而運算式不需要提供明確的上下文，因為「掌上型」範本的內容已隔離在自己的檔案中。此外，此範例顯示伺服器端執行HTL(如 **`h2`** 元素上)如何混合用戶端執行的範本語言，例如「掌上型電腦」(在 **`h3`** 元素上顯示)。

不過，更現代化的技巧完全是使用HTML **`template`** 元素，因為這樣就不需要將範本內容隔離為個別檔案。

**下一步閱讀：**

* [運算式語言](expression-language.md) -詳細瞭解可在HTL運算式內完成的內容。
* [區塊陳述式](block-statements.md) -可探索HTL中可用的所有區塊陳述式，以及如何使用。
