---
title: HTL 運算式語言
description: 了解如何在 AEM 中使用 HTL 運算式語言。 HTML 範本語言會使用運算式語言來存取資料結構，以提供 HTML 輸出的動態元素。
exl-id: 57e3961b-8c84-4d56-a049-597c7b277448
source-git-commit: 7b53eff0652f650ffb8caae0e69aa349b5c548eb
workflow-type: ht
source-wordcount: '1860'
ht-degree: 100%

---

# HTL 運算式語言 {#htl-expression-language}

HTML 範本語言會使用運算式語言來存取資料結構，以提供 HTML 輸出的動態元素。 這些運算式會以 `${` 和 `}` 字元分隔。 為了避免格式錯誤的 HTML，只能在屬性值、元素內容或註解中使用運算式。

```xml
<!-- ${component.path} -->
<h1 class="${component.name}">
    ${properties.jcr:title}
</h1>
```

可在運算式前面加上 `\` 字元來逸出運算式，例如，`\${test}` 將會呈現 `${test}`。

>[!NOTE]
>
>若要嘗試此頁提供的範例，可以使用稱為[「讀取、求值、輸出」迴圈](https://github.com/Adobe-Marketing-Cloud/aem-sightly-repl)的即時執行環境。

運算式語法包括[變數](#variables)、[常值](#literals)、[運算子](#operators)和[選項](#options)：

## 變數 {#variables}

變數是存放資料值或物件的容器。 變數的名稱稱為識別碼。

HTL 讓您不需要指定任何內容，在加入 `global.jsp` 後即可讓您存取 JSP 中提供的所有常用物件。 [全域物件](global-objects.md)頁面提供了 HTL 提供存取權的所有物件清單。

### 屬性存取 {#property-access}

有兩種方法可以存取變數的屬性：使用點標記法或方括弧標記法：

```xml
${currentPage.title}  
${currentPage['title']} or ${currentPage["title"]}
```

大多數情況下應該比較適合使用較簡單的點標記法，方括弧標記法則應該用來存取包含無效識別碼字元的屬性，或是以動態方式存取屬性。 以下兩章將詳細介紹這兩種情況。

存取的屬性可以是函數，但不支援傳遞引數，所以只能存取不需要引數的函數，例如 getter。 這是我們期望的限制，其用意是為了減少內嵌在運算式中的邏輯數量。 如有需要，可使用 [`data-sly-use`](block-statements.md#use) 陳述式將參數傳遞給邏輯。

上面的範例中也顯示，無需在前面加上 `get`，將隨後的字元變成小寫即可存取 Java getter 函數 (例如 `getTitle()`)。

### 有效的識別碼字元 {#valid-identifier-characters}

稱為識別碼的變數名稱需遵守某些規則。 其開頭必須為字母 (`A`-`Z` 和 `a`-`z`) 或底線 (`_`)，隨後的字元也可以是數字 (`0`-`9`) 或冒號 (`:`)。 識別碼中不能使用 Unicode 字元 (例如 `å` 和 `ü`)。

由於冒號 (`:`) 字元在 AEM 屬性名稱中很常見，應強調它是方便使用的有效識別碼字元：

`${properties.jcr:title}`

方括弧標記法可用來存取包含無效識別碼字元的屬性，像是以下範例中的空格字元：

`${properties['my property']}`

### 以動態方式存取成員 {#accessing-members-dynamically}

```xml
${properties[myVar]}
```

### 允許處理 Null 值 {#permissive-handling-of-null-values}

```xml
${currentPage.lastModified.time.toString}
```

## 常值 {#literals}

常值是用來表示固定值的一種標記法。

### 布林值 {#boolean}

布林值代表邏輯實體，而且可以有兩個值：`true` 和 `false`。

`${true} ${false}`

### 數字 {#numbers}

數字類型只有一種：正整數。 雖然變數支援其他數字格式 (像是浮點數)，但這些格式不能表示為常值。

`${42}`

### 字串 {#strings}

字串代表文字資料，可以用單引號或雙引號括住：

`${'foo'} ${"bar"}`

除了一般字元以外，也可以使用以下特殊字元：

* `\\` 反斜線字元
* `\'` 單引號 (或縮寫符號)
* `\"` 雙引號
* `\t` 定位字元
* `\n` 換行字元
* `\r` 歸位字元
* `\f` 換頁字元
* `\b` 退格字元
* `\uXXXX` 由四個十六進位數字 XXXX 所指定的 Unicode 字元。\
   一些實用的 Unicode 逸出序列包括：

   * `\u0022` 代表 `"`
   * `\u0027` 代表 `'`

