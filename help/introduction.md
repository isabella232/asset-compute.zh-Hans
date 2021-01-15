---
title: ' [!DNL Asset Compute Service]简介。'
description: '[!DNL Asset Compute Service] 是一种云本机资产处理服务，可降低复杂性并提高可扩展性。'
translation-type: tm+mt
source-git-commit: d26ae470507e187249a472ececf5f08d803a636c
workflow-type: tm+mt
source-wordcount: '314'
ht-degree: 0%

---


# [!DNL Asset Compute Service] {#overview}概述

[!DNL Asset Compute Service] 是处理数字资产的可扩展 [!DNL Adobe Experience Cloud] 和可扩展服务。它可以将图像、视频、文档和其他文件格式转换为不同的再现，包括缩略图、提取的文本和元数据以及存档。

开发人员可以插入自定义资源应用程序（也称为自定义工作器）以解决自定义用例。 该服务在[!DNL Adobe I/O]运行时上工作。 它可通过写入Node.js的[!DNL Project Firefly]无头应用程序扩展。 这些操作可以执行自定义操作，如调用外部API执行图像操作或利用[!DNL Adobe Sensei]支持。

[!DNL Project Firefly] 是在运行时构建和部署自定义web应用程序以扩 [!DNL Adobe I/O] 展Adobe Experience Cloud解决方案的框架。要创建自定义应用程序，开发人员可以利用[!DNL React Spectrum](Adobe的UI工具包)、创建微服务、创建自定义事件和编排API。 请参阅[Project Firefly](https://www.adobe.io/apis/experienceplatform/project-firefly/docs.html)的文档。

>[!NOTE]
>
>目前，[!DNL Asset Compute Service]只能通过[!DNL Experience Manager]作为[!DNL Cloud Service]使用。 管理员创建可调用[!DNL Asset Compute Service]以传递要处理的资源的处理用户档案。 请参阅[使用资产微服务和处理用户档案](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html)。

## 支持[!DNL Asset Compute Service] {#possible-use-cases-benefits}的用例

[!DNL Asset Compute Service] 支持基本图像处理等一些常见的业务用例；Adobe应用程序特定转换；和自定义应用程序创建，它们可以协调复杂的业务要求。

可以使用[!DNL Asset Compute] Web服务为不同文件类型生成缩略图，为[支持的文件格式](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/file-format-support.html)生成高质量图像渲染。 请参阅通过自定义配置](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html)支持的[用例。

>[!NOTE]
>
>服务不提供资产存储。 用户提供它，并在云存储中提供对源和再现文件位置的引用。

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
>* [资产微型服务资产处理 [!DNL Adobe Experience Manager] 概述a [!DNL Cloud Service]](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/asset-microservices-overview.html)。
>* [Project Firefly的文档](https://www.adobe.io/apis/experienceplatform/project-firefly/docs.html)。
>* [支持处理的文件格式](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/file-format-support.html)。
>* [asset compute服务发行说明](release-notes.md)


<!-- **TBD:**
* Clarify the service can only be used within AEM as Cloud Service. The docs provided as context for custom application developers. Not to be used as a standalone service.
  ** and API as that plays a role in custom applications (accepting standard params, invoking Nui itself in the future, etc. (this is an outlook))

* link to aem as cloud service docs on asset ingestion and customization with processing profiles.
-->
