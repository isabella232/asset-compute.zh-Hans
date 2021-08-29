---
title: ' [!DNL Asset Compute Service]的架构'
description: 如何 [!DNL Asset Compute Service] API、应用程序和SDK协同工作来提供云原生资产处理服务。
exl-id: 658ee4b7-5eb1-4109-b263-1b7d705e49d6
source-git-commit: eed9da4b20fe37a4e44ba270c197505b50cfe77f
workflow-type: tm+mt
source-wordcount: '485'
ht-degree: 0%

---

# [!DNL Asset Compute Service]的架构 {#overview}

[!DNL Asset Compute Service]构建在无服务器[!DNL Adobe I/O]运行时平台之上。 它为资产提供Adobe Sensei内容服务支持。 调用客户端（仅支持[!DNL Experience Manager]作为[!DNL Cloud Service]）提供了它为资产搜索的Adobe Sensei生成信息。 返回的信息采用JSON格式。

[!DNL Asset Compute Service] 可通过基于创建自定义应用程序来扩 [!DNL Project Firefly]展。这些自定义应用程序是[!DNL Project Firefly]无头应用程序，执行添加自定义转换工具或调用外部API等任务以执行图像操作。

[!DNL Project Firefly] 是在运行时构建和部署自定义web应用程序的 [!DNL Adobe I/O] 框架。要创建自定义应用程序，开发人员可以利用[!DNL React Spectrum](Adobe的UI工具包)、创建微服务、创建自定义事件和编排API。 请参阅项目Firefly](https://www.adobe.io/apis/experienceplatform/project-firefly/docs.html)的[文档。

架构所基于的基础包括：

* 应用程序的模块化（仅包含给定任务所需的内容）允许将应用程序相互分离，并保持轻量级。

* [!DNL Adobe I/O]运行时的无服务器概念带来许多好处：异步、高度可扩展、隔离、基于作业的处理，非常适合进行资产处理。

* 二进制云存储提供了单独存储和访问资产文件和演绎版的必要功能，无需使用预签名URL引用来完全访问存储。 传输加速、CDN缓存以及与云存储共享的计算应用程序，可实现最佳的低延迟内容访问。 支持AWS和Azure云。

![asset compute服务的架构](assets/architecture-diagram.png)

*图：其体系结构 [!DNL Asset Compute Service] 以及它如何与、存储 [!DNL Experience Manager]和处理应用程序集成。*

架构包含以下部分：

* **API和编排层** 接收指示服务将源资产转换为多个演绎版的请求（JSON格式）。请求是异步的，并会通过激活ID（即作业ID）返回。 说明是纯声明性的，对于所有标准处理工作（如缩略图生成、文本提取），用户只指定所需的结果，而不指定处理某些呈现的应用程序。 通用API功能（如身份验证、分析、速率限制）使用服务前的AdobeAPI网关进行处理，并管理到[!DNL Adobe I/O]运行时的所有请求。 应用程序路由由编排层动态完成。 客户端可以为特定演绎版指定自定义应用程序，并包含自定义参数。 应用程序执行可以完全并行，因为它们是[!DNL Adobe I/O]运行时中单独的无服务器函数。

* **用于处理专门** 处理特定类型文件格式或目标呈现的资产的应用程序。从概念上讲，应用程序与Unix管道概念类似：输入文件被转换为一个或多个输出文件。

* **常用 [应用程](https://github.com/adobe/asset-compute-sdk)** 序库可处理常见任务，如下载源文件、上传演绎版、错误报告、事件发送和监控。这样，开发应用程序就会尽可能地简单，遵循无服务器的思想，并且可以限制在本地文件系统交互。

<!-- TBD:

* About the YAML file?
* See [https://www.adobe.io/project-firefly/docs/getting_started/first_app/#5-anatomy-of-a-project-firefly-application](https://www.adobe.io/project-firefly/docs/getting_started/first_app/#5-anatomy-of-a-project-firefly-application).

* minimize description to custom applications
* remove all internal stuff (e.g. Photoshop application, API Gateway) from text and diagram
* update diagram to focus on 3rd party custom applications ONLY
* Explain important transactions/handshakes?
* Flow of assets/control? See the illustration on the Nui diagrams wiki.
* Illustrations. See the SVG shared by Alex.
* Exceptions? Limitations? Call-outs? Gotchas?
* Do we want to add what basic processing is not available currently, that is expected by existing AEM customers?
-->
