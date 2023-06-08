---
title: 疑难解答 [!DNL Asset Compute Service]
description: 使用以下方式排除自定义应用程序故障并调试 [!DNL Asset Compute Service].
exl-id: 017fff91-e5e9-4a30-babf-5faa1ebefc2f
source-git-commit: 5257e091730f3672c46dfbe45c3e697a6555e6b1
workflow-type: tm+mt
source-wordcount: '291'
ht-degree: 1%

---

# 疑难解答 {#troubleshoot}

以下是一般故障排除提示，可帮助您排除Asset compute服务故障：

* 确保JavaScript应用程序在启动时不会崩溃。 此类崩溃通常与缺少库或依赖项有关。
* 确保要安装的所有依赖项在应用程序的 `package.json` 文件。
* 确保任何可能因失败时清理而导致的错误不会生成隐藏原始问题的自身错误。

* 首次使用新启动开发人员工具时 [!DNL Asset Compute Service] 集成，如果未完全设置Asset compute事件日志，则可能会使第一个处理请求失败。 请等待一段时间以设置日志，然后再发送另一个请求。
* 如果您在发送Asset compute时遇到错误 `/register` 或 `/process` 请求时，确保所有必需的API都已添加到 [!DNL Adobe I/O] 项目和工作区 — 即Asset compute， [!DNL Adobe I/O] 活动， [!DNL Adobe I/O] 事件管理，以及 [!DNL Adobe I/O] 运行时。

## 通过以下方式登录问题 [!DNL Adobe I/O] CLI {#login-via-aio-cli}

如果您在登录 [!DNL Adobe Developer Console] [通过 [!DNL Adobe I/O] CLI](https://developer.adobe.com/app-builder/docs/getting_started/first_app/#3-signing-in-from-cli)，然后手动添加开发、测试和部署自定义应用程序所需的凭据：

1. 在上导航到您的Adobe Developer App Builder项目和工作区。 [Adobe Developer控制台](https://console.adobe.io/) 并按 **[!UICONTROL 下载]** 从右上角。 打开此文件并将此JSON保存到计算机上的安全位置。

1. 导航到Adobe Developer App Builder应用程序中的环境文件。

1. 添加 [!DNL Adobe I/O] 运行时凭据。 获取 [!DNL Adobe I/O] 下载的JSON中的运行时凭据。 凭据位于 `project.workspace.services.runtime`. 添加 [!DNL Adobe I/O] 中的运行时凭据 `AIO_runtime_XXX` 变量：

   ```json
   AIO_runtime_auth=
   AIO_runtime_namespace=
   ```

1. 在步骤1中添加已下载JSON的绝对路径：

   ```json
       ASSET_COMPUTE_INTEGRATION_FILE_PATH=
   ```

1. 设置其余的 [所需的凭据](develop-custom-application.md) 开发人员工具所需。

<!-- TBD for later:
Add any best practices for developers in this section:
* Any items to take care of when creating projects.
* Any naming conventions, reserved keywords, etc.?
* Any terms that can become a source of confusion later based on our OOTB naming.

* If required, add limitations for custom applications and spin those off as best practices.
* Do NOT borrow any content from https://git.corp.adobe.com/nui/nui/blob/master/doc/worker_api.md. It is outdated and irrelevant for 3rd party custom applications.
-->