如果是上面未列出的字元，在字元前面加上反斜線字元將會顯示錯誤。

以下是如何使用字串逸出的幾個範例：

```xml
<p>${'it\'s great, she said "yes!"'}</p>
<p title="${'it\'s great, she said \u0022yes!\u0022'}">...</p>
```

這會產生以下輸出，因為 HTL 將套用上下文特有的逸出：

```xml
<p>it&#39;s great, she said &#34;yes!&#34;</p>
<p title="it&#39;s great, she said &#34;yes!&#34;">...</p>
```

### 陣列 {#arrays}

陣列是一組經過排序的值，可以透過名稱和索引來參照。 可以混合其元素的類型。

```xml
${[1,2,3,4]}
${myArray[2]}
```

陣列對於提供範本中的值清單很有用。

```xml
<ul data-sly-list="${[1,2,3,4]}">
  <li>${item}</li>
</ul>
```

## 運算子 {#operators}

### 邏輯運算子 {#logical-operators}

這類運算子通常會與布林值一起使用，但就像在 JavaScript 中一樣，這類運算子實際上會傳回其中一個指定的運算元的值，所以在與非布林值一起使用時，它們可能會傳回非布林值。

如果某個值可以轉換為 `true`，該值就是所謂的「為真」。 如果某個值可以轉換為 `false`，該值就是所謂的「為假」。 可轉換為 `false` 的值包含未定義的變數、null 值、數字零和空字串。

#### 邏輯 NOT {#logical-not}

`${!myVar}` 會在其單一運算元可以轉換為 `true` 的情況下傳回 `false`，否則會傳回 `true`。

例如，這可用來反轉測試條件，像是只有在沒有子頁面的情況下才顯示某個元素：

```xml
<p data-sly-test="${!currentPage.hasChild}">current page has no children</p>
```

#### 邏輯 AND {#logical-and}

`${varOne && varTwo}` 如果「為假」則會傳回 `varOne`，否則會傳回 `varTwo`。

此運算子可用於一次測試兩個條件，像是驗證兩個屬性是否存在：

```xml
<div data-sly-test="${properties.jcr:title && properties.jcr:description}">
    <h1>${properties.jcr:title}</h1>
    <p>${properties.jcr:description}</p>
</div>
```

邏輯 AND 運算子也可用於有條件地顯示 HTML 屬性，因為如果屬性的動態設定的值運算結果為 false 或空字串，HTL 就會移除這些屬性。 所以在以下範例中，只有當 `logic.showClass`「為真」，而且 `logic.className` 存在且不是空白時，才會顯示 `class` 屬性：

```xml
<div class="${logic.showClass && logic.className}">...</div>
```

#### 邏輯 OR {#logical-or}

`${varOne || varTwo}` 如果「為真」則會傳回 `varOne`，否則會傳回 `varTwo`。

此運算子可用於測試是否會套用兩個條件的其中一個，像是驗證是否至少有一個屬性存在：

```xml
<div data-sly-test="${properties.jcr:title || properties.jcr:description}">...</div>
```

當邏輯 OR 運算子傳回第一個「為真」的變數時，它也可以方便地用來提供遞補值。

它也可用於有條件地顯示 HTML 屬性，因為如果屬性的值是由運算結果為 false 或空字串的運算式所設定，HTL 就會移除這些屬性。 所以以下範例會在 **`properties.jcr:`** title 存在且不是空白時顯示它，否則會退回在 **`properties.jcr:description`** 存在且不是空白時顯示它，否則會顯示「no title or description provided」訊息：

```xml
<p>${properties.jcr:title || properties.jcr:description || "no title or description provided"}</p>
```

### 條件式 (三元) 運算子 {#conditional-ternary-operator}

如果 `varCondition`「為真」，則 `${varCondition ? varOne : varTwo}` 會傳回 `varOne`，否則會傳回 `varTwo`。

此運算子通常可用來定義運算式內的條件，像是根據頁面的狀態顯示不同訊息：

```xml
<p>${currentPage.isLocked ? "page is locked" : "page can be edited"}</p>
```

