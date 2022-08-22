---
title: HTL 快速入門
description: 瞭解HTL（首選和推薦的伺服器端模板系統）,AEM瞭解語言的主要概念及其基本結構。
exl-id: c95eb1b3-3b96-4727-8f4f-d54e7136a8f9
source-git-commit: 5ab1275c984135fe946f36905bbc979cf19edd80
workflow-type: tm+mt
source-wordcount: '2170'
ht-degree: 54%

---


# HTL 快速入門 {#getting-started-with-htl}

HTML模板語言(HTL)是Adobe Experience ManagerHTML的首選和推薦的伺服器端模板系統。 與所有HTML伺服器端模板化系統中一樣，HTL檔案通過指定HTML本身、一些基本演示邏輯和要在運行時評估的變數來定義發送到瀏覽器的輸出。

本文概述了HTL的目的，並介紹了語言的基本概念和結構。

>[!TIP]
>
>本檔案介紹了HTL的目的及其基本結構和概念的概述。 如果您對特定語法有疑問，請參閱 [HTL規範。](specification.md)

## HTL層 {#layers}

中使用的HTLAEM可由多個層定義。

1. **[HTL規範](specification.md)** - HTL是開放原始碼、平台無關的規範，任何人都可以自由實施。
1. **[Sling HTL指令碼引擎](specification.md)** - Sling項目建立了HTL的參考實施，供使用AEM。
1. **[擴AEM展](specification.md)**  — 在AEMSling HTL指令碼引擎之上構建，以便為開發人員提供特定的便利功AEM能。

本HTL文檔重點介紹使用HTL開發解決AEM方案。 因此，它觸及了所有三層，根據需要將外部資源連結起來。

## HTL 的基本概念 {#fundamental-concepts-of-htl}

HTML 範本語言會使用運算式語言將內容片段插入呈現的標記中，並使用 HTML5 資料屬性來定義標記區塊上的陳述式 (例如條件或反覆運算)。 當 HTL 被編譯到 Java Servlet 中時，運算式和 HTL 資料屬性全都會在伺服器端進行評估，產生的 HTML 中看不到任何內容。

