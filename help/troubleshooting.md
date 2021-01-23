---
title: 故障诊断 [!DNL Asset Compute Service]
description: 使用 [!DNL Asset Compute Service]对自定义应用程序进行疑难解答和调试。
translation-type: tm+mt
source-git-commit: 95e384d2a298b3237d4f93673161272744e7f44a
workflow-type: tm+mt
source-wordcount: '288'
ht-degree: 1%

---


# 故障诊断 {#troubleshoot}

一些可能有助于您对Asset compute服务进行故障诊断的通用疑难解答提示包括：

* 确保JavaScript应用程序在启动时不会崩溃。 此类崩溃通常与缺少的库或依赖项相关。
* 确保要安装的所有依赖关系在应用程序的`package.json`文件中被引用。
* 确保故障清除可能导致的任何错误不会生成隐藏原始问题的错误。

* 首次通过新的[!DNL Asset Compute Service]集成启动开发者工具时，它可能会失败第一个处理请求，因为Asset compute事件日志可能未完全设置。 请等待一段时间，日志设置，然后再发送其他请求。
* 如果发送Asset compute`/register`或`/process`请求时出错，请确保将所有必要的API添加到[!DNL Adobe I/O]项目和工作区，即Asset compute、[!DNL Adobe I/O]事件、[!DNL Adobe I/O]事件管理和[!DNL Adobe I/O]运行时。

## 通过[!DNL Adobe I/O] CLI {#login-via-aio-cli}登录问题

如果通过 [!DNL Adobe I/O] CLI](https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/first_app.md#3-signing-in-from-cli)登录到[!DNL Adobe Developer Console] [时遇到问题，请手动添加开发、测试和部署自定义应用程序所需的凭据：

1. 在[Adobe开发者控制台](https://console.adobe.io/)上导航到您的Firefly项目和工作区，然后按右上角的&#x200B;**[!UICONTROL 下载]**。 打开此文件，并将此JSON保存到您计算机上的安全位置。

1. 导览至Firefly应用程序中的ENV文件。

1. 添加[!DNL Adobe I/O]运行时凭据。 从下载的JSON中获取[!DNL Adobe I/O] A Runtime凭据。 凭据位于`project.workspace.services.runtime`下。 在`AIO_runtime_XXX`变量中添加[!DNL Adobe I/O]运行时凭据：

   ```json
   AIO_runtime_auth=
   AIO_runtime_namespace=
   ```

1. 在步骤1中添加下载的JSON的绝对路径：

   ```json
       ASSET_COMPUTE_INTEGRATION_FILE_PATH=
   ```

1. 设置开发人员工具所需的其余[凭据](develop-custom-application.md)。

<!-- TBD for later:
Add any best practices for developers in this section:
* Any items to take care of when creating projects.
* Any naming conventions, reserved keywords, etc.?
* Any terms that can become a source of confusion later based on our OOTB naming.

* If required, add limitations for custom applications and spin those off as best practices.
* Do NOT borrow any content from https://git.corp.adobe.com/nui/nui/blob/master/doc/worker_api.md. It is outdated and irrelevant for 3rd party custom applications.
-->
