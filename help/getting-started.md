---
title: HTL 快速入門
description: 來認識 HTL，這是在 AEM 環境中使用 HTML 時首選且推薦使用的伺服器端範本系統，並了解此語言的主要概念及其基本結構。
exl-id: c95eb1b3-3b96-4727-8f4f-d54e7136a8f9
source-git-commit: 5ab1275c984135fe946f36905bbc979cf19edd80
workflow-type: tm+mt
source-wordcount: '2170'
ht-degree: 100%

---


# HTL 快速入門 {#getting-started-with-htl}

HTML 範本語言 (HTL) 是 Adobe Experience Manager 中首選且推薦使用的 HTML 伺服器端範本系統。 因為在所有 HTML 伺服器端範本系統中，HTL 檔案會指定 HTML 本身的內容、一些基本的表現邏輯，以及要在執行階段運算的變數，藉此定義傳送給瀏覽器的輸出。

本文件概述 HTL 的用途，並介紹其基本概念和語言結構。

>[!TIP]
>
>本文件說明 HTL 的用途，並概述其基本結構和概念。 若您對於特定語法有任何疑問，請參閱 [HTL 規格。](specification.md)

## HTL 層 {#layers}

在 AEM 中使用 HTL 時會有不同的分層。

1. **[HTL 規格](specification.md)** - HTL 的規格是開放原始碼且不受平台限制，任何人均可自由實施。
1. **[Sling HTL Scripting Engine](specification.md)** - Sling 專案已建立 HTL 實施參考，由 AEM 使用。
1. **[AEM 擴充功能](specification.md)** - AEM 以 Sling HTL Scripting Engine 為基礎進行擴充，為開發人員提供 AEM 專用的便利功能。

此 HTL 文件的重點是如何使用 HTL 開發 AEM 解決方案。 因此，它會提及所有三個層，必要時會連結外部資源。

## HTL 的基本概念 {#fundamental-concepts-of-htl}

HTML 範本語言會使用運算式語言將內容片段插入呈現的標記中，並使用 HTML5 資料屬性來定義標記區塊上的陳述式 (例如條件或反覆運算)。 當 HTL 被編譯到 Java Servlet 中時，運算式和 HTL 資料屬性全都會在伺服器端進行評估，產生的 HTML 中看不到任何內容。

