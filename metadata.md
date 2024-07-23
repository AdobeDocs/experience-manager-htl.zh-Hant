---
solution: Experience Manager
type: Documentation
product: adobe experience manager
git-repo: https://github.com/AdobeDocs/experience-manager-htl.zh-Hant
index: y
recommendations: noDisplay
source-git-commit: 22f62868df0fcfc558e5d62434dde843a9f3ca83
workflow-type: tm+mt
source-wordcount: '81'
ht-degree: 40%

---


# 內部使用的中繼資料

GitHub製作系統會以階層方式定義中繼資料，並增加前一項的層級，如下所示：

1. metadata.md
1. ToC
1. 文章

metadata.md檔案中定義的中繼資料會套用至整個存放庫，但可以在ToC和文章層級覆寫。 中繼資料的任何覆寫都應該盡量在最低層級進行。

`experience-manager-core-components.en`存放庫中的中繼資料是最低要求。

metadata.md

* `product`
* `git-repo`
* `index: y`

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
* `index: n` (僅適用於舊版元件)

