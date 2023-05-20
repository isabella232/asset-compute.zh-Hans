---
title: 架構 [!DNL Asset Compute Service]
description: 如何 [!DNL Asset Compute Service] API、應用程式和SDK共同合作，提供雲端原生資產處理服務。
exl-id: 658ee4b7-5eb1-4109-b263-1b7d705e49d6
source-git-commit: 0c5ab8ab230e3f033b804849a2c005351542155f
workflow-type: tm+mt
source-wordcount: '486'
ht-degree: 0%

---

# 架構 [!DNL Asset Compute Service] {#overview}

此 [!DNL Asset Compute Service] 建置在無伺服器上 [!DNL Adobe I/O] 執行階段平台。 它提供資產的Adobe Sensei內容服務支援。 叫用使用者端(僅限 [!DNL Experience Manager] as a [!DNL Cloud Service] 支援)，並隨附其為資產尋找的Adobe Sensei產生資訊。 傳回的資訊為JSON格式。

[!DNL Asset Compute Service] 可藉由建立自訂應用程式來擴充 [!DNL Project Adobe Developer App Builder]. 這些自訂應用程式是 [!DNL Project Adobe Developer App Builder] Headless應用程式並執行工作，例如新增自訂轉換工具或呼叫外部API以執行影像操作。

[!DNL Project Adobe Developer App Builder] 是在上建置和部署自訂Web應用程式的架構 [!DNL Adobe I/O] 執行階段。 若要建立自訂應用程式，開發人員可以運用 [!DNL React Spectrum] (Adobe的UI toolkit)、建立微服務、建立自訂事件，以及協調API。 另請參閱 [Adobe Developer App Builder檔案](https://developer.adobe.com/app-builder/docs/overview).

此架構所根據的基礎包括：

* 應用程式的模組化（僅包含特定工作所需的功能）可讓應用程式彼此分離，並保持輕量級。

* 無伺服器概念 [!DNL Adobe I/O] 執行階段可帶來許多好處：非同步、高度可擴充、獨立、以工作為基礎的處理，非常適合資產處理。

* 二進位雲端儲存提供必要功能，可個別儲存及存取資產檔案和轉譯，而不需要儲存裝置的完整存取許可權（使用預先簽署的URL參照）。 傳輸加速、CDN快取，以及將運算應用程式與雲端儲存空間並置在一起，都可讓您以最佳方式存取低延遲的內容。 支援AWS和Azure雲端。

![asset compute服務架構](assets/architecture-diagram.png)

*圖：架構 [!DNL Asset Compute Service] 以及它如何與整合 [!DNL Experience Manager]、儲存和處理應用程式。*

架構包含下列部分：

* **API和協調層** 會接收要求（JSON格式），指示服務將來源資產轉換為多個轉譯。 這些請求為非同步請求，且會以啟用ID （即作業ID）傳回。 指示純粹是宣告式的，對於所有標準處理工作（例如縮圖產生、文字擷取），消費者只會指定想要的結果，不會指定處理特定轉譯的應用程式。 一般API功能（例如驗證、分析、速率限制）會使用服務前方的AdobeAPI閘道來處理，並管理傳送至以下位置的所有請求： [!DNL Adobe I/O] 執行階段。 應用程式路由由協調層動態完成。 使用者端可為特定轉譯指定自訂應用程式，並包含自訂引數。 應用程式執行可以完全平行化，因為它們是中不同的無伺服器函式 [!DNL Adobe I/O] 執行階段。

* **處理資產的應用程式** 專精於特定型別的檔案格式或目標轉譯。 從概念上講，應用程式就像Unix管道概念：輸入檔案會轉換為一個或多個輸出檔案。

* **A [通用應用程式庫](https://github.com/adobe/asset-compute-sdk)** 處理下載來源檔案、上傳轉譯、錯誤報告、事件傳送和監控等常見工作。 如此設計可讓應用程式的開發遵循無伺服器概念，儘可能保持簡單，並可限製為本機檔案系統互動。

<!-- TBD:

* About the YAML file?
* minimize description to custom applications
* remove all internal stuff (e.g. Photoshop application, API Gateway) from text and diagram
* update diagram to focus on 3rd party custom applications ONLY
* Explain important transactions/handshakes?
* Flow of assets/control? See the illustration on the Nui diagrams wiki.
* Illustrations. See the SVG shared by Alex.
* Exceptions? Limitations? Call-outs? Gotchas?
* Do we want to add what basic processing is not available currently, that is expected by existing AEM customers?
-->
