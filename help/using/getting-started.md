---
title: 開始使用 HTL
description: AEM支援的HTL取代JSP，成為AEM中HTML的偏好和建議的伺服器端範本系統。
translation-type: tm+mt
source-git-commit: ee712ef61018b5e05ea052484e2a9a6b12e6c5c8
workflow-type: tm+mt
source-wordcount: '2490'
ht-degree: 0%

---


# 開始使用 HTL {#getting-started-with-htl}

Adobe Experience Manager(AEM)支援的HTML範本語言(HTL)是AEM中HTML的偏好和建議的伺服器端範本系統。 它取代舊版AEM中使用的JSP(JavaServer Pages)。

>[!NOTE]
>
>要運行本頁上提供的大多數示例，可以使用名為「讀取評 [估打印循環](https://github.com/Adobe-Marketing-Cloud/aem-htl-repl) 」的即時執行環境。

## HTL over JSP {#htl-over-jsp}

建議新的AEM專案使用「HTML範本語言」，因為與JSP相比，它提供多項優點。 但是，對於現有項目，只有在預計遷移比在未來幾年維護現有JSP的工作量小的情況下，遷移才有意義。

但是，改用HTL並非一無是處的選擇，因為以HTL編寫的元件與以JSP或ESP編寫的元件相容。 這表示現有專案可以毫無問題地使用HTL來建立新元件，同時保留現有元件的JSP。

即使在同一個元件中，HTL檔案也可與JSP和ESP搭配使用。 以下示例說 **明第1行** ，如何從JSP檔案包含HTL檔案，以及第2 **行** ，如何從HTL檔案包含JSP檔案：

```xml
<cq:include script="template.html"/>
<sly data-sly-include="template.jsp"/>
```

### 常見問題{#frequently-asked-questions}

開始使用HTML範本語言之前，我們先先回答一些與JSP與HTL主題相關的問題。

**HTL是否有JSP所無法提供的限制？** -與JSP相比，HTL並沒有其他限制，因為使用JSP可做的事也應該可以透過HTL達成。 但是，HTL在設計上比JSP要嚴格，而且在單一JSP檔案中可實現的功能，可能需要將HTL分割為Java類別或JavaScript檔案，才能在HTL中實現。 但這通常是為了確保邏輯和標籤之間的關注之間很好地分離。

**HTL是否支援JSP標籤庫？** -否，但如「載入客戶端庫」 [部分所示](getting-started.md#loading-client-libraries) ，模板 [](block-statements.md#template-call) 和調用語句提供類似的模式。

**AEM專案可擴充HTL功能嗎？** -不，他們不能。 HTL具有強大的擴充機制，可重複使用邏輯- [Use-API](getting-started.md#use-api-for-accessing-logic) —— 和標籤( [template &amp; call](block-statements.md#template-call) statements)，可用來模組化專案的程式碼。

**HTL優於JSP的主要優點為何？** -安全性和項目效率是主要優點，詳見 [概述](overview.md)。

**JSP最終會消失嗎？** -在目前日期，沒有這些計畫。

## HTL的基本概念 {#fundamental-concepts-of-htl}

HTML範本語言使用運算式語言將內容片段插入轉譯的標籤中，而HTML5資料屬性則用來定義標籤區塊上的陳述式（例如條件或小版本）。 當HTL編譯為Java Servlet時，運算式和HTL資料屬性都會完全在伺服器端進行評估，而產生的HTML中不會顯示任何內容。

### 區塊與運算式 {#blocks-and-expressions}

以下是第一個範例，它可像在檔案中一樣包 **`template.html`** 含：

```xml
<h1 data-sly-test="${properties.jcr:title}">
    ${properties.jcr:title}
</h1>
```

有兩種不同的語法可以區分：

* **[區塊陳述式](block-statements.md)**-若要有條件地顯&#x200B;**示&lt;h1>元素**，請使用[`data-sly-test`](block-statements.md#test)HTML5資料屬性。 HTL提供多種此類屬性，可將行為附加至任何HTML元素，而且所有屬性都加上前置詞`data-sly`。

* **[運算式語言](expression-language.md)**- HTL運算式以字元和`${`分隔`}`。 在執行時期，會評估這些運算式，並將其值插入傳出的HTML串流。

上述連結的兩頁提供語法可用功能的詳細清單。

### SLY元素 {#the-sly-element}

HTL的核心概念是提供重複使用現有HTML元素來定義區塊陳述式的可能性，避免插入其他分隔字元來定義陳述式的開始和結束位置。 此標籤的不顯眼註解可將靜態HTML轉換為功能正常的動態範本，可讓您不破壞HTML程式碼的有效性，因此即使是靜態檔案，也能正常顯示。

但是，有時可能沒有現有元素存在於必須插入塊語句的確切位置。 在這種情況下，可以插入將自動從輸出中移除的特殊SLY元素，同時執行附加的塊語句並相應地顯示其內容。

所以，以下例子：

```xml
<sly data-sly-test="${properties.jcr:title && properties.jcr:description}">
    <h1>${properties.jcr:title}</h1>
    <p>${properties.jcr:description}</p>
</sly>
```

將輸出類似以下HTML的內容，但只有在定義了兩者、a **`jcr:title`** 和 **`jcr:description`** a屬性且兩者均為空時：

```xml
<h1>MY TITLE</h1>
<p>MY DESCRIPTION</p>
```

要記住的一點是，當沒有現有元素可以用塊陳述式加上註解時，僅使用SLY元素，因為SLY元素會阻止語言提供的值，使靜態HTML變為動態。

例如，如果上一個範例原本已包裝在DIV元素內，則新增的SLY元素會是辱罵性的：

```xml
<div>
    <sly data-sly-test="${properties.jcr:title && properties.jcr:description}">
        <h1>${properties.jcr:title}</h1>
        <p>${properties.jcr:description}</p>
    </sly>
</div>
```

而DIV元素可能已加上條件注釋：

```xml
<div data-sly-test="${properties.jcr:title && properties.jcr:description}">
    <h1>${properties.jcr:title}</h1>
    <p>${properties.jcr:description}</p>
</div>
```

>[!NOTE]
>
>SLY元素已隨附於AEM 6.1或HTL 1.1。
>
>在此之前，必 [`data-sly-unwrap`](block-statements.md) 須改用屬性。

### HTL注釋 {#htl-comments}

下列範例顯 **示第1行** HTL註解和第2 **行** HTML註解：

```xml
<!--/* An HTL Comment */-->
<!-- An HTML Comment -->
```

HTL注釋是HTML注釋，其中附加類似JavaScript的語法。 整個HTL注釋，以及處理器將完全忽略其中的任何內容，並從輸出中刪除。

但是，標準HTML注釋的內容將傳遞，並且評估注釋中的表達式。

HTML注釋不能包含HTL注釋，反之亦然。

### 特殊上下文 {#special-contexts}

為了能夠最佳地運用HTL，請務必瞭解它以HTML語法為基礎的後果。

### 元素和屬性名稱 {#element-and-attribute-names}

運算式只能放入HTML文字或屬性值中，但不能放在元素名稱或屬性名稱中，否則將不再是有效的HTML。 為了動態設定元素名稱， [`data-sly-element`](block-statements.md#element) 語句可用於所需的元素，並動態設定屬性名稱，即使一次設定多個屬性，也可以使用 [`data-sly-attribute`](block-statements.md#attribute) 該語句。

```xml
<h1 data-sly-element="${myElementName}" data-sly-attribute="${myAttributeMap}">...</h1>
```

### 沒有塊語句的上下文 {#contexts-without-block-statements}

由於HTL使用資料屬性來定義區塊陳述式，因此無法在下列內容中定義此類區塊陳述式，因此只能在此處使用陳述式：

* HTML注釋
* 指令碼元素
* 樣式元素

原因是這些上下文的內容是文字而非HTML，而且包含的HTML元素會被視為簡單的字元資料。 因此，若沒有真正的HTML元素，也無法執行 **`data-sly`** 屬性。

這聽起來可能像是一個很大的限制，但是它是需要的，因為HTML範本語言不應被濫用來產生非HTML的輸出。 以下 [](getting-started.md#use-api-for-accessing-logic) 的「存取邏輯的使用API」一節介紹如何從範本呼叫其他邏輯，如果需要範本來準備複雜輸出，則可使用範本。 例如，從後端傳送資料至前端指令碼的簡單方式是讓元件的邏輯產生JSON字串，然後再將它放入資料屬性中，並使用簡單的HTL運算式。

以下範例說明HTML注釋的行為，但在指令碼或樣式元素中，會觀察到相同的行為：

```xml
<!--
    The title is: ${properties.jcr:title}
    <h1 data-sly-test="${properties.jcr:title}">${properties.jcr:title}</h1>
-->
```

將會輸出類似下列HTML的內容：

```xml
<!--
    The title is: MY TITLE
    <h1 data-sly-test="MY TITLE">MY TITLE</h1>
-->
```

### 需要顯式上下文 {#explicit-contexts-required}

如下方的「自 [](getting-started.md#automatic-context-aware-escaping) 動內容感知逸出」一節所述，HTL的目標之一是透過自動將內容感知逸出套用至所有陳述式，降低引入跨網站指令碼(XSS)弱點的風險。 雖然HTL可自動偵測置於HTML標籤內之運算式的上下文，但它不會分析內嵌JavaScript或CSS的語法，因此需仰賴開發人員明確指定應套用至這些運算式的確切上下文。

由於XSS弱點中未套用正確的逸出結果，因此，HTL會在未宣告上下文時，移除指令碼和樣式內容中所有運算式的輸出。

以下是如何設定置入指令碼和樣式內之運算式的上下文範例：

```xml
<script> var trackingID = "${myTrackingID @ context='scriptString'}"; </script>
<style> a { font-family: "${myFont @ context='styleString'}"; } </style>
```

如需如何控制逸出的詳細資訊，請參閱「運算式語言顯 [示內容」一節](expression-language.md#display-context) 。

### 特殊語境的解除限制 {#lifting-limitations-of-special-contexts}

在需要略過指令碼、樣式和註解內容限制的特殊情況下，可以將其內容隔離在個別的HTL檔案中。 HTL會將位於其檔案中的所有項目解讀為一般的HTML片段，而忽略可能包含它的限制內容。

如需范 [例，請參閱「使用用戶端範本](getting-started.md#working-with-client-side-templates) 」一節。

>[!CAUTION]
>
>此技術可能會引入跨網站指令碼(XSS)弱點，若採用此技術，應仔細研究其安全性。 通常，實施同樣的事情，有比依靠這種做法更好的方法。

## HTL的一般功能 {#general-capabilities-of-htl}

本節快速介紹HTML範本語言的一般功能。

### 存取邏輯的使用API {#use-api-for-accessing-logic}

請考慮下列範例：

```xml
<p data-sly-use.logic="logic.js">${logic.title}</p>
```

在伺服器 `logic.js` 端執行的JavaScript檔案旁放置：

```javascript
use(function () {
    return {
        title: currentPage.getTitle().substring(0, 10) + "..."
    };
});
```

由於HTML範本語言不允許在標籤內混用程式碼，因此它提供Use-API擴充功能機制，以輕鬆從範本執行程式碼。

上述範例使用伺服器端執行的JavaScript來將標題縮短為10個字元，但也可以使用Java程式碼來提供完全限定的Java類別名稱。 一般而言，商業邏輯應該是以Java建立，但當元件需要從Java API提供的檢視特定變更時，使用伺服器端執行的JavaScript來進行變更會很方便。

以下各節中對此的詳細說明：

* 語句上的一節 [`data-sly-use` 說明](block-statements.md#use) ，使用該語句可以執行的所有操作。
* 「使 [用-API」頁面提供一些資訊](use-api.md) ，可協助您選擇使用Java或JavaScript編寫邏輯。
* 若要詳細說明如何編寫邏輯， [JavaScript Use-API](use-api-javascript.md) 和 [](use-api-java.md) Java Use-API頁面應有所幫助。

### 自動上下文感知逸出 {#automatic-context-aware-escaping}

請考慮下列範例：

```xml
<p data-sly-use.logic="logic.js">
    <a href="${logic.link}" title="${logic.title}">
        ${logic.text}
    </a>
</p>
```

在大部分範本語言中，此範例可能會造成跨網站指令碼(XSS)弱點，因為即使所有變數都自動以HTML逸出， `href` 屬性仍必須特別以URL逸出。 這種遺漏是最常見的錯誤之一，因為它很容易被遺忘，而且很難以自動方式發現。

為協助您，「HTML範本語言」會自動將每個變數轉義至放置變數的上下文。 這要歸功於HTL瞭解HTML語法。

假設有下列 `logic.js` 檔案：

```javascript
use(function () {
    return {
        link:  "#my link's safe",
        title: "my title's safe",
        text:  "my text's safe"
    };
});
```

初始範例將產生下列輸出：

```xml
<p>
    <a href="#my%20link%27s%20safe" title="my title&#39;s safe">
        my text&#39;s safe
    </a>
</p>
```

請注意，這兩個屬性的逸出方式不同，因為HTL知道 `href` URI上 `src` 下文必須逸出屬性和屬性。 此外，如果URI是從開 **`javascript:`**&#x200B;始的，則屬性會完全移除，除非上下文已明確變更為其他內容。

如需如何控制逸出的詳細資訊，請參閱「運算式語言顯 [示內容」一節](expression-language.md#display-context) 。

### 自動刪除空屬性 {#automatic-removal-of-empty-attributes}

請考慮下列範例：

```xml
<p class="${properties.class}">some text</p>
```

如果屬性的值 `class` 剛好空白，HTML範本語言會自動從輸出中 `class` 移除整個屬性。

同樣地，這也是可能的，因為HTL瞭解HTML語法，因此只有在屬性值不為空時，才能有條件地顯示具有動態值的屬性。 這非常方便，因為它避免在屬性周圍添加條件塊，這會導致標籤無效和不可讀。

此外，在運算式中放置的變數類型也很重要：

* **String:**
   * **非空白：** 將字串設為屬性值。
   * **空白：** 完全移除屬性。

* **編號：** 將值設定為屬性值。

* **布林函數:**
   * **true:** 顯示不帶值的屬性（作為布爾型HTML屬性）
   * **false:** 完全移除屬性。

以下是布林運算式如何允許控制布林HTML屬性的範例：

```xml
<input type="checkbox" checked="${properties.isChecked}"/>
```

對於設定屬性， [`data-sly-attribute`](block-statements.md#attribute) 該語句可能也很有用。

## 使用HTL的常見圖樣 {#common-patterns-with-htl}

本節介紹一些常見的案例，以及如何使用HTML範本語言來最佳解決它們。

### 載入客戶端庫 {#loading-client-libraries}

在HTL中，用戶端程式庫會透過AEM提供的輔助範本載入，您可透過此範本存取 [`data-sly-use`](block-statements.md#use)。 此檔案中提供3個範本，可透過下列方式呼叫 [`data-sly-call`](block-statements.md#template-call):

* **`css`** -僅載入參考用戶端程式庫的CSS檔案。
* **`js`** -僅載入參考用戶端程式庫的JavaScript檔案。
* **`all`** -載入參考用戶端程式庫的所有檔案（包括CSS和JavaScript）。

每個幫助模板都需要一 **`categories`** 個用於引用所需客戶端庫的選項。 該選項可以是字串值陣列或包含逗號分隔值清單的字串。

以下是兩個簡短的範例：

### 一次完全載入多個客戶端庫 {#loading-multiple-client-libraries-fully-at-once}

```xml
<sly data-sly-use.clientlib="/libs/granite/sightly/templates/clientlib.html"
     data-sly-call="${clientlib.all @ categories=['myCategory1', 'myCategory2']}"/>
```

### 在頁面的不同區段中參照用戶端程式庫 {#referencing-a-client-library-in-different-sections-of-a-page}

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

在上述第二個範例中，如果HTML和 **`head`****`body`** elements被置入不同的檔案，則 **`clientlib.html`** 範本必須載入每個需要的檔案中。

範本與呼叫陳述 [式的章節](block-statements.md#template-call) ，提供了有關宣告與呼叫這些範本如何運作的詳細資訊。

### 將資料傳遞至用戶端 {#passing-data-to-the-client}

一般而言，將資料傳遞給用戶端的最佳且最優雅的方式，是使用資料屬性。

以下範例說明如何使用邏輯（也可以以Java編寫）來非常方便地序列化要傳遞至用戶端的物件，然後將它放入資料屬性中：

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

從這裡，您可以輕鬆想像用戶端JavaScript如何存取該屬性並重新剖析JSON。 例如，這會是要放入用戶端程式庫的對應JavaScript:

```javascript
var elements = document.querySelectorAll("[data-json]");
for (var i = 0; i < elements.length; i++) {
    var obj = JSON.parse(elements[i].dataset.json);
    //console.log(obj);
}
```

### 使用用戶端範本 {#working-with-client-side-templates}

一個特殊情況是，在 [Lifting Limitations of Special Contexts](getting-started.md#lifting-limitations-of-special-contexts) （特殊內容的提升限制）一節中說明的技術可合法使用，即編寫位於指令碼元素中的用戶端範本（例如Handlebars） **** 。 此技巧之所以可安全使用，是因為指令 **碼元素** ，並未如假設般包含JavaScript，而是其他HTML元素。 以下是如何運作的範例：

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

如上所示，將包含在元素中的標籤可以包含 **`script`** HTL塊語句，而表達式不需要提供顯式上下文，因為Handlebars模板的內容已隔離在其自己的檔案中。 此外，此範例說明伺服器端執行的HTL(如元素上 **`h2`** )如何與用戶端執行的範本語言（如Handlebars）混 **`h3`** 合（如元素上所示）。

不過，更現代的技巧是改用HTML元素，因為這樣的好處是，它不需要將範本的內容隔離為個別的檔案。 **`template`**

**閱讀下一節內容:**

* [運算式語言](expression-language.md) -詳細瞭解在HTL運算式中可執行的動作。
* [塊語句](block-statements.md) -發現HTL中所有可用的塊語句，以及如何使用它們。
