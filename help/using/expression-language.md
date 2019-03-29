---
title: HTL運算式語言
seo-title: HTL運算式語言
description: HTML範本語言使用運算式語言來存取提供HTML輸出動態元素的資料結構。
seo-description: HTML範本語言使用運算式語言來存取提供HTML輸出動態元素的資料結構。
uuid: 38b4a259-03b5-4847-91c6-e20377600070
contentOwner: 使用者
products: SG_ PERIENCENCENAGER/HTL
topic-tags: html-template-language
content-type: 引用
discoiquuid: 9ba37ca0-f318-48b0-a791-a944 a72502 eed
mwpw-migration-script-version: 2017-10-12T214658.665-0400
translation-type: tm+mt
source-git-commit: 796c55d3d85e6b5a3efaa5c04a25be1b0b4e54dd

---


# HTL運算式語言 {#htl-expression-language}

HTML範本語言使用運算式語言來存取提供HTML輸出動態元素的資料結構。這些運算式會以字元 `${` 分隔 `}`。為避免格式錯誤，運算式只能用於屬性值、元素內容中或註解中。

```xml
<!-- ${component.path} -->
<h1 class="${component.name}">
    ${properties.jcr:title}
</h1>
```

運算式可由 **`\`** 字元前頭逸出，例如 **`\${test}`** 演算 **`${test}`**。

>[!NOTE]
>
>若要試用本頁上提供的範例，可使用稱為「Read Eval Print Loop [](https://github.com/Adobe-Marketing-Cloud/aem-sightly-repl) 」的即時執行環境。

運算式語法包含 [變數](#variables)、 [字詞](#literals)、 [運算子](#operators) 和 [選項](#options)：

## 變數 {#variables}

變數是儲存資料值或物件的容器。變數名稱稱為識別碼。

HTL不需指定任何項目，可讓您存取在加入JSP後常用的所有物件 `global.jsp`。[「全域物件](global-objects.md) 」頁面提供所有物件的清單，由HTL提供存取權。

### 屬性存取 {#property-access}

有兩種方法可存取變數屬性、點標記或括號符號：

`${currentPage.title}  
${currentPage['title']} or ${currentPage["title"]}`

大多數情況下應偏好較簡單的點標記，且應使用括號符號來存取包含無效識別碼字元的屬性，或動態存取屬性。下列兩個章節將提供這兩個案例的詳細資訊。

存取的屬性可以是函數，但傳遞的引數不受支援，所以只有不預期引數的函數才能存取，例如getter。這是所需的限制，目的是降低嵌入運算式中的邏輯量。如有需要， [`data-sly-use`](block-statements.md#use) 可使用陳述式來傳遞參數至邏輯。

上述範例中也顯示Java getter函數(例如 `getTitle()`，可存取)而不預先擱置， **`get`**並降低後續字元的大小寫。

### 有效的Indentifier字元 {#valid-indentifier-characters}

變數名稱(稱為識別碼)符合某些規則。它們必須以字母(-**`A`****`Z`** 和 **`a`**-)**`z`**開頭，而後續**`_`**的字元也可以是**`0`**數字(-**`9`**)或冒號(**`:`**)。Unicode字母(例如 **`å`** ， **`ü`** 無法用於識別碼)。

由於冒號(**：**)字元在AEM屬性名稱中很常見，因此很方便它是有效的識別碼字元：

`${properties.jcr:title}`

括號符號可用來存取包含無效識別碼字元的屬性，例如下列範例中的空格字元：

`${properties['my property']}`

### 動態存取成員 {#accessing-members-dynamically}

<!-- 

Comment Type: draft

<p>TODO: add description</p>

 -->

```xml
${properties[myVar]}
```

### 空值的權限處理 {#permissive-handling-of-null-values}

<!-- 

Comment Type: draft

<p>TODO: add description</p>

 -->

```xml
${currentPage.lastModified.time.toString}
```

## 常式 {#literals}

常值是表示固定值的符號。

### 布林值 (Boolean){#boolean}

布林代表邏輯實體，可以有兩個值： **`true`****`false`**以及.

`${true} ${false}`

### 數字 {#numbers}

只有一種數字類型：正整數。雖然其他數字格式(例如浮點)在變數中受支援，但無法以文字表示。

`${42}`

### 字串 {#strings}

它們代表文字資料，而且可以是單一或雙引號：

`${'foo'} ${"bar"}`

除了一般字元之外，還可使用下列特殊字元：

* **`\\`** 反斜線字元
* **`\'`** 單引號(或撇號)
* **`\"`** 雙引號
* **`\t`** 標籤式
* **`\n`** 新行
* **`\r`** 歸位
* **`\f`** 表單摘要
* **`\b`** Backspace
* `\uXXXX` 由四個十六進位數字XXXX指定的Unicode字元。\
   一些有用的Unicode逸出序列為：

   * **\u0022** for **"**
   * **\u0027** for **'**

For characters not listed above, preceding a backslash caracter will display an error.

Here are some examples of how to use string escaping:

