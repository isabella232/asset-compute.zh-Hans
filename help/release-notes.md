---
title: ' [!DNL Asset Compute Service]的发行说明'
description: ' [!DNL Asset Compute Service]中的新增功能、增强功能和已知问题。'
exl-id: b348fa8f-0cd6-4ca1-bfe3-f31e8d6583f0
source-git-commit: 187a788d036f33b361a0fd1ca34a854daeb4a101
workflow-type: tm+mt
source-wordcount: '191'
ht-degree: 0%

---

# [!DNL Asset Compute Service] {#release-notes}的发行说明

[!DNL Asset Compute Service]的最新版本于2020年7月30日发布。

<!--

To test your custom applications with the [developer tool](https://github.com/adobe/asset-compute-devtool), you need access to a [cloud storage container](https://github.com/adobe/asset-compute-devtool#prerequisites). Currently, Adobe supports Azure Blob Storage and AWS S3.

>[!NOTE]
>
>Cloud storage access is only required for using the developer tool. You can still create, test and deploy custom applications with out using the developer tool.
-->

## {#what-is-new}的新增功能

这是[!DNL Asset Compute Service]的第一个版本。 它是[!DNL Adobe Experience Cloud]的一项用于处理数字资产的可扩展服务。 它可以将图像、视频、文档和其他文件格式转换为不同的演绎版，包括缩略图、提取的文本和元数据，以及存档。

目前，[!DNL Asset Compute Service]只能在[!DNL Experience Manager]中用作[!DNL Cloud Service]。

## 限制和已知问题{#known-limitations}

要使用[开发人员工具](https://github.com/adobe/asset-compute-devtool)测试自定义应用程序，您需要访问[云存储容器](https://github.com/adobe/asset-compute-devtool#prerequisites)。

* 仅开发人员工具需要云存储（与[!DNL Experience Manager] blob存储不同）访问。 您仍然可以创建、测试和部署自定义应用程序，而无需使用开发人员工具。
* 它可以是多个开发人员跨不同项目使用的共享容器。

## Contribute {#contribute-open-source}

[!DNL Asset Compute Service] 可扩展性是在github.com/adobe上的一个开 [放开发模型下开发的，](https://github.com/adobe) 该模型欢迎扩展开发人员的贡献。与开发、创建、测试和部署自定义应用程序相关的所有组件都是开源的。 请参阅[如何以及在何处贡献计算服务](contribute-to-compute-service.md)。

<!-- **TBD:**
* Are we versioning the releases?
* Is there any compatibility information to be added? With Project Firefly versions, or AEMaaCS releases, or other offerings/integrations such as InDesign Server?
-->
