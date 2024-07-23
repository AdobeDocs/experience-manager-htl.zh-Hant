---
title: HTL 歷史記錄
description: 對於 AEM 的長期使用者，本文件說明 HTL 的背景、它如何取代 JSP 以及將名稱更改為 Sightly。
exl-id: 00985b35-2130-4946-959a-0a09a34a0f05
source-git-commit: c6bb6f0954ada866cec574d480b6ea5ac0b51a3f
workflow-type: tm+mt
source-wordcount: '540'
ht-degree: 40%

---


# HTL 歷史記錄 {#history-of-htl}

對於 AEM 的長期使用者，本文件說明 HTL 的背景、它如何取代 JSP 以及將名稱更改為 Sightly。

## 先前稱為 Sightly {#sightly}

HTML 範本語言 (HTL) 是 Adobe Experience Manager 中首選且推薦使用的 HTML 伺服器端範本系統。 它取代了舊版 AEM 中所使用的 JSP (JavaServer Pages)。

## HTL 優於 JSP {#htl-over-jsp}

Adobe建議針對新AEM專案，使用HTML範本語言。 原因是相較於JSP，它提供了多項優點。 但如果是現有專案，只有在預估遷移比未來幾年維護現有 JSP 的工作量少時，遷移才有意義。

移至HTL不見得是全有或全無的選擇，因為使用HTL撰寫的元件與使用JSP或ESP撰寫的元件相容。 此方法表示現有專案可以毫無問題地將HTL用於新元件，同時繼續將JSP用於現有元件。

即便是在相同元件中，HTL 檔案也可以與 JSP 和 ESP 一起使用。 以下範例在&#x200B;**第1**&#x200B;行顯示如何加入JSP檔案中的HTL檔案，並在&#x200B;**第2**&#x200B;行顯示如何加入HTL檔案中的JSP檔案：

```xml
<cq:include script="template.html"/>
<sly data-sly-include="template.jsp"/>
```

## 常見問題 {#frequently-asked-questions}

剛開始接觸HTL的經驗豐富的AEM開發人員通常會提出下列問題：

### HTL 是否有任何 JSP 沒有的限制？ {#limitations}

相較於JSP，HTL沒有什麼限制，因為可以使用JSP完成的事情應該也可以使用HTL達成。 不過，在多個方面，HTL在設計上比JSP嚴格。 可以在單一JSP檔案中達成的目標，可能需要分散到Java類別或JavaScript檔案中，才能在HTL中達成。 但這種方法通常需要確保在邏輯和標籤之間有良好的關注點分離。

### HTL 是否支援 JSP 標記庫？ {#tag-libraries}

不適用。 不過，如同[快速入門]檔案的[載入使用者端資料庫[](getting-started.md#loading-client-libraries)]區段中所示，範本和呼叫陳述式可提供類似的模式。

### 可以在 AEM 專案上擴充 HTL 功能嗎？ {#extended}

不適用。 HTL 具有可重複使用邏輯的強大擴充機制，利用這些邏輯 ([Use-API](#use-api-for-accessing-logic) 和標記 (範本和呼叫陳述式)) 可將專案的程式碼模組化。

### HTL 主要有哪些優點勝過 JSP？ {#benefits}

安全性與專案效率是主要優點，在[概述](overview.md)中有詳細說明。

### JavaServer Pages (JSP)會離開嗎？ {#go-away}

不適用。 沒有中斷JSP的計畫。

## 名稱的含意為何？ {#what-is-in-a-name}

在AEM 6.0和6.1中，HTL稱為&#x200B;**Sightly**。 Adobe已將其重新命名為「**HTML範本語言**」或「**HTL**」，以釐清此規格的用途，並大致符合Adobe的命名准則。 此命名變更自 2016 年 8 月起生效，適用於 AEM 6.0 和更高版本。

>[!NOTE]
>
>此命名變更不會影響程式碼或API，因此相容性不受影響。 如需詳細資訊，請[參考此宣告影片](https://helpx.adobe.com/tw/experience-manager/how-to/announce-htl.html)。

若要進一步瞭解HTL，請參閱[HTML範本語言(HTL)快速入門手冊](overview.md)。