```xml
<p>${'it\'s great, she said "yes!"'}</p>
<p title="${'it\'s great, she said \u0022yes!\u0022'}">...</p>
```

which will result in following output, because HTL will apply context-specific escaping:

```xml
<p>it&#39;s great, she said &#34;yes!&#34;</p>
<p title="it&#39;s great, she said &#34;yes!&#34;">...</p>
```

### Arrays {#arrays}

An array is an ordered set of values that can be referred to with a name and an index. The types of its elements can be mixed.

<!-- 

Comment Type: draft

<p>TODO: add description</p>

 -->

```xml
${[1,2,3,4]}
${myArray[2]}
```

Arrays are useful to provide a list of values from the template.

```xml
<ul data-sly-list="${[1,2,3,4]}">
  <li>${item}</li>
</ul>
```

## Operators {#operators}

### Logical Operators {#logical-operators}

These operators are typically used with Boolean values, however, like in JavaScript, they actually return the value of one of the specified operands, so when used with non-Boolean values, they may return a non-Boolean value.

If a value can be converted to **`true`**, the value is so-called truthy. If a value can be converted to **`false`**, the value is so-called falsy. Values that can be converted to **`false`** are: undefined variables, null values, the number zero, and empty strings.

#### Logical NOT {#logical-not}

**`${!myVar}`** returns **`false`** if its single operand can be converted to `true`; otherwise, returns **`true`**.

This can for instance be used to invert a test condition, like displaying an element only if there are no child pages:

```xml
<p data-sly-test="${!currentPage.hasChild}">current page has no children</p>
```

#### Logical AND {#logical-and}

**`${varOne && varTwo}`** returns `varOne` if it is falsy; otherwise, returns **varTwo**.

This operator can be used to test two conditions at once, like verifying the existence of two properties:

```xml
<div data-sly-test="${properties.jcr:title && properties.jcr:description}">
    <h1>${properties.jcr:title}</h1>
    <p>${properties.jcr:description}</p>
</div>
```

The logical AND operator can also be used to conditionally display HTML attributes, because HTL removes attributes with values set dynamically that evaluate to false, or to an empty string. So in the example below, the **`class`** attribute is only shown if **`logic.showClass`** is truthy and if **`logic.className`** exists and is not empty:

```xml
<div class="${logic.showClass && logic.className}">...</div>
```

#### Logical OR {#logical-or}

**`${varOne || varTwo}`** returns **varOne** if it is truthy; otherwise, returns **varTwo**.

This operator can be used to test if one of two conditions apply, like verifying the existence of at least one property:

```xml
<div data-sly-test="${properties.jcr:title || properties.jcr:description}">...</div>
```

As the logical OR operator returns the first variable that is truthy, it can also very conveniently be used to provide fallback values.

conditionally display HTML attributes, because HTL removes attributes with values set by expressions that evaluate to false or to an empty string. So the example below will display **`properties.jcr:`**title if it exists and is not empty, else it falls back to dislaying **`properties.jcr:description`** if it exists and is not empty, else it will display the message "no title or description provided":

```xml
<p>${properties.jcr:title || properties.jcr:description || "no title or description provided"}</p>
```

### Conditional (ternary) Operator {#conditional-ternary-operator}

**`${varCondition ? varOne : varTwo}`** returns **`varOne`** if **`varCondition`** is truthy; otherwise it returns **`varTwo`**.

This operator can typically be used to define conditions within expressions, like displaying a different message based on the status of the page:

```xml
<p>${currentPage.isLocked ? "page is locked" : "page can be edited"}</p>
```

An important note, since colon characters are also permitted in identifiers, it is best to separate the ternary operators with a white space to provide clarity to the parser:

```xml
<p>${properties.showDescription ? properties.jcr:description : properties.jcr:title}</p>
```

### Comparison Operators {#comparison-operators}

The equality and inequality operators only support operands that are of identical types. When the types don't match, an error is displayed.

* Strings are equal when they have the same sequence of characters.
* Numbers are equal when they have the same value
* Booleans are equal if both are **`true`** or both are **`false`**.

* Null or undefined variables are equal to themselves and to each other.

**`${varOne == varTwo}`** returns **`true`** if **`varOne`** and **`varTwo`** are equal.

**`${varOne != varTwo}`** returns **`true`** if **`varOne`** and **`varTwo`** are not equal.

The relational operators only support operands that are numbers. For all other types, an error is displayed.

**`${varOne > varTwo}`** returns **`true`** if **`varOne`** is greater than **`varTwo`**.

**`${varOne < varTwo}`** returns **`true`** if **`varOne`** is smaller than **`varTwo`**.

**`${varOne >= varTwo}`** returns **`true`** if **`varOne`** is greater or equal to **`varTwo`**.

**`${varOne <= varTwo}`** returns **`true`** if **`varOne`** is smaller or equal to **`varTwo`**.

### Grouping parentheses {#grouping-parentheses}

The grouping operator **`(`** **`)`** controls the precedence of evaluation in expressions.

