---
title: HTL 歷史記錄
description: 對於 AEM 的長期使用者，本文件說明 HTL 的背景、它如何取代 JSP 以及將名稱更改為 Sightly。
exl-id: 00985b35-2130-4946-959a-0a09a34a0f05
source-git-commit: c6bb6f0954ada866cec574d480b6ea5ac0b51a3f
workflow-type: ht
source-wordcount: '540'
ht-degree: 100%

---


# HTL 歷史記錄 {#history-of-htl}

對於 AEM 的長期使用者，本文件說明 HTL 的背景、它如何取代 JSP 以及將名稱更改為 Sightly。

## 先前稱為 Sightly {#sightly}

HTML 範本語言 (HTL) 是 Adobe Experience Manager 中首選且推薦使用的 HTML 伺服器端範本系統。 它取代了舊版 AEM 中所使用的 JSP (JavaServer Pages)。

## HTL 優於 JSP {#htl-over-jsp}

Adobe 建議，全新的 AEM 專案要使用 HTML 範本語言。原因是與 JSP 相比，此語言具有多種優勢。但如果是現有專案，只有在預估遷移比未來幾年維護現有 JSP 的工作量少時，遷移才有意義。

移至 HTL 不見得是全有或全無的選擇，因為使用 HTL 撰寫的元件與使用 JSP 或 ESP 撰寫的元件相容。 此方式表示現有專案可以為新元件使用 HTL，但同時可繼續為現有元件使用 JSP。

即便是在相同元件中，HTL 檔案也可以與 JSP 和 ESP 一起使用。 以下範例在&#x200B;**第 1 行**&#x200B;顯示如何加入 JSP 檔案中的 HTL 檔案，並在&#x200B;**第 2 行**&#x200B;顯示如何加入 HTL 檔案中的 JSP 檔案：

```xml
<cq:include script="template.html"/>
<sly data-sly-include="template.jsp"/>
```

## 常見問題 {#frequently-asked-questions}

剛接觸 HTL 但經驗豐富的 AEM 開發人員通常會問以下問題：

### HTL 是否有任何 JSP 沒有的限制？ {#limitations}

相較於 JSP，HTL 並沒有真正的限制，因為可使用 JSP 可以完成的事情應該也可以使用 HTL 達成。然而，HTL 的設計在許多方面上比 JSP 更嚴格。在單一 JSP 檔案中可以做到的內容可能需要分成 Java 類別或 JavaScript 檔案，這樣才能在 HTL 中做到。但這個方法通常需要確保邏輯與標記之間有良好的分離關注點。

### HTL 是否支援 JSP 標記庫？ {#tag-libraries}

否。 但是，如同「快速入門」文件的「[載入用戶端資料庫](getting-started.md#loading-client-libraries)」一節所述，範本和呼叫陳述式可提供類似的模式。

### 是否可以在 AEM 專案上擴充 HTL 功能？ {#extended}

否。HTL 具有可重複使用邏輯的強大擴充機制，利用這些邏輯 ([Use-API](#use-api-for-accessing-logic) 和標記 (範本和呼叫陳述式)) 可將專案的程式碼模組化。

### HTL 主要有哪些優點勝過 JSP？ {#benefits}

安全性和專案效率是主要優點，這些在[概覽](overview.md)中有詳述。

### 是否會停止使用 JavaServer Pages (JSP)？ {#go-away}

否。 目前還沒有停止使用 JSP 的計劃。

## 名稱的含意為何？ {#what-is-in-a-name}

在 AEM 6.0 和 6.1 中，HTL 稱為 **Sightly**。Adobe 將其重新命名為「**HTML 範本語言**」，或 **HTL**，以釐清此規格的用途，而且大體上符合 Adobe 的命名準則。此命名變更自 2016 年 8 月起生效，適用於 AEM 6.0 和更高版本。

>[!NOTE]
>
>此命名變更不會影響程式碼或 API，所以相容性不受影響。 如需詳細資訊，請[參考這段公告影片](https://helpx.adobe.com/tw/experience-manager/how-to/announce-htl.html)。

若要了解有關 HTL 的更多資訊，請參閱 [HTML 範本語言 (HTL) 入門指南](overview.md)。
