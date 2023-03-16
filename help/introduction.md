---
title: 简介 [!DNL Asset Compute Service]
description: '"[!DNL Asset Compute Service] 是一项云原生资产处理服务，可降低复杂性并提高可扩展性。”'
exl-id: f8c89f65-5a94-44f3-aaac-4612ae291101
source-git-commit: 2dde177933477dc9ac2ff5a55af1fd2366e18359
workflow-type: tm+mt
source-wordcount: '309'
ht-degree: 6%

---

# 概述 [!DNL Asset Compute Service] {#overview}

[!DNL Asset Compute Service] 是的 [!DNL Adobe Experience Cloud] 处理数字资产。 它可以将图像、视频、文档和其他文件格式转换为不同的演绎版，包括缩略图、提取的文本和元数据，以及存档。

开发人员可以插件自定义资产应用程序（也称为自定义工作程序），以解决自定义用例。 服务在 [!DNL Adobe I/O] 运行时。 它可以通过 [!DNL Adobe Developer App Builder] 使用Node.js编写的无头应用程序。 这些API可以执行自定义操作，例如调用外部API来执行图像操作或利用 [!DNL Adobe Sensei] 支持。

[!DNL Adobe Developer App Builder] 是在 [!DNL Adobe I/O] 运行时扩展Adobe Experience Cloud解决方案。 要创建自定义应用程序，开发人员可以利用 [!DNL React Spectrum] (Adobe的UI工具包)、创建微服务、创建自定义事件和编排API。 请参阅 [Adobe Developer App Builder文档](https://developer.adobe.com/app-builder/docs/overview/).

>[!NOTE]
>
>目前， [!DNL Asset Compute Service] 只能通过 [!DNL Experience Manager] as a [!DNL Cloud Service]. 管理员创建可以调用 [!DNL Asset Compute Service] 以传递资产以进行处理。 请参阅 [使用资产微服务和处理配置文件](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html).

## 支持的用例 [!DNL Asset Compute Service] {#possible-use-cases-benefits}

[!DNL Asset Compute Service] 支持一些常见的业务用例，如基本图像处理；Adobe应用程序特定转化；和自定义应用程序创建，以编排复杂的业务要求。

您可以使用 [!DNL Asset Compute] web服务，为不同文件类型生成缩略图，为 [支持的文件格式](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/file-format-support.html). 请参阅 [通过自定义配置支持的用例](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html).

>[!NOTE]
>
>该服务不提供资产存储。 用户会提供并引用云存储中的源文件和演绎版文件位置。

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
>* [资产微服务在 [!DNL Adobe Experience Manager] as a [!DNL Cloud Service]](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/asset-microservices-overview.html).
>* [Adobe Developer App Builder文档](https://developer.adobe.com/app-builder/docs/overview).
>* [支持处理的文件格式](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/file-format-support.html).


<!-- **TBD:**
* Clarify the service can only be used within AEM as Cloud Service. The docs provided as context for custom application developers. Not to be used as a standalone service.
  ** and API as that plays a role in custom applications (accepting standard params, invoking Nui itself in the future, etc. (this is an outlook))

* link to aem as cloud service docs on asset ingestion and customization with processing profiles.
-->
