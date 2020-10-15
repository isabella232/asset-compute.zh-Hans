---
title: 部署 [!DNL Asset Compute Service] 自定义应用程序。
description: 部署 [!DNL Asset Compute Service] 自定义应用程序。
translation-type: tm+mt
source-git-commit: 1c2a1dc41296bf26c432c51b5afa20cb07a4c5c5
workflow-type: tm+mt
source-wordcount: '201'
ht-degree: 8%

---


# 部署自定义应用程序 {#deploy-custom-application}

要部署应用程序，请使 [用aio app deploy](https://github.com/adobe/aio-cli#aio-appdeploy) 命令。 在终端中，该命令显示一个URL以访问自定义应用程序。 URL的格式为 `https://[namespace].adobeio-static.net/api/v1/web/[appname]-[appversion]/[workername]`。

要获得相同的URL而不重新部署应用程序，请使用 [`aio app get-url`](https://github.com/adobe/aio-cli#aio-appget-url-action) 命令。

在Experience Manager的处理 [用户档案中使用URL作为Cloud Service](https://docs.adobe.com/content/help/zh-Hans/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html) ，将应用程序 [!DNL Experience Manager] 与Cloud Service集成。

确保您的Firefly项目和工作区与要 [!DNL Experience Manager] 在其中使用动作的Cloud Service环境相对应。 它具有不同的开发、分阶段和生产环境。 您可以通过检查在Firefly应 `AIO_runtime_*` 用程序根目录的ENV文件中定义的凭据来验证环境。 例如，要部署到工 `Stage` 作区， `AIO_runtime_namespace` 其格式为 `xxxxxx_xxxxxxxxx_stage`。 要作为Cloud Service [!DNL Experience Manager] 生产环境与集成，请使用Firefly工作区中的应用程序 `Production` URL。

>[!CAUTION]
>
>切勿在关键环境上使用个人工 [!DNL Experience Manager] 作区。

>[!MORELIKETHIS]
>
>* [以环境的身份理解和管理Experience Manager](https://docs.adobe.com/content/help/zh-Hans/experience-manager-cloud-service/implementing/using-cloud-manager/manage-environments.html)。

