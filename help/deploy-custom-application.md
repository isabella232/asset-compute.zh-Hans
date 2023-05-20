---
title: 部署 [!DNL Asset Compute Service] 自訂應用程式
description: 部署 [!DNL Asset Compute Service] 自訂應用程式。
exl-id: a68d4f59-8a8f-43b2-8bc6-19320ac8c9ef
source-git-commit: 50f69e16772cee7f79a812f2b86f0ef0221db369
workflow-type: tm+mt
source-wordcount: '190'
ht-degree: 3%

---

# 部署自訂應用程式 {#deploy-custom-application}

若要部署您的應用程式，請使用 [aio應用程式部署](https://github.com/adobe/aio-cli#aio-appdeploy) 命令。 在終端機中，命令會顯示存取自訂應用程式的URL。 URL的格式為 `https://[namespace].adobeio-static.net/api/v1/web/[appname]-[appversion]/[workername]`.

若要在不重新部署應用程式的情況下取得相同的URL，請使用 [`aio app get-url`](https://github.com/adobe/aio-cli#aio-app-get-url-action) 命令。

在「 」中使用URL [處理設定檔於 [!DNL Experience Manager] as a [!DNL Cloud Service]](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html) 將您的應用程式與 [!DNL Experience Manager] as a [!DNL Cloud Service].

確認您的App Builder專案和工作區對應至 [!DNL Experience Manager] as a [!DNL Cloud Service] 您想要使用動作的環境。 它有不同的開發、測試和生產環境。 您可以透過檢查來驗證環境 `AIO_runtime_*` 在Adobe Developer App Builder應用程式根目錄的ENV檔案中定義的認證。 例如，若要部署至 `Stage` 工作區， `AIO_runtime_namespace` 為格式 `xxxxxx_xxxxxxxxx_stage`. 若要與整合 [!DNL Experience Manager] as a [!DNL Cloud Service] 生產環境，使用Adobe Developer App Builder中的應用程式URL `Production` 工作區。

>[!CAUTION]
>
>請勿在關鍵事件上使用個人工作區 [!DNL Experience Manager] 環境。

>[!MORELIKETHIS]
>
>* [瞭解並管理中的環境 [!DNL Experience Manager] as a [!DNL Cloud Service]](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/implementing/using-cloud-manager/manage-environments.html).

