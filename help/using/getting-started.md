---
title: 開始使用 HTL
description: AEM支援的HTL取代JSP，成為AEM中HTML偏好且建議的伺服器端範本系統。
exl-id: c95eb1b3-3b96-4727-8f4f-d54e7136a8f9
source-git-commit: 8e70ee4921a7ea071ab7e06947824c371f4013d8
workflow-type: tm+mt
source-wordcount: '2471'
ht-degree: 0%

---

# 開始使用 HTL {#getting-started-with-htl}

Adobe Experience Manager(AEM)支援的HTML範本語言(HTL)是AEM中HTML偏好且建議的伺服器端範本系統。 它取代了舊版AEM中使用的JSP(JavaServer Pages)。

>[!NOTE]
>
>要運行此頁上提供的大多數示例，可以使用名為[讀取Eval Print Loop](https://github.com/Adobe-Marketing-Cloud/aem-htl-repl)的即時執行環境。

## JSP上的HTL {#htl-over-jsp}

建議新AEM專案使用HTML範本語言，因為與JSP相比，它提供多項優點。 但是，對於現有項目，只有在預計未來幾年，遷移所花費的精力比維護現有JSP要少時才有意義。

但改用HTL並非一無是處的選擇，因為以HTL撰寫的元件與以JSP或ESP撰寫的元件相容。 這表示現有專案可順利將HTL用於新元件，同時保留現有元件的JSP。

即使在同一個元件中，HTL檔案也可與JSP和ESP搭配使用。 以下範例說明&#x200B;**第1行**&#x200B;如何從JSP檔案納入HTL檔案，以及如何從HTL檔案納入JSP檔案&#x200B;**第2行**:

```xml
<cq:include script="template.html"/>
<sly data-sly-include="template.jsp"/>
```

### 常見問題{#frequently-asked-questions}

開始使用HTML範本語言之前，請先回答與JSP與HTL主題相關的幾個問題。

**HTL是否有JSP沒有的限制？** - HTL與JSP相比並無其他限制，因為使用JSP可執行的操作也應可透過HTL達成。然而，HTL的設計在幾個方面比JSP更嚴格，而所有可在單一JSP檔案中達成的目標，可能需要分成Java類別或JavaScript檔案，才能在HTL中實現。 但這通常是為了確保邏輯與標籤之間的關注之間良好地分離。

**HTL是否支援JSP標籤程式庫？**  — 否，但如載入用戶端程 [式庫區](getting-started.md#loading-client-libraries) 段所示，範本 [和callstatement](block-statements.md#template-call) 提供類似的模式。

**HTL功能可在AEM專案上擴充嗎？**  — 不，他們不能。HTL具有強大的擴充功能機制，可重複使用邏輯 — [Use-API](getting-started.md#use-api-for-accessing-logic) — 和標籤（[範本和呼叫](block-statements.md#template-call)陳述式），這些陳述式可用來模組化專案的程式碼。

**HTL優於JSP的主要優點為何？**  — 安全性和項目效率是主要優勢，詳見 [概述](overview.md)。

**JSP最終會消失嗎？**  — 在當前日期，沒有按這些方式計畫。

## HTL {#fundamental-concepts-of-htl}的基本概念

HTML模板語言使用表達式語言將內容片段插入到呈現的標籤中，而HTML5資料屬性用於定義標籤塊（如條件或迭代）上的語句。 當HTL編譯到Java Servlet中時，運算式和HTL資料屬性都會在伺服器端進行評估，因此產生的HTML中不會有任何項目顯示。

### 塊和表達式{#blocks-and-expressions}

以下是第一個範例，可如&#x200B;**`template.html`**&#x200B;檔案中所述包含：

```xml
<h1 data-sly-test="${properties.jcr:title}">
    ${properties.jcr:title}
</h1>
```

可以區分兩種不同的語法：

* **[塊語句](block-statements.md)**  — 要有條件地顯示  **&lt;h1>** 元素，則 [`data-sly-test`](block-statements.md#test) 會使用HTML5資料屬性。HTL提供多個此類屬性，可將行為附加至任何HTML元素，且所有屬性都會加上前置詞`data-sly`。

* **[運算式語言](expression-language.md)**  - HTL運算式由字元和 `${` 分隔 `}`。在執行階段中，會評估這些運算式，並將其值插入傳出的HTML資料流中。

以上連結的兩個頁面提供可用於語法的詳細功能清單。

### SLY元素{#the-sly-element}

HTL的核心概念是可重複使用現有的HTML元素來定義區塊陳述式，如此可避免插入其他分隔字元來定義陳述式的開始和結束位置。 此標籤的不明確註解可將靜態HTML轉換為正常運作的動態範本，可讓您不破壞HTML程式碼的有效性，因此即使是靜態檔案，仍可正常顯示。

但有時候，必須插入block語句的確切位置可能沒有現有元素。 對於這種情況，可以插入將自動從輸出中移除的特殊SLY元素，同時執行附加的塊語句並相應地顯示其內容。

下面的例子：

```xml
<sly data-sly-test="${properties.jcr:title && properties.jcr:description}">
    <h1>${properties.jcr:title}</h1>
    <p>${properties.jcr:description}</p>
</sly>
```

會輸出類似下列HTML的內容，但僅限於已定義&#x200B;**`jcr:title`**&#x200B;和&#x200B;**`jcr:description`**&#x200B;屬性，且兩者皆非空白時：

```xml
<h1>MY TITLE</h1>
<p>MY DESCRIPTION</p>
```

請記住，只有當沒有任何現有元素可以以block陳述式加上註解時，才使用SLY元素，因為SLY元素會抑制語言提供的值，使其動態時不會變更靜態HTML。

例如，如果先前的範例已包裝在DIV元素內，則新增的SLY元素將具有辱罵性：

```xml
<div>
    <sly data-sly-test="${properties.jcr:title && properties.jcr:description}">
        <h1>${properties.jcr:title}</h1>
        <p>${properties.jcr:description}</p>
    </sly>
</div>
```

而DIV元素本可以用條件加上註解：

```xml
<div data-sly-test="${properties.jcr:title && properties.jcr:description}">
    <h1>${properties.jcr:title}</h1>
    <p>${properties.jcr:description}</p>
</div>
```

### HTL註解{#htl-comments}

下列範例顯示在&#x200B;**第1**&#x200B;行的HTL註解，以及在&#x200B;**第2**&#x200B;行的HTML註解：

```xml
<!--/* An HTL Comment */-->
<!-- An HTML Comment -->
```

HTL註解是HTML註解，其語法與JavaScript類似。 處理器會完全忽略整個HTL註解，並從輸出中移除。

不過，標準HTML註解的內容會傳遞，且會評估註解中的運算式。

HTML註解不能包含HTL註解，反之亦然。

### 特殊內容{#special-contexts}

為了能充分運用HTL，請務必充分了解其以HTML語法為基礎的後果。

### 元素和屬性名稱{#element-and-attribute-names}

運算式只能放入HTML文字或屬性值中，但不能放在元素名稱或屬性名稱內，否則將不再是有效的HTML。 為了動態設定元素名稱，[`data-sly-element`](block-statements.md#element)語句可用於所需元素，並且用於動態設定屬性名稱，即使一次設定多個屬性，[`data-sly-attribute`](block-statements.md#attribute)語句也可以使用。

```xml
<h1 data-sly-element="${myElementName}" data-sly-attribute="${myAttributeMap}">...</h1>
```

### 沒有塊語句的上下文{#contexts-without-block-statements}

由於HTL使用資料屬性來定義區塊陳述式，因此無法在下列內容中定義這類區塊陳述式，因此只能在該處使用運算式：

* HTML註解
* 指令碼元素
* 樣式元素

原因是這些內容的內容是文字，而非HTML，而且包含的HTML元素會視為簡單的字元資料。 因此，若沒有真正的HTML元素，也無法執行&#x200B;**`data-sly`**&#x200B;屬性。

這聽起來可能像是大限制，不過這是需要的限制，因為不應濫用HTML範本語言來產生非HTML的輸出。 以下[存取邏輯的Use-API](getting-started.md#use-api-for-accessing-logic)一節介紹如何從範本呼叫其他邏輯，以便在需要準備這些內容的複雜輸出時使用。 例如，從後端傳送資料至前端指令碼的簡單方式，是要有元件的邏輯以產生JSON字串，接著再以簡單的HTL運算式將字串放在資料屬性中。

以下範例說明HTML註解的行為，但在指令碼或樣式元素中，會觀察到相同的行為：

```xml
<!--
    The title is: ${properties.jcr:title}
    <h1 data-sly-test="${properties.jcr:title}">${properties.jcr:title}</h1>
-->
```

會輸出類似下列HTML的內容：

```xml
<!--
    The title is: MY TITLE
    <h1 data-sly-test="MY TITLE">MY TITLE</h1>
-->
```

### 需要的顯式上下文{#explicit-contexts-required}

如下方的[自動內容感知逸出](getting-started.md#automatic-context-aware-escaping)一節所述，HTL的一個目標是透過自動將內容感知逸出套用至所有運算式，以降低引入跨網站指令碼(XSS)漏洞的風險。 雖然HTL可自動偵測放置於HTML標籤內的運算式內容，但不會分析內嵌JavaScript或CSS的語法，因此需仰賴開發人員明確指定必須套用至這類運算式的確切內容。

由於未套用正確的逸出結果會導致XSS弱點，因此當內容未宣告時，HTL會移除指令碼和樣式內容中所有運算式的輸出。

以下是如何為置於指令碼和樣式內的運算式設定內容的範例：

```xml
<script> var trackingID = "${myTrackingID @ context='scriptString'}"; </script>
<style> a { font-family: "${myFont @ context='styleString'}"; } </style>
```

如需如何控制逸出的詳細資訊，請參閱[運算式語言顯示內容](expression-language.md#display-context)區段。

### 特殊上下文的提升限制{#lifting-limitations-of-special-contexts}

在需要略過指令碼、樣式和註解內容限制的特殊情況下，可將其內容隔離在個別的HTL檔案中。 位於其專屬檔案中的所有項目，都會被HTL解譯為一般HTML片段，而忘記可能已納入的限制內容。

如需範例，請參閱深入下文的[使用用戶端範本](getting-started.md#working-with-client-side-templates)區段。

>[!CAUTION]
>
>此技術可能會引入跨網站指令碼(XSS)漏洞，若使用此漏洞，應謹慎研究安全性。 通常有比依賴這種做法更好的方法來實施相同的事物。

## HTL {#general-capabilities-of-htl}的一般功能

本節會快速說明HTML範本語言的一般功能。

### 用於訪問邏輯{#use-api-for-accessing-logic}的Use-API

請考量下列範例：

```xml
<p data-sly-use.logic="logic.js">${logic.title}</p>
```

並依照放在旁邊的`logic.js`伺服器端執行的JavaScript檔案：

```javascript
use(function () {
    return {
        title: currentPage.getTitle().substring(0, 10) + "..."
    };
});
```

由於HTML範本語言不允許在標籤內混用程式碼，因此它提供Use-API擴充功能機制，以便從範本輕鬆執行程式碼。

上述範例使用伺服器端執行的JavaScript來將標題縮短為10個字元，但透過提供完全限定的Java類別名稱，也可能使用Java程式碼來執行相同操作。 一般而言，商業邏輯應該建置在Java中，但若元件需要從Java API提供的內容進行某些檢視專屬變更，則使用某些伺服器端執行的JavaScript來執行這項作業會相當方便。

以下各節將詳細說明此事項：

* [`data-sly-use`語句](block-statements.md#use)上的部分說明了可以使用該語句完成的所有操作。
* [Use-API頁面](use-api.md)提供一些資訊，以協助您在以Java或JavaScript撰寫邏輯之間進行選擇。
* 若要詳細說明如何撰寫邏輯，[JavaScript Use-API](use-api-javascript.md)和[Java Use-API](use-api-java.md)頁面應該會有所幫助。

### 自動內容感知逸出{#automatic-context-aware-escaping}

請考量下列範例：

```xml
<p data-sly-use.logic="logic.js">
    <a href="${logic.link}" title="${logic.title}">
        ${logic.text}
    </a>
</p>
```

在大多數範本語言中，此範例可能會造成跨網站指令碼(XSS)漏洞，因為即使所有變數都自動以HTML逸出，`href`屬性仍必須明確以URL逸出。 這種遺漏是最常見的錯誤之一，因為很容易被遺忘，而且很難以自動的方式識別。

為了提供這些協助，HTML範本語言會自動將每個變數逸出至放置變數的內容。 這要歸功於HTL了解HTML的語法。

假設如下`logic.js`檔案：

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

請注意，兩個屬性的逸出方式不同，因為HTL知道`href`和`src`屬性必須針對URI內容逸出。 此外，如果URI以&#x200B;**`javascript:`**&#x200B;開頭，則該屬性將被完全刪除，除非上下文被顯式更改為其他內容。

如需如何控制逸出的詳細資訊，請參閱[運算式語言顯示內容](expression-language.md#display-context)區段。

### 自動刪除空屬性{#automatic-removal-of-empty-attributes}

請考量下列範例：

```xml
<p class="${properties.class}">some text</p>
```

如果`class`屬性的值恰好為空，則HTML模板語言將自動從輸出中刪除整個`class`屬性。

同樣地，這也是可能的，因為HTL了解HTML語法，因此只有在屬性值不為空時，才能有條件地顯示具有動態值的屬性。 這非常方便，因為它避免在屬性周圍添加條件塊，這會導致標籤無效且不可讀。

此外，運算式中放置的變數類型也很重要：

* **String:**
   * **非空白：** 將字串設為屬性值。
   * **empty:** 完全移除屬性。

* **數字：** 將值設為屬性值。

* **布林函數:**
   * **true:** 顯示不含值的屬性（作為布林HTML屬性）
   * **false:** 完全移除屬性。

以下是布林運算式如何允許控制布林HTML屬性的範例：

```xml
<input type="checkbox" checked="${properties.isChecked}"/>
```

對於設定屬性，[`data-sly-attribute`](block-statements.md#attribute)語句可能也很有用。

## 具有HTL {#common-patterns-with-htl}的常見模式

本節介紹一些常見案例，以及如何以HTML範本語言最佳解決這些案例。

### 載入客戶端庫{#loading-client-libraries}

在HTL中，用戶端程式庫是透過AEM提供的協助範本載入，可透過[`data-sly-use`](block-statements.md#use)存取。 此檔案中有三個模板，可通過[`data-sly-call`](block-statements.md#template-call)調用：

* **`css`**  — 僅載入所參考用戶端程式庫的CSS檔案。
* **`js`**  — 僅載入所參考用戶端程式庫的JavaScript檔案。
* **`all`**  — 載入所引用客戶端庫的所有檔案（包括CSS和JavaScript）。

每個幫助程式模板都需要一個&#x200B;**`categories`**&#x200B;選項來引用所需的客戶端庫。 該選項可以是字串值的陣列，或包含逗號分隔值清單的字串。

以下是兩個簡短範例：

### 一次完全載入多個客戶端庫{#loading-multiple-client-libraries-fully-at-once}

```xml
<sly data-sly-use.clientlib="/libs/granite/sightly/templates/clientlib.html"
     data-sly-call="${clientlib.all @ categories=['myCategory1', 'myCategory2']}"/>
```

### 在頁面{#referencing-a-client-library-in-different-sections-of-a-page}的不同區段中參考用戶端程式庫

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

在上述第二個範例中，若將HTML **`head`**&#x200B;和&#x200B;**`body`**&#x200B;元素放置到不同的檔案中，則必須在需要的每個檔案中載入&#x200B;**`clientlib.html`**&#x200B;範本。

[template &amp; call](block-statements.md#template-call)語句上的部分提供了有關聲明和調用此類模板如何工作的更多詳細資訊。

### 將資料傳遞到客戶端{#passing-data-to-the-client}

一般而言，將資料傳遞至用戶端（但透過HTL，更是如此）的最佳且最優雅的方式，是使用資料屬性。

以下範例說明如何使用邏輯（也可以以Java寫入）來非常方便地將要傳遞至用戶端的物件序列化為JSON，接著便可輕鬆將物件放入資料屬性中：

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

從這裡，您可以輕鬆想像用戶端JavaScript如何存取該屬性並再次剖析JSON。 例如，這會是要放入用戶端程式庫的對應JavaScript:

```javascript
var elements = document.querySelectorAll("[data-json]");
for (var i = 0; i < elements.length; i++) {
    var obj = JSON.parse(elements[i].dataset.json);
    //console.log(obj);
}
```

### 使用用戶端範本{#working-with-client-side-templates}

在&#x200B;**指令碼**&#x200B;元素中，[特殊內容提升限制](getting-started.md#lifting-limitations-of-special-contexts)一節中解釋的技術可以合法使用的一個特殊情況是寫入客戶端模板（例如Handlebars）。 在這種情況下，此技術可以安全地使用的原因是，**script**&#x200B;元素接著不包含假設的JavaScript，而是其他HTML元素。 以下是運作方式的範例：

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

如上所示，將包含在&#x200B;**`script`**&#x200B;元素中的標籤可以包含HTL區塊陳述式，且運算式不需要提供明確內容，因為Handlebars範本的內容已隔離在其自己的檔案中。 此外，此範例說明如何將伺服器端執行的HTL（如&#x200B;**`h2`**&#x200B;元素上）與用戶端執行的範本語言（如Handlebars）混合（如&#x200B;**`h3`**&#x200B;元素上所示）。

不過，更現代的技術將是改用HTML **`template`**&#x200B;元素，因為這樣的好處就是不需要將範本的內容隔離為個別檔案。

**閱讀下一節內容:**

* [運算式語言](expression-language.md)  — 詳細了解HTL運算式中可執行的操作。
* [區塊陳述式](block-statements.md)  — 探索HTL中可用的所有區塊陳述式，以及如何使用這些陳述式。
