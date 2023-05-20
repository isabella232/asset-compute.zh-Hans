---
title: 簡介 [!DNL Asset Compute Service]
description: '"[!DNL Asset Compute Service] 是雲端原生資產處理服務，可降低複雜性並改善擴充性。」'
exl-id: f8c89f65-5a94-44f3-aaac-4612ae291101
source-git-commit: 2dde177933477dc9ac2ff5a55af1fd2366e18359
workflow-type: tm+mt
source-wordcount: '309'
ht-degree: 6%

---

# 概述 [!DNL Asset Compute Service] {#overview}

[!DNL Asset Compute Service] 是可擴充的服務，適用於 [!DNL Adobe Experience Cloud] 以處理數位資產。 它可以將影像、視訊、檔案和其他檔案格式轉換為不同的轉譯專案，包括縮圖、擷取的文字和中繼資料以及封存。

開發人員可以外掛自訂資產應用程式（也稱為自訂背景工作），以解決自訂使用案例。 此服務適用於 [!DNL Adobe I/O] 執行階段。 可延伸至 [!DNL Adobe Developer App Builder] 以Node.js撰寫的Headless應用程式。 這些可執行自訂操作，例如呼叫外部API以執行影像操作或利用 [!DNL Adobe Sensei] 支援。

[!DNL Adobe Developer App Builder] 是在上建置和部署自訂Web應用程式的架構 [!DNL Adobe I/O] 執行階段以擴充Adobe Experience Cloud解決方案。 若要建立自訂應用程式，開發人員可以運用 [!DNL React Spectrum] (Adobe的UI toolkit)、建立微服務、建立自訂事件，以及協調API。 另請參閱 [Adobe Developer App Builder檔案](https://developer.adobe.com/app-builder/docs/overview/).

>[!NOTE]
>
>目前， [!DNL Asset Compute Service] 只能透過 [!DNL Experience Manager] as a [!DNL Cloud Service]. 管理員會建立處理設定檔，以呼叫 [!DNL Asset Compute Service] 以傳遞資產以供處理。 另請參閱 [使用資產微服務和處理設定檔](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html).

## 支援的使用案例 [!DNL Asset Compute Service] {#possible-use-cases-benefits}

[!DNL Asset Compute Service] 支援一些常見的業務使用案例，例如基本影像處理、Adobe應用程式特定轉換，以及建立可協調複雜業務需求的自訂應用程式。

您可以使用 [!DNL Asset Compute] 可針對不同檔案型別產生縮圖的Web服務、高品質的影像轉譯 [支援的檔案格式](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/file-format-support.html). 另請參閱 [透過自訂設定支援的使用案例](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html).

>[!NOTE]
>
>此服務不提供資產儲存。 使用者會提供該檔案，並參考雲端儲存空間中的來源和轉譯檔案位置。

<!-- TBD: Should this be mentioned in the docs?

|Asset Compute Service does not do this|Expectations from implementing client|
|---|---|
| Binary uploads or API-based asset ingestion. | Use other methods to ingest assets. |
| Store binaries or any persisted data across processing requests.| Each request is independent so treat it as a standalone request by sharing binary and processing instructions. |
| Store any configurations such as processing rules or settings for a user or an organization's account. | Add processing request to each request/instruction. |
| Direct event handling of asset creation events from storage systems and processing completed notifications, and errors. | Use [!DNL Adobe I/O] Events and other methods. |

-->

>[!MORELIKETHIS]
>
>* [使用中的資產微服務進行資產處理概觀 [!DNL Adobe Experience Manager] as a [!DNL Cloud Service]](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/asset-microservices-overview.html).
>* [Adobe Developer App Builder檔案](https://developer.adobe.com/app-builder/docs/overview).
>* [支援處理的檔案格式](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/file-format-support.html).


<!-- **TBD:**
* Clarify the service can only be used within AEM as Cloud Service. The docs provided as context for custom application developers. Not to be used as a standalone service.
  ** and API as that plays a role in custom applications (accepting standard params, invoking Nui itself in the future, etc. (this is an outlook))

* link to aem as cloud service docs on asset ingestion and customization with processing profiles.
-->
