---
product: Adobe Experience Manager
git-repo: https://git.corp.adobe.com/AdobeDocs/experience-manager-htl.zh-Hant
index: y
solution-title: HTL的學習與支援
solution-hub-url: https://docs.adobe.com/content/help/zh-Hant/experience-manager-cloud-service/sites/home.html
getting-started-title: AEM快速入門開發
getting-started-url: https://docs.adobe.com/content/help/zh-Hant/experience-manager-cloud-service/core-concepts/home.html
tutorials-title: AEM 教學課程
tutorials-url: https://docs.adobe.com/content/help/en/experience-manager-learn/cloud-service/overview.html
translation-type: tm+mt
source-git-commit: d3426d87dce09ac34ff1aca431ff2bfad2f7134a
workflow-type: tm+mt
source-wordcount: '139'
ht-degree: 20%

---


# 內部使用的中繼資料

GitHub製作系統中的中繼資料是階層式的，並定義了下列不斷提升的先例等級。

1. metadata.md
1. ToC
1. 文章

中繼資料。md檔案中定義的中繼資料會套用至整個回購區，但可在ToC和文章層級覆寫。 任何對中繼資料的覆寫都應盡可能在最低層級進行。

experience-manager-core-components.en repo中的中繼資料是最低要求。

metadata.md

* `product`
* `git-repo`
* `index: y`
* `solution-title`
* `solution-hub-url`
* `getting-started-title`
* `getting-started-url`
* `tutorials-title`
* `tutorials-url`

ToCs

* `sub-product`
* `user-guide-title`

文章

* `title`
* `description`
* `index: n` （僅適用於舊版元件）

有關中繼資料的其他資訊，請參閱內 [部製作指南。](https://docs.adobe.com/help/en/collaborative-doc-instructions/collaboration-guide/markdown/metadata.html#solution-metadata)
