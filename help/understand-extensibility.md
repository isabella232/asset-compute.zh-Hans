---
title: 了解扩展 [!DNL Asset Compute Service]。
description: 何时以及如何扩展功 [!DNL Asset Compute Service] 能以进行自定义资产处理。
translation-type: tm+mt
source-git-commit: 54afa44d8d662ee1499a385f504fca073ab6c347
workflow-type: tm+mt
source-wordcount: '275'
ht-degree: 3%

---


# 可扩展性简介 {#introduction-to-extensibilty}

许多再现要求（如转换为格式和调整图像大小）都由处 [理用户档案 [!DNL Experience Manager] 作为Cloud Service解决](https://docs.adobe.com/content/help/en/experience-manager-cloud-service/assets/asset-microservices-overview.html)。 更复杂的业务需求可能需要定制的解决方案来满足组织的需求。 [!DNL Asset Compute Service] 可以通过创建从中的处理用户档案调用的自定义应用程序进行扩展 [!DNL Experience Manager]。 这些自定义应用程序适合 [支持的用例](https://docs.adobe.com/content/help/zh-Hans/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html)。

>[!NOTE]
>
>[!DNL Asset Compute Service] 只能与Cloud Service [!DNL Experience Manager] 一起使用。

自定义应用程序是无头 [项目Firefly应用](https://github.com/AdobeDocs/project-firefly) 程序。 通过 [!DNL Asset Compute Service] Asset Compute SDK和Project Firefly开发人员工 [具，使用自定义应用程](https://github.com/adobe/asset-compute-sdk) 序进行扩展变得非常简单。 这使开发人员能专注于业务逻辑。 创建自定义应用程序与创建无服务器的简单Adobe I/O Runtime操作一样简单。 它是单个Node.js JavaScript函数。 基本 [的自定义应用程序示例](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-basic/worker-basic.js) 说明了这一点。

## 先决条件和配置要求 {#prerequisites-and-provisioning}

确保满足以下先决条件：

* 您的计算机上已安装项目Firefly工具。
* 组 [!DNL Experience Cloud] 织。 此处提供更 [多信息](https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/setup.md#acquire-access-and-credentials)。
* 体验组织必须 [!DNL Experience Manager] 启用Cloud Service。
* [!DNL Adobe Experience Cloud] 组织是开发人员预览 [!DNL Project Firefly] 项目的一部分。 了 [解如何应用访问](https://github.com/AdobeDocs/project-firefly/blob/master/overview/getting_access.md)。
* 确保开发人员在组织中具有开发人员角色或管理员权限。
* 确保 [AdobeI/O CLI已](https://github.com/adobe/aio-cli) 本地安装。

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
