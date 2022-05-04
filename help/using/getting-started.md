---
title: HTL 快速入門
description: AEM 支援的 HTL 取代了 JSP，成為 AEM 中適用於 HTML 的首選和推薦的伺服器端範本系統。
exl-id: c95eb1b3-3b96-4727-8f4f-d54e7136a8f9
source-git-commit: 8e70ee4921a7ea071ab7e06947824c371f4013d8
workflow-type: ht
source-wordcount: '2471'
ht-degree: 100%

---

# HTL 快速入門 {#getting-started-with-htl}

Adobe Experience Manager (AEM) 支援的 HTML 範本語言 (HTL) 是 AEM 中適用於 HTML 的首選和推薦的伺服器端範本系統。 它取代了舊版 AEM 中所使用的 JSP (JavaServer Pages)。

>[!NOTE]
>
>若要執行此頁提供的大多數範例，可以使用稱為[「讀取、求值、輸出」迴圈](https://github.com/Adobe-Marketing-Cloud/aem-htl-repl)的即時執行環境。

## HTL 優於 JSP {#htl-over-jsp}

建議新 AEM 專案最好使用 HTML 範本語言，因為相較於 JSP，HTL 提供了多項優點。 但如果是現有專案，只有在預估遷移比未來幾年維護現有 JSP 的工作量少時，遷移才有意義。

不過，移至 HTL 不見得是全有或全無的選擇，因為使用 HTL 撰寫的元件與使用 JSP 或 ESP 撰寫的元件相容。 這表示現有專案可以毫無問題地將 HTL 用於新元件，同時繼續將 JSP 用於現有元件。

即便是在相同元件中，HTL 檔案也可以與 JSP 和 ESP 一起使用。 以下範例在&#x200B;**第 1 行**&#x200B;顯示如何加入 JSP 檔案中的 HTL 檔案，並在&#x200B;**第 2 行**&#x200B;顯示如何加入 HTL 檔案中的 JSP 檔案：

```xml
<cq:include script="template.html"/>
<sly data-sly-include="template.jsp"/>
```

### 常見問答{#frequently-asked-questions}

在開始使用 HTML 範本語言之前，讓我們先來回答幾個與 JSP 和 HTL 主題有關的問題。

**HTL 是否有任何 JSP 沒有的限制？** - 相較於 JSP，HTL 沒有什麼真正的限制，因為可以使用 JSP 完成的事情應該也可以使用 HTL 達成。 不過有幾個方面，HTL 在設計上比 JSP 嚴格，可以在單一 JSP 檔案中完成的所有事情，可能需要分散到 Java 類別或 JavaScript 檔案中才能在 HTL 中達成。 但這通常需要確保邏輯與標記之間有良好的關注點分離。

**HTL 是否支援 JSP 標記庫？** - 否，但如同[載入用戶端程式庫](getting-started.md#loading-client-libraries)一節中所述，[template 和 call](block-statements.md#template-call) 陳述式可提供類似的模式。

**可以在 AEM 專案上擴充 HTL 功能嗎？** - 不行。 HTL 具有強大的擴充機制可重複使用邏輯 - [Use-API](getting-started.md#use-api-for-accessing-logic) - 和標記 ([template 和 call](block-statements.md#template-call) 陳述式)，這些可用來將專案的程式碼模組化。

**HTL 主要有哪些優點勝過 JSP？** - 安全性和專案效率是主要優點，這些在[概覽](overview.md)中有詳述。

**JSP 最終會消失嗎？** - 目前沒有這方面的計劃。

## HTL 的基本概念 {#fundamental-concepts-of-htl}

HTML 範本語言會使用運算式語言將內容片段插入呈現的標記中，並使用 HTML5 資料屬性來定義標記區塊上的陳述式 (例如條件或反覆運算)。 當 HTL 被編譯到 Java Servlet 中時，運算式和 HTL 資料屬性全都會在伺服器端進行評估，產生的 HTML 中看不到任何內容。

### 區塊和運算式 {#blocks-and-expressions}

以下是第一個範例，此範例可以依原樣納入 **`template.html`** 檔案中：

```xml
<h1 data-sly-test="${properties.jcr:title}">
    ${properties.jcr:title}
</h1>
```

可以區分兩種不同的語法：

* **[區塊陳述式](block-statements.md)** - 若要有條件地顯示 **&lt;h1>** 元素，則會使用 [`data-sly-test`](block-statements.md#test) HTML5 資料屬性。 HTL 提供多個這類屬性，可讓您在任何 HTML 元素中附加行為，而且全都有 `data-sly` 前置詞。

* **[運算式語言](expression-language.md)** - HTL 運算式會以 `${` 和 `}` 字元分隔。 在執行階段會評估這些運算式，並將其值插入傳出的 HTML 資料流中。

上面連結的兩個頁面提供了可用於語法的功能的詳細清單。

### SLY 元素 {#the-sly-element}

HTL 的一個核心概念是提供重複使用現有 HTML 元素以定義區塊陳述式的可能性，這樣就不需要插入額外的分隔符號來定義陳述式的開頭和結尾處。 這種將靜態 HTML 轉換為正常的動態範本的低調標記註解提供了不破壞 HTML 程式碼有效性的優點，所以即使當作靜態檔案也能正確顯示。

不過，有時在必須插入區塊陳述式的確切位置可能沒有現有元素。 在這種情況下，可以插入特殊 SLY 元素，系統將會自動從輸出中移除該元素，同時執行附加的區塊陳述式，並適當地顯示其內容。

所以下列範例：

```xml
<sly data-sly-test="${properties.jcr:title && properties.jcr:description}">
    <h1>${properties.jcr:title}</h1>
    <p>${properties.jcr:description}</p>
</sly>
```

將會輸出類似以下 HTML 的內容，但前提是同時定義了 **`jcr:title`** 和 **`jcr:description`** 屬性，而且兩者都不是空的：

```xml
<h1>MY TITLE</h1>
<p>MY DESCRIPTION</p>
```

請牢記一件事，僅在沒有可加註區塊陳述式的現有元素時才使用 SLY 元素，因為 SLY 元素會制止語言提供的值，在靜態 HTML 變成動態時不改變靜態 HTML。

例如，如果上述範例已經包在 DIV 元素內，則新增的 SLY 元素會是誤用的：

```xml
<div>
    <sly data-sly-test="${properties.jcr:title && properties.jcr:description}">
        <h1>${properties.jcr:title}</h1>
        <p>${properties.jcr:description}</p>
    </sly>
</div>
```

而且 DIV 元素可能會使用加註以下條件：

```xml
<div data-sly-test="${properties.jcr:title && properties.jcr:description}">
    <h1>${properties.jcr:title}</h1>
    <p>${properties.jcr:description}</p>
</div>
```

### HTL 註解 {#htl-comments}

以下範例在&#x200B;**第 1 行**&#x200B;顯示 HTL 註解，並在&#x200B;**第 2 行**&#x200B;顯示 HTML 註解：

```xml
<!--/* An HTL Comment */-->
<!-- An HTML Comment -->
```

HTL 註解是有其他類似 JavaScript 語法的 HTML 註解。 處理器將會完全忽略整個 HTL 註解及其中的所有內容，並從輸出中將其移除。

不過，標準 HTML 註解的內容將會被傳遞，而且註解內的運算式將會受到評估。

HTML 註解不可包含 HTL 註解，反之亦然。

### 特殊上下文 {#special-contexts}

為了能夠充分利用 HTL，一定要了解根據 HTML 語法使用它的結果。

### 元素和屬性名稱 {#element-and-attribute-names}

運算式只能置於 HTML 文字或屬性值中，但不能置於元素名稱或屬性名稱中，否則將不再是有效的 HTML。 若要以動態方式設定元素名稱，可以在所要的元素上使用 [`data-sly-element`](block-statements.md#element) 陳述式；若要以動態方式設定屬性名稱，甚至是一次設定多個屬性，可以使用 [`data-sly-attribute`](block-statements.md#attribute) 陳述式。

```xml
<h1 data-sly-element="${myElementName}" data-sly-attribute="${myAttributeMap}">...</h1>
```

### 沒有區塊陳述式的上下文 {#contexts-without-block-statements}

由於 HTL 會使用資料屬性來定義區塊陳述式，所以無法在以下上下文中定義這類區塊陳述式，只能使用運算式：

* HTML 註解
* 指令碼元素
* 樣式元素

原因是因為這些上下文的內容是文字，而不是 HTML，而且包含的 HTML 元素會被視為簡單的字元資料。 所以如果沒有真正的 HTML 元素，也無法執行 **`data-sly`** 屬性。

這可能聽起來像是很大的限制，但其實是理想的限制，因為不該濫用 HTML 範本語言來產生不是 HTML 的輸出。 底下[用於存取邏輯的 Use-API](getting-started.md#use-api-for-accessing-logic) 一節會介紹如何從範本呼叫其他邏輯，可以在需要為這些上下文準備複雜輸出時使用它。 例如，從後端將資料傳送到前端指令碼的一個簡單方法是讓元件的邏輯產生 JSON 字串，然後可以使用簡單 HTL 運算式將該字串置於資料屬性中。

以下範例說明 HTML 註解的行為，但在指令碼或樣式元素中，將觀察到相同的行為：

```xml
<!--
    The title is: ${properties.jcr:title}
    <h1 data-sly-test="${properties.jcr:title}">${properties.jcr:title}</h1>
-->
```

將會輸出類似以下 HTML 的內容：

```xml
<!--
    The title is: MY TITLE
    <h1 data-sly-test="MY TITLE">MY TITLE</h1>
-->
```

### 需要明確的上下文 {#explicit-contexts-required}

如同底下[自動上下文感知逸出](getting-started.md#automatic-context-aware-escaping)一節中所述，HTL 的一個目的是藉由自動套用上下文感知逸出到所有運算式來減少引進跨網站指令碼 (XSS) 安全漏洞的風險。 雖然 HTL 可以自動偵測置於 HTML 標記內的運算式的上下文，但是它無法分析內嵌 JavaScript 或 CSS 的語法，所以有賴開發人員明確指定必須套用到這類運算式的確切上下文。

由於未套用正確的逸出會導致 XSS 安全漏洞，所以 HTL 會在未宣告上下文時移除指令碼和樣式上下文中的所有運算式輸出。

以下範例說明如何為置於指令碼和樣式內的運算式設定上下文：

```xml
<script> var trackingID = "${myTrackingID @ context='scriptString'}"; </script>
<style> a { font-family: "${myFont @ context='styleString'}"; } </style>
```

如需有關如何控制逸出的詳細資訊，請參閱[運算式語言顯示上下文](expression-language.md#display-context)一節。

### 解除特殊上下文的限制 {#lifting-limitations-of-special-contexts}

在需要略過指令碼、樣式和註解上下文限制的特殊情況下，可以將其內容隔離在個別 HTL 檔案中。 位於自身檔案中的所有內容都將由 HTL 解譯為一般 HTML 片段，而忘記了可能包含該內容的限制性上下文。

如需範例，請參閱下面的[使用用戶端範本](getting-started.md#working-with-client-side-templates)一節。

>[!CAUTION]
>
>此技巧可能會帶來跨網站指令碼 (XSS) 安全漏洞，所以在使用此技巧時，應該對安全性方面仔細研究。 通常有比依賴此做法更好的方式可實作相同的事。

## HTL 的一般功能 {#general-capabilities-of-htl}

本節將快速介紹 HTML 範本語言的一般功能。

### 用於存取邏輯的 Use-API {#use-api-for-accessing-logic}

考量下列範例：

```xml
<p data-sly-use.logic="logic.js">${logic.title}</p>
```

以及放置在它旁邊的下列 `logic.js` 伺服器端執行的 JavaScript 檔案：

```javascript
use(function () {
    return {
        title: currentPage.getTitle().substring(0, 10) + "..."
    };
});
```

由於 HTML 範本語言不允許將程式碼混合在標記中，所以它提供 Use-API 延伸模組機制，以便從範本輕鬆地執行程式碼。

上述範例使用伺服器端執行的 JavaScript 將標題縮短為 10 個字元，但它原本也可藉由提供完整 Java 類別名稱來使用 Java 程式碼做相同的事。 一般來說，商業邏輯應該使用 Java 來建立，但是當元件需要從 Java API 提供的內容中進行一些檢視特有的變更時，使用某個伺服器端執行的 JavaScript 來做這件事會很方便。

下列章節會提供更多相關資訊：

* 有關 [`data-sly-use` 陳述式](block-statements.md#use)的章節說明了可使用該陳述式執行的所有工作。
* [Use-API 頁面](use-api.md)提供一些資訊來幫助您選擇使用 Java 或 JavaScript 撰寫邏輯。
* 若要取得如何撰寫邏輯的詳細資訊，[JavaScript Use-API](use-api-javascript.md) 和 [Java Use-API](use-api-java.md) 頁面應該會有幫助。

### 自動上下文感知逸出 {#automatic-context-aware-escaping}

考量下列範例：

```xml
<p data-sly-use.logic="logic.js">
    <a href="${logic.link}" title="${logic.title}">
        ${logic.text}
    </a>
</p>
```

在大多數的範本語言中，此範例可能會產生跨網站指令碼 (XSS) 安全漏洞，因為即便所有變數都自動進行 HTML 逸出，`href` 屬性還是必須專門進行 URL 逸出。 這種遺漏是最常見的錯誤之一，因為很容易被忘記，而且很難以自動化方式被發現。

為了協助解決此問題，HTML 範本語言會根據每個變數所在的上下文自動逸出該變數。 這是可行的，因為 HTL 了解 HTML 的語法。

假設有以下 `logic.js` 檔案：

```javascript
use(function () {
    return {
        link:  "#my link's safe",
        title: "my title's safe",
        text:  "my text's safe"
    };
});
```

然後最初範例會產生以下輸出：

```xml
<p>
    <a href="#my%20link%27s%20safe" title="my title&#39;s safe">
        my text&#39;s safe
    </a>
</p>
```

請注意這兩個屬性如何以不同方式逸出，因為 HTL 知道必須為 URI 上下文逸出 `href` 和 `src` 屬性。 此外，如果 URI 的開頭為 **`javascript:`**，則屬性將被完全移除，除非明確地將上下文變更為其他內容。

如需有關如何控制逸出的詳細資訊，請參閱[運算式語言顯示上下文](expression-language.md#display-context)一節。

### 自動移除空屬性 {#automatic-removal-of-empty-attributes}

請考量下列範例：

```xml
<p class="${properties.class}">some text</p>
```

如果 `class` 屬性 (Property) 的值剛好是空的，則 HTML 範本語言會自動從輸出中移除整個 `class` 屬性 (Attribute)。

這也是可行的，因為 HTL 了解 HTML 語法，因此可以有條件地顯示具有動態值的屬性，但前提是屬性值不是空的。 這樣會非常方便，因為可避免在屬性周圍加入條件區塊，這種區塊會讓標記變得無效且無法讀取。

此外，放在運算式中的變數類型很重要：

* **字串：**
   * **不是空的：**&#x200B;將字串設為屬性值。
   * **空的：**&#x200B;完全移除屬性。

* **數字：**&#x200B;將值設為屬性值。

* **布林值：**
   * **true：**&#x200B;顯示不含值的屬性 (當作布林值 HTML 屬性)
   * **false：**&#x200B;完全移除屬性。

以下範例說明布林值運算式如何允許控制布林值 HTML 屬性：

```xml
<input type="checkbox" checked="${properties.isChecked}"/>
```

若要設定屬性，[`data-sly-attribute`](block-statements.md#attribute) 陳述式可能也很實用。

## HTL 的常見模式 {#common-patterns-with-htl}

本節會介紹一些常見情境，以及如何透過 HTML 範本語言以最佳方式解決其中的問題。

### 載入用戶端程式庫 {#loading-client-libraries}

在 HTL 中，用戶端程式庫是透過 AEM 提供的 Helper 範本來載入，該範本可透過 [`data-sly-use`](block-statements.md#use) 進行存取。 此檔案中有三個範本可用，這些範本可透過 [`data-sly-call`](block-statements.md#template-call) 來呼叫：

* **`css`** - 僅載入參照的用戶端程式庫的 CSS 檔案。
* **`js`** - 僅載入參照的用戶端程式庫的 JavaScript 檔案。
* **`all`** - 載入參照的用戶端程式庫的所有檔案 (CSS 和 JavaScript)。

每個 helper 範本都需要 **`categories`** 選項來參照所需的用戶端程式庫。 這個選項可以是字串值陣列，或是包含逗號分隔值清單的字串。

以下是兩個簡短範例：

### 一次完整載入多個用戶端程式庫 {#loading-multiple-client-libraries-fully-at-once}

```xml
<sly data-sly-use.clientlib="/libs/granite/sightly/templates/clientlib.html"
     data-sly-call="${clientlib.all @ categories=['myCategory1', 'myCategory2']}"/>
```

### 參照頁面的不同區段中的用戶端程式庫 {#referencing-a-client-library-in-different-sections-of-a-page}

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

在上面的第二個範例中，如果 HTML **`head`** 和 **`body`** 元素被放置在不同檔案中，就必須在需要 **`clientlib.html`** 範本的每個檔案中載入它。

有關 [template 和 call](block-statements.md#template-call) 陳述式的章節提供了宣告及呼叫這類範本的運作方式的相關細節。

### 傳遞資料給用戶端 {#passing-data-to-the-client}

一般來說，傳遞資料給用戶端的最優雅也是最好的方式是使用資料屬性，對 HTL 來說更是如此。

以下範例說明如何使用邏輯 (也可以使用 Java 撰寫邏輯) 方便地將傳遞給用戶端的物件序列化為 JSON，然後就可以非常輕鬆地將其置於資料屬性中：

```xml
<!--/* template.html file: */-->
<div data-sly-use.logic="logic.js" data-json="${logic.json}">...</div>
```

```javascript
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

從那裡，就可以輕鬆地想像用戶端 JavaScript 如何存取該屬性，然後再次剖析 JSON。 例如，這會是將對應的 JavaScript 置於用戶端程式庫中：

```javascript
var elements = document.querySelectorAll("[data-json]");
for (var i = 0; i < elements.length; i++) {
    var obj = JSON.parse(elements[i].dataset.json);
    //console.log(obj);
}
```

### 使用用戶端範本 {#working-with-client-side-templates}

有一個特殊情況可以合法地使用[解除特殊上下文的限制](getting-started.md#lifting-limitations-of-special-contexts)一節中所述的技巧，那就是編寫位於 **script** 元素中的用戶端範本 (例如 Handlebars) 時。 在此情況下可以安全地使用這個技巧的原因是，**script** 元素沒有像假設情況那樣包含 JavaScript，而是包含更多 HTML 元素。 以下是其運作方式的範例：

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

如上所示，將包含在 **`script`** 元素中的標記可以包含 HTL 區塊陳述式，而且運算式不需要提供明確上下文，因為 Handlebars 範本的內容已被隔離在自己的檔案中。 此外，這個範例也說明如何將伺服器端執行的 HTL (例如在 **`h2`** 元素上) 與用戶端執行的範本語言 (例如顯示在 **`h3`** 元素上的 Handlebars) 混合在一起。

不過，一種更新型的技巧是改用 HTML **`template`** 元素，因為這樣做有個好處：不需要將範本的內容隔離到個別檔案中。

**閱讀後續章節：**

* [運算式語言](expression-language.md) - 詳細了解在 HTL 運算式中可以做哪些事。
* [區塊陳述式](block-statements.md) - 探索 HTL 中提供的所有區塊陳述式以及如何加以使用。
