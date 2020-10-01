---
title: HTL 運算式語言
description: HTML範本語言使用運算式語言來存取提供HTML輸出動態元素的資料結構。
translation-type: tm+mt
source-git-commit: c7fa6014cd954a2ccb175e4c3a6be9deb83af890
workflow-type: tm+mt
source-wordcount: '1854'
ht-degree: 0%

---


# HTL 運算式語言 {#htl-expression-language}

HTML範本語言使用運算式語言來存取提供HTML輸出動態元素的資料結構。 這些運算式會以字元和 `${` 分隔 `}`。 為避免格式錯誤的HTML，運算式只能用於屬性值、元素內容或注釋中。

```xml
<!-- ${component.path} -->
<h1 class="${component.name}">
    ${properties.jcr:title}
</h1>
```

運算式可借由預先標示字元來 `\` 逸出，例如 `\${test}` 將會演算 `${test}`。

>[!NOTE]
>
>要試用本頁上提供的示例，可以使用名為「讀取評估打印循環 [](https://github.com/Adobe-Marketing-Cloud/aem-sightly-repl) 」的即時執行環境。

運算式變 [數](#variables)、文字 [、運](#literals)算子和 [選](#operators) 項語法 [](#options):

## 變數 {#variables}

變數是儲存資料值或物件的容器。 變數的名稱稱為識別碼。

HTL不需指定任何項目，就可讓您在加入後，存取JSP中常用的所有物件 `global.jsp`。 「全 [域物件](global-objects.md) 」頁面提供HTL可存取的所有物件清單。

### 屬性存取 {#property-access}

存取變數屬性的方法有兩種：點記號或方括弧記號：

```xml
${currentPage.title}  
${currentPage['title']} or ${currentPage["title"]}
```

大多數情況下，最好使用更簡單的點標籤法，並且應使用括弧標籤法來訪問包含無效標識符字元的屬性，或動態訪問屬性。 以下兩章將提供這兩個案例的詳細資訊。

訪問的屬性可以是函式，但不支援傳遞參數，因此只能訪問不期望參數的函式，如getter。 這是一個需要的限制，其目的是減少內嵌在運算式中的邏輯量。 如果需要， [`data-sly-use`](block-statements.md#use) 可以使用語句將參數傳遞到邏輯。

上例中還顯示了Java getter函式(如 `getTitle()`,)可以訪問，而不用預置 `get`，並通過降低後面字元的大小寫。

### 有效識別碼字元 {#valid-identifier-characters}

變數的名稱（稱為識別碼）符合特定規則。 它們必須以字母(`A`-`Z``a`和-`z`)或下划線(`_`)開頭，後續字元也可以是數字(-`0``9``:`)或冒號()。 Unicode字母(如 `å` 和) `ü` 不能用於標識符。

由於冒號(`:`)字元在AEM屬性名稱中很常見，因此應強調它是有效的識別碼字元：

`${properties.jcr:title}`

括弧符號可用於訪問包含無效標識符字元的屬性，如以下示例中的空格字元：

`${properties['my property']}`

### 動態訪問成員 {#accessing-members-dynamically}

```xml
${properties[myVar]}
```

### Null值的權限處理 {#permissive-handling-of-null-values}

```xml
${currentPage.lastModified.time.toString}
```

## 文字 {#literals}

常值是表示固定值的符號。

### 布林值 (Boolean){#boolean}

布爾值表示邏輯實體，可以有兩個值： `true`和 `false`。

`${true} ${false}`

### 數字 {#numbers}

只有一種數字類型：正整數。 其他數字格式（如浮點）在變數中受支援，但不能表示為文字。

`${42}`

### 字串 {#strings}

字串代表文字資料，可以是單引號或雙引號：

`${'foo'} ${"bar"}`

除了普通字元外，還可使用下列特殊字元：

* `\\` 反斜線字元
* `\'` 單引號（或撇號）
* `\"` 雙引號
* `\t` 頁籤
* `\n` 新行
* `\r` 歸位
* `\f` 表單摘要
* `\b` 回空格
* `\uXXXX` 由四個十六進位數字XXXX指定的Unicode字元。\
   一些有用的Unicode轉義序列包括：

   * `\u0022` 的 `"`
   * `\u0027` 的 `'`

對於上方未列出的字元，反斜線字元前面會顯示錯誤。

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

陣列是一組有序值，可以用名稱和索引引用。 其元素類型可混合。

```xml
${[1,2,3,4]}
${myArray[2]}
```

陣列在提供模板值清單時很有用。

```xml
<ul data-sly-list="${[1,2,3,4]}">
  <li>${item}</li>
</ul>
```

## 營運商 {#operators}

### 邏輯運算子 {#logical-operators}

這些運算子通常與布林值搭配使用，但是，就像在JavaScript中一樣，它們實際上會傳回其中一個指定運算元的值，因此當與非布林值搭配使用時，可能會傳回非布林值。

如果值可以轉換為 `true`，則該值稱為truthy。 如果值可轉換為 `false`，則該值稱為falsy。 可轉換為的值是未 `false` 定義的變數、空值、數字零和空字串。

