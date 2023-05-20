---
title: 瞭解延伸 [!DNL Asset Compute Service]
description: 何時及如何延伸 [!DNL Asset Compute Service] 進行自訂資產處理的功能。
exl-id: 3b903364-34cc-44d5-9a03-24a0102cf85d
source-git-commit: 2dde177933477dc9ac2ff5a55af1fd2366e18359
workflow-type: tm+mt
source-wordcount: '260'
ht-degree: 5%

---

# 擴充性簡介 {#introduction-to-extensibilty}

許多轉譯需求（例如轉換為格式及調整影像大小）都透過以下方式解決 [處理設定檔於 [!DNL Experience Manager] as a [!DNL Cloud Service]](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/asset-microservices-overview.html). 更複雜的業務需求可能需要適合組織需求的自訂解決方案。 [!DNL Asset Compute Service] 可藉由建立自訂應用程式來擴充，這些應用程式是從中的處理設定檔呼叫的 [!DNL Experience Manager]. 這些自訂應用程式適合 [支援的使用案例](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html).

>[!NOTE]
>
>[!DNL Asset Compute Service] 僅供搭配使用 [!DNL Experience Manager] as a [!DNL Cloud Service].

自訂應用程式為Headless [Adobe Developer App Builder](https://github.com/AdobeDocs/app-builder) 應用程式。 延伸 [!DNL Asset Compute Service] 使用自訂應用程式，可透過 [ASSET COMPUTESDK](https://github.com/adobe/asset-compute-sdk) 和Adobe Developer App Builder開發人員工具。 如此一來，開發人員就能專注於商業邏輯。 建立自訂應用程式，就像建立簡易的無伺服器應用程式一樣簡單 [!DNL Adobe I/O] 執行階段動作。 這是單一Node.js JavaScript函式。 此 [基本自訂應用程式範例](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-basic/worker-basic.js) 加以說明。

## 先決條件和布建需求 {#prerequisites-and-provisioning}

確定您符合下列必要條件：

* 您的電腦上已安裝Adobe Developer App Builder工具。
* 一個 [!DNL Experience Cloud] 組織。 更多資訊 [此處](https://developer.adobe.com/app-builder/docs/getting_started/#acquire-access-and-credentials).
* 體驗組織必須具備 [!DNL Experience Manager] as a [!DNL Cloud Service] 已啟用。
* [!DNL Adobe Experience Cloud] 組織屬於 [!DNL Adobe Developer App Builder] 開發人員預覽計畫。 另請參閱 [如何申請存取權](https://developer.adobe.com/app-builder/docs/overview/getting_access).
* 確保開發人員的組織中有開發人員角色或管理員許可權。
* 確定 [[!DNL Adobe I/O] CLI](https://github.com/adobe/aio-cli) 本機安裝。

<!-- TBD for later:

* What all accesses and licenses are required?
* What all permissions are required to create, debug, and deploy custom applications?
* How do developers get access and provision the required apps?
* What is repository management?
* Anything on security and data transfer?
* What about handling personal or sensitive information?
* Custom application SLA is dependent on SLAs of various services it depends on.
* Document how the devs can get to know the KPIs of their custom applications. The KPIs are dependent on the performance at Adobe's side, amongst other things.
-->
