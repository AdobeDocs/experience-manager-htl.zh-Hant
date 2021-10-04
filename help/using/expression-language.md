---
title: HTL 運算式語言
description: HTML範本語言使用運算式語言來存取提供HTML輸出之動態元素的資料結構。
exl-id: 57e3961b-8c84-4d56-a049-597c7b277448
source-git-commit: 89b9e89254f341e74f1a5a7b99735d2e69c8a91e
workflow-type: tm+mt
source-wordcount: '1852'
ht-degree: 0%

---

# HTL 運算式語言 {#htl-expression-language}

HTML範本語言使用運算式語言來存取提供HTML輸出之動態元素的資料結構。 這些運算式由字元`${`和`}`分隔。 為避免格式錯誤的HTML，運算式只能用於屬性值、元素內容或註解中。

```xml
<!-- ${component.path} -->
<h1 class="${component.name}">
    ${properties.jcr:title}
</h1>
```

可以通過預置`\`字元來轉義表達式，例如，將呈現`\${test}`。`${test}`

>[!NOTE]
>
>若要試用本頁上提供的示例，可以使用名為[讀取Eval Print Loop](https://github.com/Adobe-Marketing-Cloud/aem-sightly-repl)的即時執行環境。

運算式語法包含[variables](#variables)、[literals](#literals)、[operators](#operators)和[options](#options):

## 變數 {#variables}

變數是儲存資料值或物件的容器。 變數的名稱稱為識別碼。

HTL不需指定任何項目，即可存取在包含`global.jsp`後，JSP中通常可用的所有物件。 [全域物件](global-objects.md)頁面提供HTL可存取之所有物件的清單。

### 屬性存取 {#property-access}

存取變數屬性有兩種方式，點記號或括弧記號：

```xml
${currentPage.title}  
${currentPage['title']} or ${currentPage["title"]}
```

大多數情況下，最好使用更簡單的點記號，而括弧記號應用於存取包含無效識別碼字元的屬性，或動態存取屬性。 以下兩章將提供這兩種情況的詳細資訊。

所存取的屬性可以是函式，但不支援傳遞引數，因此只有不預期引數的函式（如getter）才可以存取。 這是一個需要的限制，其目的是減少內嵌在運算式中的邏輯量。 如有需要，可使用[`data-sly-use`](block-statements.md#use)陳述式將參數傳遞至邏輯。

上例中還顯示了Java getter函式（如`getTitle()`）可以訪問，而不需要預置`get`，也可以通過降低以下字元的大小寫來訪問。

### 有效的標識符字元 {#valid-identifier-characters}

變數的名稱（稱為識別碼）符合特定規則。 它們必須以字母（`A`-`Z`和`a`-`z`）或底線(`_`)開頭，後續字元也可以是數字(`0`-`9`)或冒號(`:`)。 標識符中不能使用Unicode字母，如`å`和`ü`。

由於冒號(`:`)字元在AEM屬性名稱中很常見，因此應強調它是有效的識別碼字元：

`${properties.jcr:title}`

括弧標籤法可用來存取包含無效識別碼字元的屬性，如以下範例中的空格字元：

`${properties['my property']}`

### 動態訪問成員 {#accessing-members-dynamically}

```xml
${properties[myVar]}
```

### Null值的許可性處理 {#permissive-handling-of-null-values}

```xml
${currentPage.lastModified.time.toString}
```

## 文字 {#literals}

常值是表示固定值的標籤法。

### 布林值 (Boolean) {#boolean}

布林值代表邏輯實體，可以有兩個值：`true`和`false`。

`${true} ${false}`

### 數字 {#numbers}

只有一個數字類型：正整數。 雖然其他數字格式（如浮點）在變數中受支援，但無法以文字表示。

`${42}`

### 字串 {#strings}

字串代表文字資料，可以是單引號或雙引號：

`${'foo'} ${"bar"}`

除了普通字元外，還可使用下列特殊字元：

* `\\` 反斜線字元
* `\'` 單引號（或單引號）
* `\"` 雙引號
* `\t` 標籤
* `\n` 新行
* `\r` 歸位
* `\f` 表單摘要
* `\b` 空格
* `\uXXXX` 由四十六進位數字XXXX指定的Unicode字元。\
   一些有用的unicode轉義序列包括：

   * `\u0022` 的 `"`
   * `\u0027` 的 `'`

