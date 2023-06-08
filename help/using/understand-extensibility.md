---
title: 了解扩展 [!DNL Asset Compute Service]
description: 何时以及如何扩展 [!DNL Asset Compute Service] 用于自定义资产处理的功能。
exl-id: 3b903364-34cc-44d5-9a03-24a0102cf85d
source-git-commit: 5257e091730f3672c46dfbe45c3e697a6555e6b1
workflow-type: tm+mt
source-wordcount: '260'
ht-degree: 5%

---

# 可扩展性简介 {#introduction-to-extensibilty}

许多演绎版要求（如转换为格式和调整图像大小）都由来解决 [处理配置文件 [!DNL Experience Manager] as a [!DNL Cloud Service]](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/asset-microservices-overview.html). 更复杂的业务需求可能需要一个适合组织需求的定制解决方案。 [!DNL Asset Compute Service] 可以通过创建从中的处理配置文件调用的自定义应用程序来扩展 [!DNL Experience Manager]. 这些自定义应用程序适用于 [支持的用例](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html).

>[!NOTE]
>
>[!DNL Asset Compute Service] 仅适用于以下情况： [!DNL Experience Manager] as a [!DNL Cloud Service].

自定义应用程序是Headless [Adobe Developer App Builder](https://github.com/AdobeDocs/app-builder) 应用程序。 扩展 [!DNL Asset Compute Service] 通过简化自定义应用程序 [ASSET COMPUTESDK](https://github.com/adobe/asset-compute-sdk) 和Adobe Developer App Builder开发人员工具。 这使开发人员能够专注于业务逻辑。 创建自定义应用程序与创建普通的无服务器应用程序一样简单 [!DNL Adobe I/O] 运行时操作。 它是单个Node.js JavaScript函数。 此 [基本自定义应用程序示例](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-basic/worker-basic.js) 说明了。

## 先决条件和配置要求 {#prerequisites-and-provisioning}

确保满足以下先决条件：

* 您的计算机上已安装Adobe Developer App Builder工具。
* An [!DNL Experience Cloud] 组织。 更多信息 [此处](https://developer.adobe.com/app-builder/docs/getting_started/#acquire-access-and-credentials).
* 体验组织必须具有 [!DNL Experience Manager] as a [!DNL Cloud Service] 已启用。
* [!DNL Adobe Experience Cloud] 组织属于 [!DNL Adobe Developer App Builder] 开发人员预览计划。 参见 [如何申请访问权限](https://developer.adobe.com/app-builder/docs/overview/getting_access).
* 确保开发人员在组织中拥有开发人员角色或管理员权限。
* 确保 [[!DNL Adobe I/O] CLI](https://github.com/adobe/aio-cli) 本地安装。

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
