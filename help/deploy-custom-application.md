---
title: 部署 [!DNL Asset Compute Service] 自定义应用程序
description: 部署 [!DNL Asset Compute Service] 自定义应用程序。
exl-id: a68d4f59-8a8f-43b2-8bc6-19320ac8c9ef
source-git-commit: 187a788d036f33b361a0fd1ca34a854daeb4a101
workflow-type: tm+mt
source-wordcount: '183'
ht-degree: 0%

---

# 部署自定义应用程序{#deploy-custom-application}

要部署应用程序，请使用[aio app deploy](https://github.com/adobe/aio-cli#aio-appdeploy)命令。 在终端中，命令显示一个用于访问自定义应用程序的URL。 URL的格式为`https://[namespace].adobeio-static.net/api/v1/web/[appname]-[appversion]/[workername]`。

要在不重新部署应用程序的情况下获取相同的URL，请使用[`aio app get-url`](https://github.com/adobe/aio-cli#aio-appget-url-action)命令。

使用 [!DNL Experience Manager] as a [!DNL Cloud Service]](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html)中的[处理配置文件中的URL，将您的应用程序与[!DNL Experience Manager] as a [!DNL Cloud Service]集成。

确保您的Firefly项目和工作区与[!DNL Experience Manager]作为[!DNL Cloud Service]环境对应，您可以在该环境中使用操作。 它有不同的开发、暂存和生产环境。 您可以通过检查`AIO_runtime_*`凭据来验证环境，该凭据是在Firefly应用程序根目录的ENV文件中定义的。 例如，要部署到`Stage`工作区，`AIO_runtime_namespace`的格式为`xxxxxx_xxxxxxxxx_stage`。 要与[!DNL Experience Manager]作为[!DNL Cloud Service]生产环境进行集成，请使用Firefly `Production`工作区中的应用程序URL。

>[!CAUTION]
>
>请勿在关键的[!DNL Experience Manager]环境中使用个人工作区。

>[!MORELIKETHIS]
>
>* [了解和管理 [!DNL Experience Manager] 环境 [!DNL Cloud Service]](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/implementing/using-cloud-manager/manage-environments.html)。