若字元未列於上方，反斜線字元的前面會顯示錯誤。

以下是如何使用字串逸出的一些範例：

```xml
<p>${'it\'s great, she said "yes!"'}</p>
<p title="${'it\'s great, she said \u0022yes!\u0022'}">...</p>
```

這會導致下列輸出，因為HTL會套用內容特定逸出：

```xml
<p>it&#39;s great, she said &#34;yes!&#34;</p>
<p title="it&#39;s great, she said &#34;yes!&#34;">...</p>
```

### 陣列 {#arrays}

陣列是一組有序的值，可以用名稱和索引引用。 其元素的類型可混合。

```xml
${[1,2,3,4]}
${myArray[2]}
```

陣列可用來從範本提供值清單。

```xml
<ul data-sly-list="${[1,2,3,4]}">
  <li>${item}</li>
</ul>
```

## 運算子 {#operators}

### 邏輯運算子 {#logical-operators}

這些運算子通常與布林值一起使用，但是，與JavaScript中一樣，它們實際上會返回指定操作數之一的值，因此當與非布林值一起使用時，它們可能返回非布林值。

如果值可轉換為`true`，則值稱為truthy。 如果值可轉換為`false`，則值稱為假。 可轉換為`false`的值是未定義的變數、空值、數字零和空字串。

#### 邏輯NOT {#logical-not}

`${!myVar}` 如果 `false` 其單個操作數可轉換為，則返回 `true`;否則，它會傳回 `true`。

例如，這可用來反轉測試條件，例如只有在沒有子頁面時才顯示元素：

```xml
<p data-sly-test="${!currentPage.hasChild}">current page has no children</p>
```

#### 邏輯和 {#logical-and}

`${varOne && varTwo}` 如果 `varOne` 是假的，則返回；否則，它會傳回 `varTwo`。

此運算子可用來一次測試兩個條件，例如驗證是否存在兩個屬性：

```xml
<div data-sly-test="${properties.jcr:title && properties.jcr:description}">
    <h1>${properties.jcr:title}</h1>
    <p>${properties.jcr:description}</p>
</div>
```

邏輯AND運算子也可用來有條件地顯示HTML屬性，因為HTL會移除具有以動態方式設定且評估為false或空字串的值的屬性。 因此，在以下範例中，`class`屬性僅在`logic.showClass`為真且`logic.className`存在且非空白時顯示：

```xml
<div class="${logic.showClass && logic.className}">...</div>
```

#### 邏輯或 {#logical-or}

`${varOne || varTwo}` 若 `varOne` 為真則傳回；否則，它會傳回 `varTwo`。

此運算子可用來測試是否適用以下兩個條件之一，例如驗證至少一個屬性的存在：

```xml
<div data-sly-test="${properties.jcr:title || properties.jcr:description}">...</div>
```

由於邏輯OR運算子會傳回第一個真的變數，因此也很方便地用來提供後援值。

HTL也可用來有條件地顯示HTML屬性，因為HTL會移除由運算式設定的值屬性，這些運算式會評估為false或空字串。 因此，如果&#x200B;**`properties.jcr:`**&#x200B;標題存在且非空白，則以下範例將顯示&#x200B;**`properties.jcr:description`**&#x200B;標題，否則若存在且非空白，則會回復為顯示，否則會顯示訊息「未提供標題或說明」：

```xml
<p>${properties.jcr:title || properties.jcr:description || "no title or description provided"}</p>
```

### 條件（三元）運算子 {#conditional-ternary-operator}

`${varCondition ? varOne : varTwo}` 若 `varOne` 為 `varCondition` truthy則傳回；否則會傳回 `varTwo`。

此運算子通常可用來定義運算式內的條件，例如根據頁面狀態顯示不同的訊息：

```xml
<p>${currentPage.isLocked ? "page is locked" : "page can be edited"}</p>
```

>[!TIP]
>
>由於識別碼中也允許冒號字元，因此最好將三值運算子與空白字元分開，以向剖析器提供清晰度：

