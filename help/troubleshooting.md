---
title: 疑难解答 [!DNL Asset Compute Service]
description: 使用对自定义应用程序进行故障诊断和调试 [!DNL Asset Compute Service].
exl-id: 017fff91-e5e9-4a30-babf-5faa1ebefc2f
source-git-commit: 2dde177933477dc9ac2ff5a55af1fd2366e18359
workflow-type: tm+mt
source-wordcount: '291'
ht-degree: 1%

---

# 疑难解答 {#troubleshoot}

一些可能有助于您对Asset compute服务进行故障诊断的通用故障诊断提示包括：

* 确保JavaScript应用程序在启动时不会崩溃。 此类崩溃通常与缺少的库或依赖项相关。
* 确保要安装的所有依赖项都在应用程序的 `package.json` 文件。
* 确保在失败时清除可能导致的任何错误不会生成它们自己隐藏原始问题的错误。

* 首次使用新 [!DNL Asset Compute Service] 集成时，如果“Asset compute事件日记帐”未完全设置，则可能会失败第一个处理请求。 等待一段时间以设置日志，然后再发送另一个请求。
* 如果发送Asset compute时遇到错误 `/register` 或 `/process` 请求，请确保将所有必需的API添加到 [!DNL Adobe I/O] 项目和工作区 — 即Asset compute, [!DNL Adobe I/O] 事件、 [!DNL Adobe I/O] 事件管理和 [!DNL Adobe I/O] 运行时。

## 通过 [!DNL Adobe I/O] CLI {#login-via-aio-cli}

如果您在登录到时遇到问题 [!DNL Adobe Developer Console] [到 [!DNL Adobe I/O] CLI](https://developer.adobe.com/app-builder/docs/getting_started/first_app/#3-signing-in-from-cli)，然后手动添加开发、测试和部署自定义应用程序所需的凭据：

1. 在上导航到Adobe Developer App Builder项目和工作区 [Adobe Developer控制台](https://console.adobe.io/) 按 **[!UICONTROL 下载]** 中。 打开此文件，并将此JSON保存到您计算机上的安全位置。

1. 导航到您的Adobe Developer App Builder应用程序中的ENV文件。

1. 添加 [!DNL Adobe I/O] 运行时凭据。 获取 [!DNL Adobe I/O] 从下载的JSON中获取的运行时凭据。 凭据在下 `project.workspace.services.runtime`. 添加 [!DNL Adobe I/O] 中的运行时凭据 `AIO_runtime_XXX` 变量：

   ```json
   AIO_runtime_auth=
   AIO_runtime_namespace=
   ```

1. 在步骤1中，添加已下载JSON的绝对路径：

   ```json
       ASSET_COMPUTE_INTEGRATION_FILE_PATH=
   ```

1. 设置 [必需凭据](develop-custom-application.md) 开发人员工具所需。

<!-- TBD for later:
Add any best practices for developers in this section:
* Any items to take care of when creating projects.
* Any naming conventions, reserved keywords, etc.?
* Any terms that can become a source of confusion later based on our OOTB naming.

* If required, add limitations for custom applications and spin those off as best practices.
* Do NOT borrow any content from https://git.corp.adobe.com/nui/nui/blob/master/doc/worker_api.md. It is outdated and irrelevant for 3rd party custom applications.
-->
