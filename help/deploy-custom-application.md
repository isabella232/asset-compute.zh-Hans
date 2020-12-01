---
title: 部署 [!DNL Asset Compute Service] 自定义应用程序。
description: 部署 [!DNL Asset Compute Service] 自定义应用程序。
translation-type: tm+mt
source-git-commit: 79630efa8cee2c8919d11e9bb3c14ee4ef54d0f3
workflow-type: tm+mt
source-wordcount: '197'
ht-degree: 0%

---


# 部署自定义应用程序{#deploy-custom-application}

要部署应用程序，请使用[aio app deploy](https://github.com/adobe/aio-cli#aio-appdeploy)命令。 在终端中，该命令显示一个URL以访问自定义应用程序。 URL的格式为`https://[namespace].adobeio-static.net/api/v1/web/[appname]-[appversion]/[workername]`。

要获得相同的URL而不重新部署应用程序，请使用[`aio app get-url`](https://github.com/adobe/aio-cli#aio-appget-url-action)命令。

将[处理用户档案中的URL用作Cloud Service](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html)，将应用程序与[!DNL Experience Manager]作为Cloud Service集成。

确保您的Firefly项目和工作区与[!DNL Experience Manager]对应，作为您要使用操作的Cloud Service环境。 它具有不同的开发、分阶段和生产环境。 您可以通过检查`AIO_runtime_*`凭据来验证环境，这些凭据是在Firefly应用程序根目录的ENV文件中定义的。 例如，要部署到`Stage`工作区，`AIO_runtime_namespace`的格式为`xxxxxx_xxxxxxxxx_stage`。 要作为Cloud Service生产环境与[!DNL Experience Manager]集成，请使用Firefly `Production`工作区中的应用程序URL。

>[!CAUTION]
>
>请勿在关键[!DNL Experience Manager]环境上使用个人工作区。

>[!MORELIKETHIS]
>
>* [以环境的身份理解和管理Experience Manager](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/implementing/using-cloud-manager/manage-environments.html)。