```xml
<p>${properties.showDescription ? properties.jcr:description : properties.jcr:title}</p>
```

### 比較運算子 {#comparison-operators}

等式和不等式運算子僅支援相同類型的操作數。 類型不相符時，會顯示錯誤。

* 如果字串的字元順序相同，則字串相等。
* 數值相同時，數值相等
* 如果兩者均為`true`或兩者均為`false`，則布林值相等。
* Null或未定義的變數彼此相等。

`${varOne == varTwo}` 若 `true` 和 `varOne` 相 `varTwo` 等則傳回。

`${varOne != varTwo}` 若 `true` 和 `varOne` 不 `varTwo` 相等則傳回。

關係運算子僅支援數字操作數。 對於所有其他類型，會顯示錯誤。

`${varOne > varTwo}` 若 `true` 大 `varOne` 於，則傳 `varTwo`回。

`${varOne < varTwo}` 如果 `true` 小 `varOne` 於，則傳 `varTwo`回。

`${varOne >= varTwo}` 如 `true` 果 `varOne` 大於或等於則傳 `varTwo`回。

`${varOne <= varTwo}` 如 `true` 果 `varOne` 小於或等於，則傳 `varTwo`回。

### 分組括弧 {#grouping-parentheses}

分組運算子`()`控制表達式中評估的優先順序。

`${varOne && (varTwo || varThree)}`

## 選項 {#options}

運算式選項可對運算式執行並加以修改，或與區塊陳述式搭配使用時作為參數。

`@`之後的所有項目都是選項：

```xml
${myVar @ optOne}
```

選項可以有值，可以是變數或常值：

```xml
${myVar @ optOne=someVar}
${myVar @ optOne='bar'}
${myVar @ optOne=10}
${myVar @ optOne=true}
```

多個選項以逗號分隔：

```xml
${myVar @ optOne, optTwo=bar}
```

也可以使用僅包含選項的參數表達式：

```xml
${@ optOne, optTwo=bar}
```

### 字串格式 {#string-formatting}

用相應的變數替換枚舉佔位符{*n*}的選項：

```xml
${'Page {0} of {1}' @ format=[current, total]}
```

## URL操作 {#url-manipulation}

提供一組新的URL操作。

請參閱下列範例，了解其使用方式：

將html擴充功能新增至路徑。

```xml
<a href="${item.path @ extension = 'html'}">${item.name}</a>
```

將html擴充功能和選取器新增至路徑。

```xml
<a href="${item.path @ extension = 'html', selectors='products'}">${item.name}</a>
```

