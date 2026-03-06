---
solution: Experience Manager
type: Documentation
product: adobe experience manager
git-repo: https://github.com/AdobeDocs/experience-manager-htl.zh-Hant
index: true
landing-page-name: experience-manager
landing-page-breadcrumb-title: AEM
recommendations: noDisplay
source-git-commit: 944fa924e7ccba0a195b2c92584ab75df86b1f83
workflow-type: tm+mt
source-wordcount: '86'
ht-degree: 2%

---


# 內部使用的中繼資料

GitHub製作系統會以階層方式定義中繼資料，並增加前一項的層級，如下所示：

1. metadata.md
1. ToC
1. 文章

metadata.md檔案中定義的中繼資料會套用至整個存放庫，但可以在ToC和文章層級覆寫。 中繼資料的任何覆寫都應該儘可能在最低層級進行。

`experience-manager-core-components.en`存放庫中的中繼資料是最低要求。

metadata.md

* `product`
* `git-repo`
* `index: true`

已不再使用：

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
* `index: false` （僅適用於舊版元件）

