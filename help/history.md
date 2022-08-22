---
title: HTL歷史
description: 對於長時間的用戶AEM，本文給出了HTL的背景、JSP的替換方式，以及從Atley更改名稱。
exl-id: 00985b35-2130-4946-959a-0a09a34a0f05
source-git-commit: 5ab1275c984135fe946f36905bbc979cf19edd80
workflow-type: tm+mt
source-wordcount: '542'
ht-degree: 45%

---


# HTL歷史 {#history-of-htl}

對於長時間的用戶AEM，本文給出了HTL的背景、JSP的替換方式，以及從Atley更改名稱。

## 先前稱為 Sightly {#sightly}

HTML模板語言(HTL)是Adobe Experience ManagerHTML的首選和推薦的伺服器端模板系統。 它取代了舊版 AEM 中所使用的 JSP (JavaServer Pages)。

## HTL 優於 JSP {#htl-over-jsp}

建議新 AEM 專案最好使用 HTML 範本語言，因為相較於 JSP，HTL 提供了多項優點。 但如果是現有專案，只有在預估遷移比未來幾年維護現有 JSP 的工作量少時，遷移才有意義。

不過，移至 HTL 不見得是全有或全無的選擇，因為使用 HTL 撰寫的元件與使用 JSP 或 ESP 撰寫的元件相容。 這表示現有專案可以毫無問題地將 HTL 用於新元件，同時繼續將 JSP 用於現有元件。

即便是在相同元件中，HTL 檔案也可以與 JSP 和 ESP 一起使用。 以下範例在&#x200B;**第 1 行**&#x200B;顯示如何加入 JSP 檔案中的 HTL 檔案，並在&#x200B;**第 2 行**&#x200B;顯示如何加入 HTL 檔案中的 JSP 檔案：

```xml
<cq:include script="template.html"/>
<sly data-sly-include="template.jsp"/>
```

## 常見問答 {#frequently-asked-questions}

這些是HTL的新經驗豐富的開AEM發人員經常問的問題。

### HTL 是否有任何 JSP 沒有的限制？ {#limitations}

HTL與JSP相比並沒有任何限制，因為使用JSP可以做的事也應該通過HTL實現。 但是，HTL在設計上比JSP要嚴格。可以在單個JSP檔案中實現的所有內容，可能需要分離為Java類或JavaScript檔案，以便在HTL中實現。 但這通常需要確保邏輯與標記之間有良好的關注點分離。

### HTL 是否支援 JSP 標記庫？ {#tag-libraries}

否，但如 [正在載入客戶端庫](getting-started.md#loading-client-libraries) 「入門」文檔的「模板」和「呼叫」語句提供類似的模式。

### 可以在 AEM 專案上擴充 HTL 功能嗎？ {#extended}

不，他們不能。 HTL具有強大的邏輯復用擴展機制(REU) [使用 — API](#use-api-for-accessing-logic))和標籤（模板和調用語句），這些語句可用於模組化項目代碼。

### HTL 主要有哪些優點勝過 JSP？ {#benefits}

安全性和項目效率是主要的效益，詳細介紹了 [概述。](overview.md)

### JSP 最終會消失嗎？ {#go-away}

沒有這樣的計畫。

## 名字裡有什麼？ {#what-is-in-a-name}

在AEM6.0和6.1中，HTL稱為 **美麗**。 Adobe將其更名為 **HTML模板語言** 或 **HTL** 闡明規範的用途，並一般與Adobe的命名准則一致。 此命名變更自 2016 年 8 月起生效，適用於 AEM 6.0 和更高版本。

>[!NOTE]
>
>此命名變更不會影響程式碼或 API，所以相容性不受影響。 如需更多資訊，請 [請參閱此發佈視頻。](https://helpx.adobe.com/tw/experience-manager/how-to/announce-htl.html)

要瞭解HTL的更多資訊，我們的官員將提供一個理想的起點 [《HTML模板語言(HTL)入門指南》。](overview.md)
