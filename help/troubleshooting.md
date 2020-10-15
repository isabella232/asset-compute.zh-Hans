---
title: 故障诊断 [!DNL Asset Compute Service].
description: 使用排除故障和调试自定义应用程序 [!DNL Asset Compute Service]。
translation-type: tm+mt
source-git-commit: 68d910cd092fccb599c361f24daff80460129e1c
workflow-type: tm+mt
source-wordcount: '306'
ht-degree: 1%

---


# 故障诊断 {#troubleshoot}

一些有助于您对资产计算服务进行故障诊断的通用疑难解答提示包括：

* 确保JavaScript应用程序在启动时不会崩溃。 此类崩溃通常与缺少的库或依赖项相关。
* 确保要安装的所有依赖关系在应用程序的文件中被引 `package.json` 用。
* 确保故障清除可能导致的任何错误不会生成隐藏原始问题的错误。

* 首次通过新集成启动开发人员工具时，它可 [!DNL Asset Compute Service] 能会失败第一个处理请求，因为资产计算事件日志可能未完全设置。 请等待一段时间，日志设置，然后再发送其他请求。
* 如果在发送资产计算或请 `/register` 求时 `/process` 遇到错误，请确保将所有必要的API添加到AdobeI/O项目和工作区，即资产计算、IO事件、IO事件管理和运行时。

## 通过AdobeI/O CLI登录问题 {#login-via-aio-cli}

如果通过AdobeI/O CLI登录 [!DNL Adobe Developer Console] 时 [遇到问题](https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/first_app.md#3-signing-in-from-cli)，请手动添加开发、测试和部署自定义应用程序所需的凭据：

1. 在Adobe开发人员控制台上导航到 [您的Firefly项目](https://console.adobe.io/)**[!UICONTROL 和工作区]** ，然后按右上角的“下载”。 打开此文件，并将此JSON保存到您计算机上的安全位置。

1. 导览至Firefly应用程序中的ENV文件。

1. 添加Adobe I/O Runtime凭据。 从下载的JSON中获取Adobe I/O Runtime凭据。 凭据在下 `project.workspace.services.runtime`。 在变量中添加I/O运行时凭 `AIO_runtime_XXX` 据：

   ```json
   AIO_runtime_auth=
   AIO_runtime_namespace=
   ```

1. 在步骤1中添加下载的JSON的绝对路径：

   ```json
       ASSET_COMPUTE_INTEGRATION_FILE_PATH=
   ```

1. 设置开发人员工 [具所需](develop-custom-application.md) 的其余凭据。

<!-- TBD for later:
Add any best practices for developers in this section:
* Any items to take care of when creating projects.
* Any naming conventions, reserved keywords, etc.?
* Any terms that can become a source of confusion later based on our OOTB naming.

* If required, add limitations for custom applications and spin those off as best practices.
* Do NOT borrow any content from https://git.corp.adobe.com/nui/nui/blob/master/doc/worker_api.md. It is outdated and irrelevant for 3rd party custom applications.
-->
