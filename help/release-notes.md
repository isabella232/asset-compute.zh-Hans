---
title: 发行说明 [!DNL Asset Compute Service]。
description: 中的新增功能、增强功能和已知问题 [!DNL Asset Compute Service]。
translation-type: tm+mt
source-git-commit: 68d910cd092fccb599c361f24daff80460129e1c
workflow-type: tm+mt
source-wordcount: '193'
ht-degree: 0%

---


# 发行说明 [!DNL Asset Compute Service] {#release-notes}

最新版本 [!DNL Asset Compute Service] 将于2020年7月30日发布。

<!--

To test your custom applications with the [developer tool](https://github.com/adobe/asset-compute-devtool), you need access to a [cloud storage container](https://github.com/adobe/asset-compute-devtool#prerequisites). Currently, Adobe supports Azure Blob Storage and AWS S3.

>[!NOTE]
>
>Cloud storage access is only required for using the developer tool. You can still create, test and deploy custom applications with out using the developer tool.
-->

## What is new {#what-is-new}

这是第一个版本 [!DNL Asset Compute Service]。 它是处理数字资产的可扩展 [!DNL Adobe Experience Cloud] 和可扩展服务。 它可以将图像、视频、文档和其他文件格式转换为不同的再现，包括缩略图、提取的文本和元数据以及存档。

目前， [!DNL Asset Compute Service] 只能用作 [!DNL Experience Manager] Cloud Service。

## 限制和已知问题 {#known-limitations}

要使用开发人员工具测试 [您的自定义应](https://github.com/adobe/asset-compute-devtool)用程序，您需要访问 [云存储容器](https://github.com/adobe/asset-compute-devtool#prerequisites)。

* 只有开发人员工具才需 [!DNL Experience Manager] 要云存储（不同于Blob存储）访问。 您仍然可以创建、测试和部署自定义应用程序，而无需开发人员工具。
* 它可以是不同项目中的多个开发人员使用的共享容器。

## Contribute {#contribute-open-source}

[!DNL Asset Compute Service] 可扩展性是在github.com/adobe上的一个开放开发模型下开 [发的](https://github.com/adobe) ，该模型欢迎扩展开发人员的贡献。 与开发、创建、测试和部署自定义应用程序相关的所有组件都是开放源。 了 [解如何和在何处为计算服务做出贡献](contribute-to-compute-service.md)。

<!-- **TBD:**
* Are we versioning the releases?
* Is there any compatibility information to be added? With Project Firefly versions, or AEMaaCS releases, or other offerings/integrations such as InDesign Server?
-->