#### 邏輯非 {#logical-not}

`${!myVar}` 如果 `false` 其單個操作數可轉換為 `true`;否則，返回 `true`。

例如，這可用於反轉測試條件，例如只有在沒有子頁面時才顯示元素：

```xml
<p data-sly-test="${!currentPage.hasChild}">current page has no children</p>
```

#### 邏輯和 {#logical-and}

`${varOne && varTwo}` 如果 `varOne` 是假的，則返回；否則，返回 `varTwo`。

此運算子可用來一次測試兩個條件，例如驗證是否存在兩個屬性：

```xml
<div data-sly-test="${properties.jcr:title && properties.jcr:description}">
    <h1>${properties.jcr:title}</h1>
    <p>${properties.jcr:description}</p>
</div>
```

邏輯AND運算子也可用來有條件地顯示HTML屬性，因為HTL會移除值動態設定為false或空字串的屬性。 因此，在以下範例中， `class` 只有在屬性真實且 `logic.showClass` 存在且非空 `logic.className` 白時，才會顯示屬性：

```xml
<div class="${logic.showClass && logic.className}">...</div>
```

#### 邏輯或 {#logical-or}

`${varOne || varTwo}` 如果 `varOne` 是真的，則返回；否則，返回 `varTwo`。

此運算子可用來測試是否適用下列兩種條件之一，例如驗證是否存在至少一個屬性：

```xml
<div data-sly-test="${properties.jcr:title || properties.jcr:description}">...</div>
```

由於邏輯OR運算子會傳回第一個真實的變數，因此也可非常方便地用來提供備援值。

它也可用來有條件地顯示HTML屬性，因為HTL會移除由運算式設定的值，以評估為false或空字串的屬性。 因此，以下範例將顯示 **`properties.jcr:`** 標題（如果存在且不為空白），否則它會返回顯示 **`properties.jcr:description`** （如果存在且不空白），否則它將顯示訊息「未提供標題或說明」:

```xml
<p>${properties.jcr:title || properties.jcr:description || "no title or description provided"}</p>
```

### 條件（三元）運算子 {#conditional-ternary-operator}

`${varCondition ? varOne : varTwo}` 如果 `varOne` 真 `varCondition` 實，則回報；否則，它將返回 `varTwo`。

此運算子通常可用來定義運算式中的條件，例如根據頁面狀態顯示不同的訊息：

```xml
<p>${currentPage.isLocked ? "page is locked" : "page can be edited"}</p>
```

>[!TIP]
>
>由於識別碼中也允許冒號字元，因此最好將三元運算子與空白區分開，以清楚說明剖析器：

```xml
<p>${properties.showDescription ? properties.jcr:description : properties.jcr:title}</p>
```

### 比較運算子 {#comparison-operators}

等式和不等式運算子僅支援相同類型的操作數。 當類型不符合時，會顯示錯誤。

* 如果字串具有相同的字元順序，則字串是相等的。
* 數值相同時，數值相等
* 如果兩者皆為或兩者皆 `true` 為，則布爾 `false`數相等。
* Null或未定義的變數彼此相等。

`${varOne == varTwo}` 若 `true` 和 `varOne` 等 `varTwo` 於則傳回。

`${varOne != varTwo}` 若 `true` 和 `varOne` 不 `varTwo` 相等，則傳回。

關係運算子僅支援數字操作數。 對於所有其他類型，都會顯示錯誤。

`${varOne > varTwo}` 若 `true` 大於 `varOne` 則傳回 `varTwo`。

`${varOne < varTwo}` 傳 `true` 回 `varOne` 小於的 `varTwo`值。

`${varOne >= varTwo}` 若 `true` 大於或等於 `varOne` 則傳回 `varTwo`。

`${varOne <= varTwo}` 若 `true` 小 `varOne` 於或等於則傳回 `varTwo`。

### 分組括弧 {#grouping-parentheses}

群組運算子 `()` 控制運算式中評估的優先順序。

`${varOne && (varTwo || varThree)}`

## 選項 {#options}

運算式選項可對運算式執行並加以修改，或與區塊陳述式搭配使用時，可當成參數。

之後的一 `@` 切都是一種選擇：

```xml
${myVar @ optOne}
```

選項可以有值，該值可以是變數或常值：

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

也可以使用只包含選項的參數表達式：

```xml
${@ optOne, optTwo=bar}
```

### 字串格式 {#string-formatting}

將列舉的預留位置{*n*}替換為對應變數的選項：

```xml
${'Page {0} of {1}' @ format=[current, total]}
```

## URL操縱 {#url-manipulation}

有新的URL操縱集可供使用。

請參閱下列使用範例：

將html副檔名新增至路徑。

```xml
<a href="${item.path @ extension = 'html'}">${item.name}</a>
```

將html副檔名和選擇器新增至路徑。

```xml
<a href="${item.path @ extension = 'html', selectors='products'}">${item.name}</a>
```

