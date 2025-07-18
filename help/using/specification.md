---
title: HTL 規格
description: 請參閱HTL規格以取得詳細的語法資訊。
exl-id: c0657476-4db6-4fad-ad87-9252b5003237
source-git-commit: addc69e4b4e56a9b1c5f91ce9af26fa2d326d981
workflow-type: tm+mt
source-wordcount: '134'
ht-degree: 62%

---


# HTL 規格 {#htl-specification}

HTML 範本語言是首選且推薦使用的 HTML 伺服器端範本系統。

## HTL 層 {#layers}

您可以在 AEM 中透過多個層級來定義 HTL。

1. **[HTL 規格](https://github.com/adobe/htl-spec)** - HTL 的規格是開放原始碼且不受平台限制，任何人均可自由實施。其規格由其 GitHub 存放庫維護。
1. **[Sling HTL Scripting Engine](https://sling.apache.org/documentation/bundles/scripting/scripting-htl.html)** - `Sling`專案已建立HTL的參考實作，由AEM使用。 `Sling`專案會維護其檔案。
1. **[AEM擴充功能](aem-extensions.md)** - AEM以`Sling` HTL Scripting Engine為基礎進行擴充，為開發人員提供AEM專用的便利功能。 本文件集也包含這些擴充功能。

瀏覽上述連結，前往 AEM 使用的所有 HTL 層的專屬文件。