>[!TIP]
>
>由於識別碼中也允許使用冒號字元，所以最好以空格分隔三元運算子，好讓剖析器可以清楚地了解：

```xml
<p>${properties.showDescription ? properties.jcr:description : properties.jcr:title}</p>
```

### 比較運算子 {#comparison-operators}

等式和不等式運算子僅支援相同類型的運算元。 當類型不符時，就會顯示錯誤。

* 當字串擁有相同字元序列時，就表示這些字串相同。
* 當數字擁有相同值時，就表示這些數字相同。
* 當兩個布林值皆為 `true` 或 `false` 時，就表示兩者相同。
* Null 或未定義的變數與其自身和彼此都相同。

如果 `varOne` 和 `varTwo` 相同，則 `${varOne == varTwo}` 會傳回 `true`。

如果 `varOne` 和 `varTwo` 不相同，則 `${varOne != varTwo}` 會傳回 `true`。

關係運算子僅支援數字運算元。 所有其他類型都會顯示錯誤。

如果 `varOne` 大於 `varTwo`，則 `${varOne > varTwo}` 會傳回 `true`。

如果 `varOne` 小於 `varTwo`，則 `${varOne < varTwo}` 會傳回 `true`。

如果 `varOne` 大於或等於 `varTwo`，則 `${varOne >= varTwo}` 會傳回 `true`。

如果 `varOne` 小於或等於 `varTwo`，則 `${varOne <= varTwo}` 會傳回 `true`。

### 群組括號 {#grouping-parentheses}

群組運算子 `()` 可控制運算式中運算的優先權。

`${varOne && (varTwo || varThree)}`

## 選項 {#options}

運算式選項可在運算式上發揮作用並加以修改，或是在結合區塊陳述式使用時當作參數。

`@` 後面的所有內容都是選項：

```xml
${myVar @ optOne}
```

選項可以有值，值可能是變數或常值：

```xml
${myVar @ optOne=someVar}
${myVar @ optOne='bar'}
${myVar @ optOne=10}
${myVar @ optOne=true}
```

多個選項會用逗號分隔：

```xml
${myVar @ optOne, optTwo=bar}
```

也可以使用只包含選項的參數運算式：

```xml
${@ optOne, optTwo=bar}
```

### 字串格式 {#string-formatting}

將列舉的預留位置 {*n*} 替換為對應變數的選項：

```xml
${'Page {0} of {1}' @ format=[current, total]}
```

## URL 操控 {#url-manipulation}

可使用一組新的 url 操控。

請參閱下列範例以了解其使用方式：

在路徑中新增 html 副檔名。

```xml
<a href="${item.path @ extension = 'html'}">${item.name}</a>
```

在路徑中新增 html 副檔名和選擇器。

```xml
<a href="${item.path @ extension = 'html', selectors='products'}">${item.name}</a>
```