將html擴充功能和片段(#value)新增至路徑。

```xml
<a href="${item.path @ extension = 'html', fragment=item.name}">${item.name}</a>
```

`@extension`適用於所有案例，檢查是否要新增擴充功能。

```xml
${ link @ extension = 'html' }
```

### 數字/日期格式 {#number-date-formatting}

HTL可讓數字和日期以原生格式設定，而無須編寫自訂程式碼。 這也支援時區和地區設定。

下列範例顯示先指定格式，然後再指定需要格式化的值：

```xml
<h2>${ 'dd-MMMM-yyyy hh:mm:ss' @
           format=currentPage.lastModified,
           timezone='PST',
           locale='fr'}</h2>

<h2>${ '#.00' @ format=300}</h2>
```

>[!NOTE]
>
>如需可使用格式的完整詳細資訊，請參閱[HTL-specification](https://github.com/Adobe-Marketing-Cloud/htl-spec/blob/master/SPECIFICATION.md)。

### 國際化 {#internationalization}

使用目前的[字典](https://experienceleague.adobe.com/docs/experience-manager-65/developing/components/internationalization/i18n-translator.html)將字串轉譯為目前&#x200B;*source*&#x200B;的語言（請參閱下文）。 如果找不到翻譯，則使用原始字串。

```xml
${'Page' @ i18n}
```

提示選項可用於為翻譯者提供注釋，指定使用文本的上下文：

```xml
${'Page' @ i18n, hint='Translation Hint'}
```

該語言的預設源為`resource`，這意味著文本將翻譯為與內容相同的語言。 這可以更改為`user`，這表示語言取自瀏覽器區域設定或來自登錄用戶的區域設定：

```xml
${'Page' @ i18n, source='user'}
```

提供顯式語言環境將覆蓋源設定：

```xml
${'Page' @ i18n, locale='en-US'}
```

若要將變數內嵌至翻譯的字串，可使用格式選項：

```xml
${'Page {0} of {1}' @ i18n, format=[current, total]}
```

### 陣列連接 {#array-join}

依預設，將陣列顯示為文字時，HTL會顯示逗號分隔值（無間距）。

使用聯接選項指定不同的分隔符：

```xml
${['one', 'two'] @ join='; '}
```

### 顯示內容 {#display-context}

HTL運算式的顯示內容會參照其在HTML頁面結構內的位置。 例如，如果表達式在呈現後就會生成文本節點，則該表達式據稱位於`text`上下文中。 如果在屬性的值內找到，則表示它位於`attribute`上下文中，以此類推。

除了指令碼(JS)和樣式(CSS)內容外，HTL會自動偵測運算式的內容並適當地逸出，以避免XSS安全性問題。 在指令碼和CSS的情況下，必須明確設定所需的內容行為。 此外，在需要自動行為覆寫的任何其他情況下，也可以明確設定上下文行為。

在這裡，我們有三個變數，分別存在於三個不同的內容中：

* `properties.link` (內 `uri` 容)
* `properties.title` (`attribute` 內容)
* `properties.text` (`text` 內容)

HTL會根據其個別內容的安全需求，以不同方式逸出其中的每一項。 通常情況下（如以下）不需要明確的上下文設定：

```xml
<a href="${properties.link}" title="${properties.title}">${properties.text}</a>
```

為了安全地輸出標籤（即，其中運算式本身評估為HTML），會使用`html`內容：

```xml
<div>${properties.richText @ context='html'}</div>
```

必須為樣式上下文設定顯式上下文：

```xml
<span style="color: ${properties.color @ context='styleToken'};">...</span>
```

必須為指令碼上下文設定顯式上下文：

```xml
<span onclick="${properties.function @ context='scriptToken'}();">...</span>
```

逸出和XSS保護也可以關閉：

```xml
<div>${myScript @ context='unsafe'}</div>
```

### 內容設定 {#context-settings}

| 上下文 | 使用時機 | 它的作用 |
|--- |--- |--- |
| `text` | 元素內的內容預設值 | 對所有HTML特殊字元進行編碼。 |
| `html` | 要安全輸出標籤 | 篩選HTML以符合AntiSamy策略規則，刪除與規則不匹配的內容。 |
| `attribute` | 屬性值的預設值 | 對所有HTML特殊字元進行編碼。 |
| `uri` | 顯示href和src屬性值的連結和路徑預設值 | 驗證URI是否寫入為href或src屬性值，如果驗證失敗則不輸出任何內容。 |
| `number` | 顯示數字 | 驗證包含整數的URI，如果驗證失敗則輸出零。 |
| `attributeName` | 設定屬性名稱時，data-sly-attribute的預設值 | 驗證屬性名稱時，若驗證失敗則不會輸出任何內容。 |
| `elementName` | data-sly-element的預設值 | 驗證元素名稱時，如果驗證失敗，則不會輸出任何內容。 |
| `scriptToken` | 針對JS識別碼、常值數字或常值字串 | 驗證JavaScript代號時，若驗證失敗則不會輸出任何內容。 |
| `scriptString` | 在JS字串內 | 對從字串中分出的字元進行編碼。 |
| `scriptComment` | 在JS注釋內 | 驗證JavaScript註解時，若驗證失敗則不會輸出任何內容。 |
| `styleToken` | 對於CSS標識符、數字、維、字串、十六進位顏色或函式。 | 驗證CSS代號時，若驗證失敗則不會輸出任何內容。 |
| `styleString` | 在CSS字串內 | 對從字串中分出的字元進行編碼。 |
| `styleComment` | 在CSS注釋內 | 驗證CSS註解，但驗證失敗時不會輸出任何內容。 |
| `unsafe` | 只有上述情況都沒有 | 完全禁用逸出和XSS保護。 |
