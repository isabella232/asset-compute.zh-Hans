---
title: "[!DNL Adobe Asset Compute Service] 用户指南"
description: 本檔案涵蓋 [!DNL Asset Compute Service] 簡介、如何開發、管理、部署和疑難排解自訂程式碼等工作。
exl-id: 5acf87d1-a391-4802-bfce-e367fc8564df
source-git-commit: 2dde177933477dc9ac2ff5a55af1fd2366e18359
workflow-type: tm+mt
source-wordcount: '222'
ht-degree: 6%

---

# 关于 [!DNL Asset Compute Service]

[!DNL Asset Compute Service] 是可擴充的Adobe Experience Cloud服務，用於處理數位資產。 它可以將影像、影片、檔案和其他檔案格式轉換為不同的轉譯專案，包括縮圖、擷取的文字和中繼資料、封存等等。 開發人員可以外掛自訂應用程式（也稱為自訂工作程式），以處理自訂使用案例，使用建置 [Adobe Developer App Builder](https://developer.adobe.com/app-builder/docs/overview) 並在無伺服器環境中執行 [[!DNL Adobe I/O] 執行階段](https://www.adobe.io/apis/experienceplatform/runtime.html).

本檔案涵蓋 [!DNL Asset Compute Service] 如何開發、管理、部署和疑難排解自訂程式碼等主題。 瞭解內容 [!DNL Asset Compute Service] 方案為，請參閱此 [介紹](introduction.md). 簽出 [此服務可為您做什麼](introduction.md#possible-use-cases-benefits).

[!DNL Asset Compute Service] 支援轉換多種檔案格式，並整合許多Adobe服務。 檢視清單 [支援的檔案格式和服務整合](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/file-format-support.html).

檢視關於以下內容的概述： [資產微服務功能提供於 [!DNL Adobe Experience Manager] as a [!DNL Cloud Service]](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/asset-microservices-overview.html) 以及如何在中使用微服務 [!DNL Experience Manager].

[!DNL Asset Compute Service] 擴充性是在的開放式開發模型下開發 [github.com/adobe](https://github.com/adobe) 歡迎擴充功能開發人員協助撰寫。 所有與開發、建立、測試和部署自訂應用程式相關的元件都是開放原始碼。 另請參閱 [如何以及在何處協助運算服務](contribute-to-compute-service.md).

<!--
Possible to record the below info here in this landing page to centralize the miscellaneous info about Asset Compute Service?
 List of dependencies and requirements SDK, CLI, Devtools, etc.? Or may be a link to the prerequisites.
 Introduction video when Tech Marketing team shares one.
-->