`${varOne && (varTwo || varThree)}`

## 選項 {#options}

<!-- 

Comment Type: draft

<p>TODO: review text below.</p>

 -->

Expression options can act on the expression and modify it, or serve as parameters when used in conjunction with block statements.

Everything after the **`@`** is an option:

```xml
${myVar @ optOne}
```

Options can have a value, which may be a variable or a literal:

```xml
${myVar @ optOne=someVar}
${myVar @ optOne='bar'}
${myVar @ optOne=10}
${myVar @ optOne=true}
```

Multiple options are separated by commas:

```xml
${myVar @ optOne, optTwo=bar}
```

Parametric expressions containing only options are also possible:

```xml
${@ optOne, optTwo=bar}
```

### String Formatting {#string-formatting}

Option that replaces the enumerated placeholders, {*n*}, with the corresponding variable:

```xml
${'Page {0} of {1}' @ format=[current, total]}
```

### Internationalization {#internationalization}

Translates the string to the language of the current *source* (see below), using the current [dictionary](https://helpx.adobe.com/experience-manager/6-3/sites/developing/using/i18n-translator). If no translation is found, the original string is used.

```xml
${'Page' @ i18n}
```

The hint option can be used to provide a comment for translators, specifying the context in which the text is used:

```xml
${'Page' @ i18n, hint='Translation Hint'}
```

The default source for the language is 'resource', meaning that the text gets translated to the same language as the content. This can be changed to 'user', meaning that the language is taken from the browser locale or from the locale of the logged-in user:

```xml
${'Page' @ i18n, source='user'}
```

Providing an explicit locale overrides the source settings:

```xml
${'Page' @ i18n, locale='en-US'}
```

To embed variables into a translated string, the format option can be used:

```xml
${'Page {0} of {1}' @ i18n, format=[current, total]}
```

### Array Join {#array-join}

By default, when displaying an array as text, HTL will display comma separated values (without spacing).

Use the join option to specify a different separator:

```xml
${['one', 'two'] @ join='; '}
```

### Display Context {#display-context}

The display context of a HTL expression refers to its location within the structure of the HTML page. For example, if the expression appears in place that would produce a text node once rendered, then it is said to be in a **`text`** context. If it is found within the value of an attribute, then it is said to be in an **`attribute`** context, and so forth.

With the exception of script (JS) and style (CSS) contexts, HTL will automatically detect the context of expressions and escape them appropriately, to prevent XSS security problems. In the case of scripts and CSS, the desired context behavior must be explicitly set. Additionally, the context behavior can also be explicitly set in any other case where an override of the automatic behavior is desired.

Here we have three variables in three different contexts: **`properties.link`** ( `uri` context), **`properties.title`** (**`attribute`** context) and **`properties.text`**(**`text`** context). HTL will escape each of these differently in accordance with the security requirements of their respective contexts. No explicit context setting is required in normal cases such as this one:

```xml
<a href="${properties.link}" title="${properties.title}">${properties.text}</a>
```

To safely output markup (that is, where the expression itself evaluates to HTML), the `html` context is used:

```xml
<div>${properties.richText @ context='html'}</div>
```

Explicit context must be set for style contexts:

```xml
<span style="color: ${properties.color @ context='styleToken'};">...</span>
```

Explicit context must be set for script contexts:

```xml
<span onclick="${properties.function @ context='scriptToken'}();">...</span>
```

Escaping and XSS protection can also be turned off:

```xml
<div>${myScript @ context='unsafe'}</div>
```

### Context Settings {#context-settings}

| 上下文 | When to use | What it does |
|--- |--- |--- |
| 文字 | Default for content inside elements | Encodes all HTML special characters. |
| html | To safely output markup | Filters HTML to meet the AntiSamy policy rules, removing what doesn't match the rules. |
| attribute | Default for attribute values | Encodes all HTML special characters. |
| uri | To display links and paths Default for href and src attribute values | Validates URI for writing as an href or src attribute value, outputs nothing if validation fails. |
| 數字 | To display numbers | Validates URI for containing an integer, outputs zero if validation fails. |
| attributeName | Default for data-sly-attribute when setting attribute names | Validates the attribute name, outputs nothing if validation fails. |
| elementName | Default for data-sly-element | Validates the element name, outputs nothing if validation fails. |
| scriptToken | For JS identifiers, literal numbers, or literal strings | Validates the JavaScript token, outputs nothing if validation fails. |
| scriptString | Within JS strings | Encodes characters that would break out of the string. |
| scriptComment | Within JS comments | Validates the JavaScript comment, outputs nothing if validation fails. |
| styleToken | For CSS identifiers, numbers, dimensions, strings, hex colours or functions. | Validates the CSS token, outputs nothing if validation fails. |
| styleString | Within CSS strings | Encodes characters that would break out of the string. |
| styleComment | Within CSS comments | Validates the CSS comment, outputs nothing if validation fails. |
| unsafe | Only if none of the above does the job | Disables escaping and XSS protection completely. |

