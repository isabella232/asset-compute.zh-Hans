---
title: ' [!DNL Asset Compute Service]简介'
description: '[!DNL Asset Compute Service] 是一项云原生资产处理服务，可降低复杂性并提高可扩展性。'
exl-id: f8c89f65-5a94-44f3-aaac-4612ae291101
source-git-commit: a2460a0719f8c585ed72e44c904aa0df33301032
workflow-type: tm+mt
source-wordcount: '307'
ht-degree: 0%

---

# [!DNL Asset Compute Service]概述 {#overview}

[!DNL Asset Compute Service] 是一项用于处理数字资产的可 [!DNL Adobe Experience Cloud] 缩放且可扩展的服务。它可以将图像、视频、文档和其他文件格式转换为不同的演绎版，包括缩略图、提取的文本和元数据，以及存档。

开发人员可以插件自定义资产应用程序（也称为自定义工作程序），以解决自定义用例。 该服务在[!DNL Adobe I/O]运行时运行。 它可通过使用Node.js编写的[!DNL Project Firefly]无头应用程序进行扩展。 这些API可以执行自定义操作，例如调用外部API来执行图像操作或利用[!DNL Adobe Sensei]支持。

[!DNL Project Firefly] 是一个框架，用于在运行时构建和部署自定义web应用程 [!DNL Adobe I/O] 序以扩展Adobe Experience Cloud解决方案。要创建自定义应用程序，开发人员可以利用[!DNL React Spectrum](Adobe的UI工具包)、创建微服务、创建自定义事件和编排API。 请参阅项目Firefly](https://www.adobe.io/apis/experienceplatform/project-firefly/docs.html)的[文档。

>[!NOTE]
>
>目前，[!DNL Asset Compute Service]只能通过[!DNL Experience Manager]作为[!DNL Cloud Service]使用。 管理员可创建处理配置文件，以调用[!DNL Asset Compute Service]来传递要处理的资产。 请参阅[使用资产微服务和处理配置文件](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html)。

## 支持的[!DNL Asset Compute Service]用例 {#possible-use-cases-benefits}

[!DNL Asset Compute Service] 支持一些常见的业务用例，如基本图像处理；Adobe应用程序特定转化；和自定义应用程序创建，以编排复杂的业务要求。

您可以使用[!DNL Asset Compute] Web服务为不同文件类型生成缩略图，为[支持的文件格式](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/file-format-support.html)生成高质量的图像渲染。 请参阅通过自定义配置支持的[用例](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html)。

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
>* [资产微服务资产处理 [!DNL Adobe Experience Manager] 概述a [!DNL Cloud Service]](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/asset-microservices-overview.html)。
>* [项目Firefly的文档](https://www.adobe.io/apis/experienceplatform/project-firefly/docs.html)。
>* [支持处理的文件格式](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/file-format-support.html)。


<!-- **TBD:**
* Clarify the service can only be used within AEM as Cloud Service. The docs provided as context for custom application developers. Not to be used as a standalone service.
  ** and API as that plays a role in custom applications (accepting standard params, invoking Nui itself in the future, etc. (this is an outlook))

* link to aem as cloud service docs on asset ingestion and customization with processing profiles.
-->
