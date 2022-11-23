---
title: 部署 [!DNL Asset Compute Service] 自定义应用程序
description: 部署 [!DNL Asset Compute Service] 自定义应用程序。
exl-id: a68d4f59-8a8f-43b2-8bc6-19320ac8c9ef
source-git-commit: 129651ba432b75703bc27baa7081da60302f828d
workflow-type: tm+mt
source-wordcount: '184'
ht-degree: 3%

---

# 部署自定义应用程序 {#deploy-custom-application}

要部署您的应用程序，请使用 [aio应用程序部署](https://github.com/adobe/aio-cli#aio-appdeploy) 命令。 在终端中，命令显示一个用于访问自定义应用程序的URL。 URL的格式为 `https://[namespace].adobeio-static.net/api/v1/web/[appname]-[appversion]/[workername]`.

要在不重新部署应用程序的情况下获取相同的URL，请使用 [`aio app get-url`](https://github.com/adobe/aio-cli#aio-app-get-url-action) 命令。

在 [处理配置文件(位于 [!DNL Experience Manager] as a [!DNL Cloud Service]](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html) 将应用程序与 [!DNL Experience Manager] as a [!DNL Cloud Service].

确保您的App Builder项目和工作区与 [!DNL Experience Manager] as a [!DNL Cloud Service] 要在其中使用操作的环境。 它有不同的开发、暂存和生产环境。 您可以通过检查 `AIO_runtime_*` 在ENV文件中在Firefly应用程序根目录下定义的凭据。 例如，要部署到 `Stage` 工作区， `AIO_runtime_namespace` 格式为 `xxxxxx_xxxxxxxxx_stage`. 与集成 [!DNL Experience Manager] as a [!DNL Cloud Service] 生产环境，使用Firefly中的应用程序URL `Production` 工作区。

>[!CAUTION]
>
>请勿在关键 [!DNL Experience Manager] 环境。

>[!MORELIKETHIS]
>
>* [在中了解和管理环境 [!DNL Experience Manager] as a [!DNL Cloud Service]](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/implementing/using-cloud-manager/manage-environments.html).

