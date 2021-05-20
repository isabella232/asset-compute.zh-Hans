---
title: 了解有关扩展的 [!DNL Asset Compute Service]
description: 何时以及如何扩展 [!DNL Asset Compute Service] 功能以进行自定义资产处理。
exl-id: 3b903364-34cc-44d5-9a03-24a0102cf85d
source-git-commit: 187a788d036f33b361a0fd1ca34a854daeb4a101
workflow-type: tm+mt
source-wordcount: '259'
ht-degree: 0%

---

# 扩展性简介{#introduction-to-extensibilty}

许多再现要求（如转换为格式和调整图像大小）都由 [!DNL Experience Manager] 中的“处理配置文件”作为 [!DNL Cloud Service]](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/asset-microservices-overview.html)来处理。 [更复杂的业务需求可能需要定制的解决方案，以满足组织的需求。 [!DNL Asset Compute Service] 可以通过创建从中的处理配置文件调用的自定义应用程序进行 [!DNL Experience Manager]扩展。这些自定义应用程序符合[支持的用例](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html)的要求。

>[!NOTE]
>
>[!DNL Asset Compute Service] 只能与as a一 [!DNL Experience Manager] 起使 [!DNL Cloud Service]用

自定义应用程序是无标题的[项目Firefly](https://github.com/AdobeDocs/project-firefly)应用程序。 通过[Asset computeSDK](https://github.com/adobe/asset-compute-sdk)和项目Firefly开发人员工具，可以使用自定义应用程序扩展[!DNL Asset Compute Service]变得简单。 这允许开发人员专注于业务逻辑。 创建自定义应用程序与创建无服务器的纯运行时操作一样简单。 [!DNL Adobe I/O]它是一个Node.js JavaScript函数。 [基本自定义应用程序示例](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-basic/worker-basic.js)说明了该示例。

## 先决条件和配置要求{#prerequisites-and-provisioning}

确保满足以下先决条件：

* 项目Firefly工具安装在您的计算机上。
* [!DNL Experience Cloud]组织。 更多信息[此处](https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/setup.md#acquire-access-and-credentials)。
* 体验组织必须启用[!DNL Experience Manager]作为[!DNL Cloud Service]。
* [!DNL Adobe Experience Cloud] 组织是开发人员预 [!DNL Project Firefly] 览计划的一部分。请参阅[如何应用访问](https://github.com/AdobeDocs/project-firefly/blob/master/overview/getting_access.md)。
* 确保开发人员在组织中具有开发人员角色或管理员权限。
* 确保本地安装了[[!DNL Adobe I/O] CLI](https://github.com/adobe/aio-cli)。

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