>[!TIP]
>
>若要執行此頁提供的大多數範例，可以使用稱為[「讀取、求值、輸出」迴圈](https://github.com/adobe/aem-htl-repl)的即時執行環境。

### 區塊和運算式 {#blocks-and-expressions}

以下是第一個範例，此範例可以依原樣納入 `template.html` 檔案中：

```xml
<h1 data-sly-test="${properties.jcr:title}">
    ${properties.jcr:title}
</h1>
```

可以區分兩種不同的語法：

* **塊語句**  — 有條件顯示 `<h1>` 元素，a `data-sly-test` HTML5資料屬性被使用。 HTL提供多個此類屬性，允許將行為附加到任何HTML元素，並且所有屬性都以前置詞 `data-sly`。
* **表達式語言** - HTL表達式由 `${` 和 `}` 字元。 在執行階段會評估這些運算式，並將其值插入傳出的 HTML 資料流中。

查看 [HTL規範](specification.md) 的子菜單。

### SLY 元素 {#the-sly-element}

HTL的一個核心概念是提供重新使用現有HTML元素來定義塊語句的可能性，這避免了插入附加分隔符來定義語句的開始和結束位置。 這種將靜態 HTML 轉換為正常的動態範本的低調標記註解提供了不破壞 HTML 程式碼有效性的優點，所以即使當作靜態檔案也能正確顯示。

不過，有時在必須插入區塊陳述式的確切位置可能沒有現有元素。 對於這種情況，可以插入一個特別 `sly` 將自動從輸出中刪除的元素，同時執行附加的塊語句並相應地顯示其內容。

以下示例……

```xml
<sly data-sly-test="${properties.jcr:title && properties.jcr:description}">
    <h1>${properties.jcr:title}</h1>
    <p>${properties.jcr:description}</p>
</sly>
```

...將輸出類似於跟隨HTML的內容，但只有同時存在， `jcr:title` 和 `jcr:description` 定義的屬性，如果這兩個屬性都為空：

```xml
<h1>MY TITLE</h1>
<p>MY DESCRIPTION</p>
```

要記住的一件事就是 `sly` 元素時，當沒有現有元素可以用block語句進行注釋時。 這是因為 `sly` 元素阻止語言提供的值，使其動態化時不會改變靜態HTML。

例如，如果上一個示例已包裝在 `div` 元素，然後添加 `sly` 元素將是濫用的：

```xml
<div>
    <sly data-sly-test="${properties.jcr:title && properties.jcr:description}">
        <h1>${properties.jcr:title}</h1>
        <p>${properties.jcr:description}</p>
    </sly>
</div>
```

和 `div` 元素可能已用條件進行注釋：

```xml
<div data-sly-test="${properties.jcr:title && properties.jcr:description}">
    <h1>${properties.jcr:title}</h1>
    <p>${properties.jcr:description}</p>
</div>
```

### HTL 註解 {#htl-comments}

以下示例顯示第一行上的HTL注釋和第二行上的HTML注釋。

```xml
<!--/* An HTL Comment */-->
<!-- An HTML Comment -->
```

HTL 註解是有其他類似 JavaScript 語法的 HTML 註解。 處理器將會完全忽略整個 HTL 註解及其中的所有內容，並從輸出中將其移除。

不過，標準 HTML 註解的內容將會被傳遞，而且註解內的運算式將會受到評估。

HTML 註解不可包含 HTL 註解，反之亦然。

### 特殊上下文 {#special-contexts}

為了能夠充分利用 HTL，一定要了解根據 HTML 語法使用它的結果。

請參閱 [顯示上下文部分](https://github.com/adobe/htl-spec/blob/1.4/SPECIFICATION.md#121-display-context) HTL規範的詳細資訊。

### 元素和屬性名稱 {#element-and-attribute-names}

表達式只能放置在HTML文本或屬性值中，但不能放在元素名稱或屬性名稱中，否則將不再是有效的HTML。 若要以動態方式設定元素名稱，可以在所要的元素上使用 `data-sly-element` 陳述式；若要以動態方式設定屬性名稱，甚至是一次設定多個屬性，可以使用 `data-sly-attribute` 陳述式。

```xml
<h1 data-sly-element="${myElementName}" data-sly-attribute="${myAttributeMap}">...</h1>
```

### 沒有區塊陳述式的上下文 {#contexts-without-block-statements}

由於 HTL 會使用資料屬性來定義區塊陳述式，所以無法在以下上下文中定義這類區塊陳述式，只能使用運算式：

* HTML 註解
* 指令碼元素
* 樣式元素

原因是因為這些上下文的內容是文字，而不是 HTML，而且包含的 HTML 元素會被視為簡單的字元資料。 所以如果沒有真正的 HTML 元素，也無法執行 `data-sly` 屬性。

這聽起來可能是一個重要限制，但是它是一個理想的限制，因為不應濫用HTML模板語言來生成非HTML的輸出。 底下[用於存取邏輯的 Use-API](#use-api-for-accessing-logic) 一節會介紹如何從範本呼叫其他邏輯，可以在需要為這些上下文準備複雜輸出時使用它。 例如，從後端將資料傳送到前端指令碼的一個簡單方法是讓元件的邏輯產生 JSON 字串，然後可以使用簡單 HTL 運算式將該字串置於資料屬性中。

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

有關如何控制轉義的詳細資訊，請參閱 [表達式語言顯示上下文](https://github.com/adobe/htl-spec/blob/master/SPECIFICATION.md#121-display-context) HTL規範的一部分。

## HTL 的一般功能 {#general-capabilities-of-htl}

本節將快速介紹 HTML 範本語言的一般功能。

### 用於存取邏輯的 Use-API {#use-api-for-accessing-logic}

HTML 範本語言 (HTL) Java Use-API 讓 HTL 檔案能夠透過 `data-sly-use` 存取自訂 Java 類別中的 helper 方法。 如此可讓所有複雜的商業邏輯都封裝在 Java 程式碼中，而 HTL 程式碼只需處理直接標記的產生。

查看文檔 [HTL Java Use-API](java-use-api.md) 的子菜單。

### 自動上下文感知逸出 {#automatic-context-aware-escaping}

考量下列範例：

```xml
<p data-sly-use.logic="logic.js">
    <a href="${logic.link}" title="${logic.title}">
        ${logic.text}
    </a>
</p>
```

在大多數的範本語言中，此範例可能會產生跨網站指令碼 (XSS) 安全漏洞，因為即便所有變數都自動進行 HTML 逸出，`href` 屬性還是必須專門進行 URL 逸出。 這種遺漏是最常見的錯誤之一，因為容易被遺忘，並且難以以自動方式發現。

為了協助解決此問題，HTML 範本語言會根據每個變數所在的上下文自動逸出該變數。 由於HTL瞭解HTML的語法，因此這是可能的。

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

注意這兩個屬性是如何以不同的方式轉義的，因為HTL知道 `href` 和 `src` 必須為URI上下文轉義屬性。 此外，如果 URI 的開頭為 `javascript:`，則屬性將被完全移除，除非明確地將上下文變更為其他內容。

有關如何控制轉義的詳細資訊，請參閱 [表達式語言顯示上下文](https://github.com/adobe/htl-spec/blob/master/SPECIFICATION.md#121-display-context) HTL規範的一部分。

### 自動移除空屬性 {#automatic-removal-of-empty-attributes}

請考量下列範例：

```xml
<p class="${properties.class}">some text</p>
```

如果 `class` 屬性 (Property) 的值剛好是空的，則 HTML 範本語言會自動從輸出中移除整個 `class` 屬性 (Attribute)。

同樣，這是可能的，因為HTL瞭解HTML語法，因此只有其值不為空時，才能有條件地顯示具有動態值的屬性。 這非常方便，因為它避免了在屬性周圍添加條件塊，這會使標籤無效且不可讀。

此外，放在運算式中的變數類型很重要：

* **字串：**
   * **不是空的：**&#x200B;將字串設為屬性值。
   * **空的：**&#x200B;完全移除屬性。

* **數字：**&#x200B;將值設為屬性值。

* **布林值：**
   * **true：**&#x200B;顯示不含值的屬性 (當作布林值 HTML 屬性)
   * **false：**&#x200B;完全移除屬性。

下面是一個示例，說明布爾表達式如何允許控制布爾HTML屬性：

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

每個 helper 範本都需要 `categories` 選項來參照所需的用戶端程式庫。 該選項可以是字串值的陣列，也可以是包含逗號分隔值清單的字串。

以下是兩個簡短的示例。

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

在本例中，如果HTML `head` 和 `body` 元素被放置在不同的檔案中， `clientlib.html` 然後，模板必須載入到每個需要它的檔案中。

中有關模板和調用語句的部分 [HTL規範](specification.md) 提供了有關聲明和調用此類模板的工作方式的詳細資訊。

### 傳遞資料給用戶端 {#passing-data-to-the-client}

一般而言，將資料傳遞給客戶端的最好、最優雅的方法是使用HTL `data` 屬性。

以下示例說明如何使用邏輯（也可以用Java編寫）來方便地將要傳遞給客戶端的對象序列化為JSON，然後，該對象可以輕鬆地放入 `data` 屬性：

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

從那裡，就可以輕鬆地想像用戶端 JavaScript 如何存取該屬性，然後再次剖析 JSON。 例如，這將是要放入客戶端庫的相應JavaScript:

```javascript
var elements = document.querySelectorAll("[data-json]");
for (var i = 0; i < elements.length; i++) {
    var obj = JSON.parse(elements[i].dataset.json);
    //console.log(obj);
}
```

### 使用用戶端範本 {#working-with-client-side-templates}

有一個特殊情況可以合法地使用[解除特殊上下文的限制](#lifting-limitations-of-special-contexts)一節中所述的技巧，那就是編寫位於 `scrip` 元素中的用戶端範本 (例如 Handlebars) 時。 在此情況下可以安全地使用這個技巧的原因是，`script` 元素沒有像假設情況那樣包含 JavaScript，而是包含更多 HTML 元素。 以下是其運作方式的範例：

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

如上所示，將包含在 `script` 元素可以包含HTL塊語句，而且表達式不需要提供顯式上下文，因為Handlebars模板的內容已被隔離在其自己的檔案中。 此外，這個範例也說明如何將伺服器端執行的 HTL (例如在 `h2` 元素上) 與用戶端執行的範本語言 (例如顯示在 `h3` 元素上的 Handlebars) 混合在一起。

不過，一種更新型的技巧是改用 HTML `template` 元素，因為這樣做有個好處：不需要將範本的內容隔離到個別檔案中。

### 解除特殊上下文的限制 {#lifting-limitations-of-special-contexts}

在需要繞過指令碼、樣式和注釋上下文的限制的特殊情況下，可以將其內容隔離在單獨的HTL檔案中。 位於自身檔案中的所有內容都將由 HTL 解譯為一般 HTML 片段，而忘記了可能包含該內容的限制性上下文。

如需範例，請參閱下面的[使用用戶端範本](#working-with-client-side-templates)一節。

>[!CAUTION]
>
>該技術可以引入跨站指令碼(XSS)漏洞，如果必須使用該漏洞，應仔細研究其安全方面。 通常有比依賴此做法更好的方式可實作相同的事。
