---
title: ' [!DNL Asset Compute Service]的架构。'
description: 如何 [!DNL Asset Compute Service] API、应用程序和SDK协同工作来提供云本机资产处理服务。
translation-type: tm+mt
source-git-commit: 0fb256f7d9f83fbae564d9fd52ee6b2f34c5d7e9
workflow-type: tm+mt
source-wordcount: '496'
ht-degree: 0%

---


# [!DNL Asset Compute Service] {#overview}的架构

[!DNL Asset Compute Service]构建在无服务器的Adobe I/O Runtime平台上。 它为资产提供Adobe Sensei内容服务支持。 调用的客户端(仅支持[!DNL Experience Manager]作为Cloud Service)会提供其为资产寻找的由Adobe Sensei生成的信息。 返回的信息采用JSON格式。

[!DNL Asset Compute Service] 可通过基于创建自定义应用程序来扩展 [!DNL Project Firefly]。这些自定义应用程序是[!DNL Project Firefly]无外设应用程序，并执行任务，如添加自定义转换工具或调用外部API以执行图像操作。

[!DNL Project Firefly] 是在运行时构建和部署自定义Web应用程序的 [!DNL Adobe I/O] 框架。要创建自定义应用程序，开发人员可以利用[!DNL React Spectrum](Adobe的UI工具包)、创建微服务、创建自定义事件和编排API。 请参阅[Project Firefly](https://www.adobe.io/apis/experienceplatform/project-firefly/docs.html)的文档。

该架构所基于的基础包括：

* 应用程序的模块性——只包含给定任务所需的功能——允许应用程序彼此分离并保持轻量级。

* Adobe I/O Runtime的无服务器概念带来了诸多好处：异步、高度可扩展、隔离、基于作业的处理，非常适合资产处理。

* 二进制云存储提供了单独存储和访问资产文件和演绎版所必需的功能，无需使用预签名的URL引用对该存储具有完全访问权限。 借助云存储，传输加速、CDN缓存和共同定位计算应用程序可实现最佳低延迟内容访问。 支持AWS和Azure云。

![asset compute服务架构](assets/architecture-diagram.png)

*图：它的架 [!DNL Asset Compute Service] 构及其与应用程序、 [!DNL Experience Manager]存储和处理应用程序的集成方式。*

该架构由以下部分组成：

* **API和业务流程** 层接收请求（JSON格式），指示服务将源资产转换为多个演绎版。这些请求是异步的，并返回激活id（又称“作业id”）。 说明仅是声明性的，对于所有标准处理工作(如缩略图生成、文本提取)，用户只指定所需的结果，而不指定处理某些演绎版的应用程序。 通用的API功能（如身份验证、分析、速率限制）使用服务前的AdobeAPI网关进行处理，并管理所有转至I/O运行时的请求。 应用程序路由由业务流程层动态完成。 客户端可以为特定再现指定自定义应用程序并包含自定义参数。 应用程序执行可以完全并行，因为它们是I/O运行时中独立的无服务器函数。

* **用于处理专门** 处理某些类型的文件格式或目标再现的资产的应用程序。从概念上讲，应用程序与Unix管道概念类似：输入文件被转换为一个或多个输出文件。

* **通用 [应用程](https://github.com/adobe/asset-compute-sdk)** 序库处理常见任务，如下载源文件、上传演绎版、错误报告、事件发送和监视。这样，开发应用程序就会尽可能简单，遵循无服务器的思想，并且可限于本地文件系统交互。

<!-- TBD:

* About the YAML file?
* See [https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/first_app.md#5-anatomy-of-a-project-firefly-application](https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/first_app.md#5-anatomy-of-a-project-firefly-application).

* minimize description to custom applications
* remove all internal stuff (e.g. Photoshop application, API Gateway) from text and diagram
* update diagram to focus on 3rd party custom applications ONLY
* Explain important transactions/handshakes?
* Flow of assets/control? See the illustration on the Nui diagrams wiki.
* Illustrations. See the SVG shared by Alex.
* Exceptions? Limitations? Call-outs? Gotchas?
* Do we want to add what basic processing is not available currently, that is expected by existing AEM customers?
-->