>[!TIP]
>
>若要執行此頁提供的大多數範例，可以使用稱為 [Read Eval Print Loop](https://github.com/adobe/aem-htl-repl) 的即時執行環境。

### 區塊和運算式 {#blocks-and-expressions}

以下是第一個範例，此範例可以依原樣納入 `template.html` 檔案中：

```xml
<h1 data-sly-test="${properties.jcr:title}">
    ${properties.jcr:title}
</h1>
```

有兩種不同類型的語法：

* **區塊陳述式** - 若要有條件地顯示 `<h1>` 元素，則會使用 `data-sly-test` HTML5 資料屬性。 HTL 提供多個這類屬性，允許對任何 HTML 元素附加行為，而且全都有 `data-sly` 前置詞。
* **運算式語言** - HTL 運算式會以 `${` 和 `}` 字元分隔。 在執行階段，這些運算式會進行運算，並將其值插入傳出的 HTML 資料流中。

有關兩種語法的詳細資訊，請參閱 [HTL 規格](specification.md)。

### SLY 元素 {#the-sly-element}

HTL 的核心概念是讓我們可以選擇重複使用現有 HTML 元素來定義區塊陳述式，便不需要插入額外的分隔符號來定義陳述式的開頭和結尾處。這種將靜態 HTML 轉換為正常的動態範本的低調標記註解提供了不破壞 HTML 程式碼有效性的優點，所以即使當作靜態檔案也能正確顯示。

不過，有時在必須插入區塊陳述式的確切位置可能沒有現有元素。 在這種情況下，我們可以插入特殊 `sly` 元素，在執行附加的區塊陳述式並顯示其相應內容時，它會自動從輸出中移除。

下列範例...

```xml
<sly data-sly-test="${properties.jcr:title && properties.jcr:description}">
    <h1>${properties.jcr:title}</h1>
    <p>${properties.jcr:description}</p>
</sly>
```

...將會輸出類似下列 HTML 的內容，但前提是 `jcr:title` 和 `jcr:description` 屬性均有定義，而且兩者都不是空白：

```xml
<h1>MY TITLE</h1>
<p>MY DESCRIPTION</p>
```

請記得一件事，唯有沒有任何現有元素可能使用此區塊元素作為附註時，才能使用 `sly` 元素。因為 `sly` 元素會阻止此語言所提供的值，以便將 HTML 變成動態時避免更動靜態 HTML。

例如，如果上一個範例已經包在 `div` 元素內，則新增的 `sly` 元素會變成誤用：

```xml
<div>
    <sly data-sly-test="${properties.jcr:title && properties.jcr:description}">
        <h1>${properties.jcr:title}</h1>
        <p>${properties.jcr:description}</p>
    </sly>
</div>
```

而且 `div` 元素可能曾經使用以下條件作為附註：

```xml
<div data-sly-test="${properties.jcr:title && properties.jcr:description}">
    <h1>${properties.jcr:title}</h1>
    <p>${properties.jcr:description}</p>
</div>
```

### HTL 註解 {#htl-comments}

以下範例的第 1 行顯示 HTL 註解，並在第 2 行顯示 HTML 註解。

```xml
<!--/* An HTL Comment */-->
<!-- An HTML Comment -->
```

HTL 註解是有其他類似 JavaScript 語法的 HTML 註解。 處理器將會完全忽略整個 HTL 註解及其中的所有內容，並從輸出中將其移除。

不過，標準 HTML 註解的內容將會被傳遞，而且註解內的運算式將會受到評估。

HTML 註解不可包含 HTL 註解，反之亦然。

### 特殊上下文 {#special-contexts}

為了能夠充分利用 HTL，一定要了解根據 HTML 語法使用它的結果。

如需詳細資訊，請參閱 HTL 規格的「[顯示格式文法](https://github.com/adobe/htl-spec/blob/1.4/SPECIFICATION.md#121-display-context)」一節。

### 元素和屬性名稱 {#element-and-attribute-names}

唯有在 HTML 文字或屬性值當中才能使用運算式，而元素名稱或屬性名稱當中不能使用運算式，否則便不是有效的 HTML。 若要以動態方式設定元素名稱，可以在所要的元素上使用 `data-sly-element` 陳述式；若要以動態方式設定屬性名稱，甚至是一次設定多個屬性，可以使用 `data-sly-attribute` 陳述式。

```xml
<h1 data-sly-element="${myElementName}" data-sly-attribute="${myAttributeMap}">...</h1>
```

### 沒有區塊陳述式的上下文 {#contexts-without-block-statements}

由於 HTL 會使用資料屬性來定義區塊陳述式，所以無法在以下上下文中定義這類區塊陳述式，只能使用運算式：

* HTML 註解
* 指令碼元素
* 樣式元素

原因是因為這些上下文的內容是文字，而不是 HTML，而且包含的 HTML 元素會被視為簡單的字元資料。 所以如果沒有真正的 HTML 元素，也無法執行 `data-sly` 屬性。

也許聽起來像是很大的限制，然而這是我們想要的限制，因為不該誤用 HTML 範本語言來產生不是 HTML 的輸出。 底下[用於存取邏輯的 Use-API](#use-api-for-accessing-logic) 一節會介紹如何從範本呼叫其他邏輯，可以在需要為這些上下文準備複雜輸出時使用它。 例如，從後端將資料傳送到前端指令碼的一個簡單方法是讓元件的邏輯產生 JSON 字串，然後可以使用簡單 HTL 運算式將該字串置於資料屬性中。

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

如同底下[自動上下文感知逸出](#automatic-context-aware-escaping)一節中所述，HTL 的一個目的是藉由自動套用上下文感知逸出到所有運算式來減少引進跨網站指令碼 (XSS) 安全漏洞的風險。 雖然 HTL 可以自動偵測置於 HTML 標記內的運算式的上下文，但是它無法分析內嵌 JavaScript 或 CSS 的語法，所以有賴開發人員明確指定必須套用到這類運算式的確切上下文。

由於未套用正確的逸出會導致 XSS 安全漏洞，所以 HTL 會在未宣告上下文時移除指令碼和樣式上下文中的所有運算式輸出。

以下範例說明如何為置於指令碼和樣式內的運算式設定上下文：

```xml
<script> var trackingID = "${myTrackingID @ context='scriptString'}"; </script>
<style> a { font-family: "${myFont @ context='styleString'}"; } </style>
```

如需有關如何控制跳脫的詳細資訊，請參閱 HTL 規格的「[運算式語言顯示設定語法](https://github.com/adobe/htl-spec/blob/master/SPECIFICATION.md#121-display-context)」一節。

## HTL 的一般功能 {#general-capabilities-of-htl}

本節將快速介紹 HTML 範本語言的一般功能。

### 用於存取邏輯的 Use-API {#use-api-for-accessing-logic}

HTML 範本語言 (HTL) Java Use-API 讓 HTL 檔案能夠透過 `data-sly-use` 存取自訂 Java 類別中的 helper 方法。 如此可讓所有複雜的商業邏輯都封裝在 Java 程式碼中，而 HTL 程式碼只需處理直接標記的產生。

如需詳細資訊，請參閱文件「[HTL Java Use-API](java-use-api.md)」。

### 自動上下文感知逸出 {#automatic-context-aware-escaping}

考量下列範例：

```xml
<p data-sly-use.logic="logic.js">
    <a href="${logic.link}" title="${logic.title}">
        ${logic.text}
    </a>
</p>
```

在大多數的範本語言中，此範例可能會產生跨網站指令碼 (XSS) 安全漏洞，因為即便所有變數都自動進行 HTML 逸出，`href` 屬性還是必須專門進行 URL 逸出。 這種遺漏是最常見的錯誤之一，因為很容易被忘記，而且很難藉由自動化方式發現。

為了協助解決此問題，HTML 範本語言會根據每個變數在設定語法中的位置自動跳脫。 這個方法之所以可行，是因為 HTL 了解 HTML 的語法。

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

請注意這兩個屬性如何用不同方式跳脫，因為 HTL 知道 URI 設定語法必須跳脫 `href` 和 `src` 屬性。 此外，如果 URI 的開頭為 `javascript:`，則屬性會被完全移除，除非設定語法明確變更為其他內容。

如需有關如何控制跳脫的詳細資訊，請參閱 HTL 規格的「[運算式語言顯示設定語法](https://github.com/adobe/htl-spec/blob/master/SPECIFICATION.md#121-display-context)」一節。

### 自動移除空屬性 {#automatic-removal-of-empty-attributes}

請考量下列範例：

```xml
<p class="${properties.class}">some text</p>
```

如果 `class` 屬性 (Property) 的值剛好是空的，則 HTML 範本語言會自動從輸出中移除整個 `class` 屬性 (Attribute)。

同樣的，這個方法之所以可行，是因為 HTL 了解 HTML 語法，而且只有在屬性值不是空白的情況下，可以有條件地顯示具有動態值的屬性。 這是非常方便的做法，可以避免在屬性周圍增加條件區塊，而這些區塊可能讓標記變成無效和無法讀取。

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

若要設定屬性，`data-sly-attribute` 陳述式可能也很實用。

## HTL 的常見模式 {#common-patterns-with-htl}

本節會介紹一些常見情境，以及如何透過 HTML 範本語言以最佳方式解決其中的問題。

### 載入用戶端程式庫 {#loading-client-libraries}

在 HTL 中，用戶端程式庫是透過 AEM 提供的 Helper 範本來載入，該範本可透過 `data-sly-use` 進行存取。 此檔案中有三個範本可用，這些範本可透過 `data-sly-call` 來呼叫：

* **`css`** - 僅載入參照的用戶端程式庫的 CSS 檔案。
* **`js`** - 僅載入參照的用戶端程式庫的 JavaScript 檔案。
* **`all`** - 載入參照的用戶端程式庫的所有檔案 (CSS 和 JavaScript)。

每個 helper 範本都需要 `categories` 選項來參照所需的用戶端程式庫。 那個選項可能是一系列的字串值，或是包含以逗號分隔的值清單的字串。

以下是兩個簡短的範例。

#### 一次完整載入多個用戶端程式庫 {#loading-multiple-client-libraries-fully-at-once}

```xml
<sly data-sly-use.clientlib="/libs/granite/sightly/templates/clientlib.html"
     data-sly-call="${clientlib.all @ categories=['myCategory1', 'myCategory2']}"/>
```

#### 參照頁面的不同區段中的用戶端程式庫 {#referencing-a-client-library-in-different-sections-of-a-page}

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

在這個範例中，如果 HTML `head` 和 `body` 元素分別放在不同檔案，則需要 `clientlib.html` 範本的每個檔案都必須載入此範本。

在 [HTL 規格](specification.md) 中有關範本和呼叫陳述式的小節，詳細說明宣告及呼叫這類範本的運作方式。

### 傳遞資料給用戶端 {#passing-data-to-the-client}

一般來說，使用 `data` 屬性傳遞資料給用戶端是最優雅也是最好的方式，對 HTL 來說更是如此。

以下範例說明如何使用邏輯 (也可以用 Java 撰寫) 方便您將傳遞給用戶端的物件序列化為 JSON，然後可以輕鬆地將邏輯放到 `data` 屬性中：

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

透過上述方法，不難想像用戶端 JavaScript 如何存取該屬性，然後再次剖析 JSON。例如，這會是放入用戶端資料庫中的對應的 JavaScript：

```javascript
var elements = document.querySelectorAll("[data-json]");
for (var i = 0; i < elements.length; i++) {
    var obj = JSON.parse(elements[i].dataset.json);
    //console.log(obj);
}
```

### 使用用戶端範本 {#working-with-client-side-templates}

在一個特殊情況下，可以合法使用「[解除特殊設定語法的限制](#lifting-limitations-of-special-contexts)」一節中所述的技巧，那就是編寫位於 `scrip` 元素中的用戶端範本 (例如 Handlebars)。 在上述情況下可以安全使用這個技巧，是因為 `script` 元素並未包含 JavaScript，與假設不同，反而是包含更多 HTML 元素。 以下是其運作方式的範例：

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

如上所示，會包含在 `script` 元素中的標記可以包含 HTL 區塊陳述式，而且運算式不需要提供明確的設定語法，因為 Handlebars 範本的內容在其本身的檔案中已被隔離。 此外，這個範例也說明如何將伺服器端執行的 HTL (例如在 `h2` 元素上) 與用戶端執行的範本語言 (例如顯示在 `h3` 元素上的 Handlebars) 混合在一起。

不過，一種更新型的技巧是改用 HTML `template` 元素，因為這樣做有個好處：不需要將範本的內容隔離到個別檔案中。

### 解除特殊設定語法的限制 {#lifting-limitations-of-special-contexts}

在需要略過指令碼、樣式和註解設定語法之限制的特殊情況下，可以將其內容隔離在另一個 HTL 檔案中。 位於自身檔案中的所有內容都將由 HTL 解譯為一般 HTML 片段，而忘記了可能包含該內容的限制性上下文。

如需範例，請參閱下面的[使用用戶端範本](#working-with-client-side-templates)一節。

>[!CAUTION]
>
>此技巧可能會帶來跨網站指令碼 (XSS) 漏洞，若是必須使用此技巧，應仔細研究安全性問題。 通常會有更好的方法可以實施相同的事，而不必依賴此一做法。
