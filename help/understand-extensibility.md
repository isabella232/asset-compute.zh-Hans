---
title: 了解有关扩展的信息 [!DNL Asset Compute Service]。
description: 何时以及如何扩展 [!DNL Asset Compute Service] 功能以进行自定义资产处理。
translation-type: tm+mt
source-git-commit: d26ae470507e187249a472ececf5f08d803a636c
workflow-type: tm+mt
source-wordcount: '259'
ht-degree: 0%

---


# 可扩展性{#introduction-to-extensibilty}简介

 [!DNL Experience Manager] 中的“处理”用户档案(a [!DNL Cloud Service]](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/asset-microservices-overview.html))解决了许多再现要求，如转换为格式和调整图像大小。 [更复杂的业务需求可能需要定制的解决方案来满足组织的需求。 [!DNL Asset Compute Service] 可以通过创建从中的处理用户档案调用的自定义应用程序进行扩展 [!DNL Experience Manager]。这些自定义应用程序满足[支持的用例](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html)的要求。

>[!NOTE]
>
>[!DNL Asset Compute Service] 仅可与作 [!DNL Experience Manager] 为一起使 [!DNL Cloud Service]用

自定义应用程序是无标题[Project Firefly](https://github.com/AdobeDocs/project-firefly)应用程序。 通过[Asset computeSDK](https://github.com/adobe/asset-compute-sdk)和Project Firefly开发人员工具，使用自定义应用程序扩展[!DNL Asset Compute Service]变得简单。 这使开发人员能专注于业务逻辑。 创建自定义应用程序与创建无服务器的普通[!DNL Adobe I/O]运行时操作一样简单。 它是单个Node.js JavaScript函数。 [基本的自定义应用程序示例](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-basic/worker-basic.js)说明了这一点。

## 先决条件和配置要求{#prerequisites-and-provisioning}

确保满足以下先决条件：

* 您的计算机上已安装项目Firefly工具。
* [!DNL Experience Cloud]组织。 更多信息[此处](https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/setup.md#acquire-access-and-credentials)。
* 体验组织必须启用[!DNL Experience Manager]作为[!DNL Cloud Service]。
* [!DNL Adobe Experience Cloud] 组织是开发人员预览 [!DNL Project Firefly] 项目的一部分。请参阅[如何应用访问](https://github.com/AdobeDocs/project-firefly/blob/master/overview/getting_access.md)。
* 确保开发人员在组织中具有开发人员角色或管理员权限。
* 确保[[!DNL Adobe I/O] CLI](https://github.com/adobe/aio-cli)已本地安装。

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