在路徑中新增 html 副檔名和片段 (#value)。

```xml
<a href="${item.path @ extension = 'html', fragment=item.name}">${item.name}</a>
```

`@extension` 適用於所有情境，用來檢查是否要新增副檔名。

```xml
${ link @ extension = 'html' }
```

### 數字/日期格式 {#number-date-formatting}

HTL 允許使用數字和日期的原生格式，而不用撰寫自訂程式碼。 這也支援時區和地區設定。

以下範例說明先指定格式，然後指定需要格式的值：

```xml
<h2>${ 'dd-MMMM-yyyy hh:mm:ss' @
           format=currentPage.lastModified,
           timezone='PST',
           locale='fr'}</h2>

<h2>${ '#.00' @ format=300}</h2>
```

>[!NOTE]
>
>如需您可以使用的格式的完整細節，請參閱 [HTL 規格](https://github.com/Adobe-Marketing-Cloud/htl-spec/blob/master/SPECIFICATION.md)。

### 國際化 {#internationalization}

使用目前的[字典](https://experienceleague.adobe.com/docs/experience-manager-65/developing/components/internationalization/i18n-translator.html?lang=zh-Hant)將字串翻譯成目前的&#x200B;*來源*&#x200B;語言 (請參閱底下)。 如果找不到翻譯，則會使用原始字串。

```xml
${'Page' @ i18n}
```

hint 選項可用來向譯者提供註解，指定使用該文字的上下文：

```xml
${'Page' @ i18n, hint='Translation Hint'}
```

語言的預設來源為 `resource`，這表示文字會被翻譯成與內容相同的語言。 這可以變更為 `user`，這意味著語言是取自瀏覽器地區設定或登入的使用者的地區設定：

```xml
${'Page' @ i18n, source='user'}
```

提供明確地區設定會覆寫來源設定：

```xml
${'Page' @ i18n, locale='en-US'}
```

若要將變數內嵌到翻譯字串中，可以使用 format 選項：

```xml
${'Page {0} of {1}' @ i18n, format=[current, total]}
```

### 陣列 join {#array-join}

根據預設，將陣列顯示為文字時，HTL 會顯示用逗號分隔的值 (沒有空格)。

使用 join 選項可指定不同分隔符號：

```xml
${['one', 'two'] @ join='; '}
```

### 顯示上下文 {#display-context}

HTL 運算式的顯示上下文指的是它在 HTML 頁面結構中的位置。 例如，如果運算式出現在呈現後會產生文字節點的位置，則稱它位在 `text` 上下文中。 如果運算式位在屬性值內，則稱它位在 `attribute` 上下文中，依此類推。

除了指令碼 (JS) 和樣式 (CSS) 上下文以外，HTL 會自動偵測運算式的上下文並適當地將其逸出，以防止 XSS 安全問題發生。 在指令碼和 CSS 的情況下，必須明確設定所需的上下文行為。 此外，在需要覆寫自動行為的其他任何情況下，也需要明確設定上下文行為。

在這裡，我們在三個不同上下文中有三個變數：

* `properties.link` (`uri` 上下文)
* `properties.title` (`attribute` 上下文)
* `properties.text` (`text` 上下文)

HTL 將會根據其各自上下文的安全需求，以不同方式逸出每一個。 在正常情況下不需要明確設定上下文，例如以下情況：

```xml
<a href="${properties.link}" title="${properties.title}">${properties.text}</a>
```

若要安全地輸出標記 (也就是運算式本身的運算結果為 HTML 時)，將會使用 `html` 上下文：

```xml
<div>${properties.richText @ context='html'}</div>
```

必須為樣式上下文設定明確的上下文：

```xml
<span style="color: ${properties.color @ context='styleToken'};">...</span>
```

必須為指令碼上下文設定明確的上下文：

```xml
<span onclick="${properties.function @ context='scriptToken'}();">...</span>
```

也可以關閉逸出和 XSS 保護：

```xml
<div>${myScript @ context='unsafe'}</div>
```

### 上下文設定 {#context-settings}

| 上下文 | 使用時機 | 作用 |
|--- |--- |--- |
| `text` | 元素內內容的預設 | 編碼所有 HTML 特殊字元。 |
| `html` | 若要安全地輸出標記 | 篩選 HTML 以符合 AntiSamy 政策規定，移除不符合規定的內容。 |
| `attribute` | 屬性值的預設 | 編碼所有 HTML 特殊字元。 |
| `uri` | 顯示連結和路徑，href 和 src 屬性值的預設 | 驗證 URI 是否寫為 href 或 src 屬性值，驗證失敗時不會輸出任何內容。 |
| `number` | 顯示數字 | 驗證 URI 是否包含整數，驗證失敗時會輸出零。 |
| `attributeName` | 在設定屬性名稱時，為 data-sly-attribute 的預設 | 驗證屬性名稱，驗證失敗時不會輸出任何內容。 |
| `elementName` | data-sly-element 的預設 | 驗證元素名稱，驗證失敗時不會輸出任何內容。 |
| `scriptToken` | 用於 JS 識別碼、常值數字或常值字串 | 驗證 JavaScript 權杖，驗證失敗時不會輸出任何內容。 |
| `scriptString` | 在 JS 字串內 | 針對會脫離字串的字元進行編碼。 |
| `scriptComment` | 在 JS 註解內 | 驗證 JavaScript 註解，驗證失敗時不會輸出任何內容。 |
| `styleToken` | 用於 CSS 識別碼、數字、維度、字串、十六進位色彩或函數。 | 驗證 CSS 權杖，驗證失敗時不會輸出任何內容。 |
| `styleString` | 在 CSS 字串內 | 針對會脫離字串的字元進行編碼。 |
| `styleComment` | 在 CSS 註解內 | 驗證 CSS 註解，驗證失敗時不會輸出任何內容。 |
| `unsafe` | 只有當上述任何項目都無法執行工作時 | 完全停用逸出和 XSS 保護。 |
