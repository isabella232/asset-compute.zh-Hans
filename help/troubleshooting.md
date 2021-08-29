---
title: 故障诊断 [!DNL Asset Compute Service]
description: 使用 [!DNL Asset Compute Service]对自定义应用程序进行故障诊断和调试。
exl-id: 017fff91-e5e9-4a30-babf-5faa1ebefc2f
source-git-commit: eed9da4b20fe37a4e44ba270c197505b50cfe77f
workflow-type: tm+mt
source-wordcount: '285'
ht-degree: 1%

---

# 故障诊断 {#troubleshoot}

一些可能有助于您对Asset compute服务进行故障诊断的通用故障诊断提示包括：

* 确保JavaScript应用程序在启动时不会崩溃。 此类崩溃通常与缺少的库或依赖项相关。
* 确保要安装的所有依赖项都在应用程序的`package.json`文件中引用。
* 确保在失败时清除可能导致的任何错误不会生成它们自己隐藏原始问题的错误。

* 首次使用新的[!DNL Asset Compute Service]集成启动开发人员工具时，如果Asset compute事件日志未完全设置，则首次处理请求可能会失败。 等待一段时间以设置日志，然后再发送另一个请求。
* 如果在发送Asset compute`/register`或`/process`请求时遇到错误，请确保将所有必需的API添加到[!DNL Adobe I/O]项目和工作区 — 即Asset compute、[!DNL Adobe I/O]事件、[!DNL Adobe I/O]事件管理和[!DNL Adobe I/O]运行时。

## 通过[!DNL Adobe I/O] CLI登录问题 {#login-via-aio-cli}

如果通过 [!DNL Adobe I/O] CLI](https://www.adobe.io/project-firefly/docs/getting_started/first_app/#3-signing-in-from-cli)登录[!DNL Adobe Developer Console] [时遇到问题，请手动添加开发、测试和部署自定义应用程序所需的凭据：

1. 在[Adobe开发人员控制台](https://console.adobe.io/)上导航到您的Firefly项目和工作区，然后按右上角的&#x200B;**[!UICONTROL Download]**。 打开此文件，并将此JSON保存到您计算机上的安全位置。

1. 导航到Firefly应用程序中的ENV文件。

1. 添加[!DNL Adobe I/O]运行时凭据。 从下载的JSON中获取[!DNL Adobe I/O] A Runtime凭据。 凭据位于`project.workspace.services.runtime`下。 在`AIO_runtime_XXX`变量中添加[!DNL Adobe I/O]运行时凭据：

   ```json
   AIO_runtime_auth=
   AIO_runtime_namespace=
   ```

1. 在步骤1中，添加已下载JSON的绝对路径：

   ```json
       ASSET_COMPUTE_INTEGRATION_FILE_PATH=
   ```

1. 设置开发人员工具所需的[必需凭据](develop-custom-application.md)的其余部分。

<!-- TBD for later:
Add any best practices for developers in this section:
* Any items to take care of when creating projects.
* Any naming conventions, reserved keywords, etc.?
* Any terms that can become a source of confusion later based on our OOTB naming.

* If required, add limitations for custom applications and spin those off as best practices.
* Do NOT borrow any content from https://git.corp.adobe.com/nui/nui/blob/master/doc/worker_api.md. It is outdated and irrelevant for 3rd party custom applications.
-->