將html副檔名和片段(#value)新增至路徑。

```xml
<a href="${item.path @ extension = 'html', fragment=item.name}">${item.name}</a>
```

在所 `@extension` 有情況下都能運作，檢查是否要添加副檔名。

```xml
${ link @ extension = 'html' }
```

### 數字／日期格式 {#number-date-formatting}

HTL允許數字和日期的原生格式化，毋需編寫自訂程式碼。 這也支援時區和地區設定。

下列範例顯示，先指定格式，再指定需要格式的值：

```xml
<h2>${ 'dd-MMMM-yyyy hh:mm:ss' @
           format=currentPage.lastModified,
           timezone='PST',
           locale='fr'}</h2>

<h2>${ '#.00' @ format=300}</h2>
```

>[!NOTE]
>
>有關可使用格式的完整詳細資訊，請參閱 [HTL規格](https://github.com/Adobe-Marketing-Cloud/htl-spec/blob/master/SPECIFICATION.md)。

### 國際化 {#internationalization}

使用目前的字典，將字串轉譯為 *目前來源* (請參閱下 [文)](https://docs.adobe.com/content/help/en/experience-manager-65/developing/components/internationalization/i18n-translator.html)。 如果找不到任何轉譯，則會使用原始字串。

```xml
${'Page' @ i18n}
```

提示選項可用於為翻譯者提供注釋，指定使用文本的上下文：

```xml
${'Page' @ i18n, hint='Translation Hint'}
```

語言的預設來源為， `resource`這表示文字會翻譯成與內容相同的語言。 這可以變更為 `user`，這表示語言是從瀏覽器地區設定或登入使用者的地區設定取得：

```xml
${'Page' @ i18n, source='user'}
```

提供明確的語言環境會覆蓋源設定：

```xml
${'Page' @ i18n, locale='en-US'}
```

若要將變數內嵌至已翻譯的字串，可使用format選項：

```xml
${'Page {0} of {1}' @ i18n, format=[current, total]}
```

### 陣列連接 {#array-join}

依預設，當將陣列顯示為文字時，HTL會顯示逗號分隔值（無間距）。

使用聯接選項指定不同的分隔符：

```xml
${['one', 'two'] @ join='; '}
```

### 顯示內容 {#display-context}

HTL運算式的顯示內容會參照其在HTML頁面結構中的位置。 例如，如果表達式出現在原地，在渲染後將生成文本節點，則該表達式據說位於上 `text` 下文。 如果在屬性的值中找到，則表示它位於上 `attribute` 下文中，依此類推。

除了指令碼(JS)和樣式(CSS)上下文外，HTL會自動偵測運算式的上下文並適當地逸出它們，以防止XSS安全性問題。 在指令碼和CSS中，必須明確設定所要的上下文行為。 此外，在需要覆蓋自動行為的任何其他情況下，也可以顯式設定上下文行為。

在這裡，我們有三個變數：

* `properties.link` (上 `uri` 下文)
* `properties.title` (上`attribute` 下文)
* `properties.text` (上`text` 下文)

HTL將根據其各自上下文的安全要求以不同方式逃離其中每一個。 一般情況下，如下情況，則不需要明確的上下文設定：

```xml
<a href="${properties.link}" title="${properties.title}">${properties.text}</a>
```

要安全輸出標籤（即，表達式本身評估為HTML的位置），請使 `html` 用上下文：

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

| 上下文 | 使用時機 | 它的功能 |
|--- |--- |--- |
| `text` | 元素內容的預設值 | 對所有HTML特殊字元進行編碼。 |
| `html` | 要安全輸出標籤 | 篩選HTML以符合AntiSamy原則規則，移除不符合規則的項目。 |
| `attribute` | 屬性值的預設值 | 對所有HTML特殊字元進行編碼。 |
| `uri` | 要顯示連結和路徑，請預設href和src屬性值 | 驗證URI是否寫入為href或src屬性值，如果驗證失敗則不輸出任何內容。 |
| `number` | 要顯示數字 | 驗證包含整數的URI，如果驗證失敗，則輸出零。 |
| `attributeName` | 設定屬性名稱時的data-sly-attribute預設值 | 驗證屬性名稱，如果驗證失敗，則不輸出任何內容。 |
| `elementName` | 預設的資料密碼元素 | 驗證元素名稱，如果驗證失敗，則不輸出任何內容。 |
| `scriptToken` | 對於JS識別碼、常值數字或常值字串 | 驗證JavaScript Token，如果驗證失敗，則不會輸出任何內容。 |
| `scriptString` | 在JS字串中 | 編碼將從字串中分開的字元。 |
| `scriptComment` | 在JS注釋中 | 驗證JavaScript注釋，如果驗證失敗，則不會輸出任何內容。 |
| `styleToken` | 用於CSS識別碼、數字、尺寸、字串、十六進位色彩或函式。 | 驗證CSS Token，如果驗證失敗，則不會輸出任何內容。 |
| `styleString` | 在CSS字串中 | 編碼將從字串中分開的字元。 |
| `styleComment` | 在CSS注釋中 | 驗證CSS注釋，如果驗證失敗則不輸出任何內容。 |
| `unsafe` | 只有當上述任何一個 | 完全禁用轉義和XSS保護。 |
