---
title: 设置所需的开发环境 [!DNL Asset Compute Service]
description: 开发人员环境设置 [!DNL Asset Compute Service] 以开始创建和测试自定义代码。
exl-id: 91c12889-01d8-4757-9bdd-f73c491cd9d5
source-git-commit: 2dde177933477dc9ac2ff5a55af1fd2366e18359
workflow-type: tm+mt
source-wordcount: '357'
ht-degree: 1%

---

# 设置开发人员环境 {#create-dev-environment}

创建允许开发 [!DNL Asset Compute Service]，请按照以下要求和说明操作。

1. [获取访问权和凭据](https://developer.adobe.com/app-builder/docs/getting_started/#acquire-access-and-credentials) 对象 [!DNL Adobe Developer App Builder].

1. [设置本地环境](https://developer.adobe.com/app-builder/docs/getting_started/#local-environment-set-up) 以及所需的工具。

1. 还有其他一些工具可帮助您开始顺利开发，这些工具包括：

   * [Git](https://git-scm.com/)
   * [Docker桌面](https://www.docker.com/get-started)
   * [NodeJS](https://nodejs.org) （v14 LTS，不建议使用奇数版本）和 [NPM](https://www.npmjs.com). OSX HomeBrew的用户可以 `brew install node` 以同时安装这两个组件。 否则，请从以下位置下载： [NodeJS下载页面](https://nodejs.org/en/)
   * 我们建议使用一个对NodeJS有益的IDE [Visual Studio Code (VS Code)](https://code.visualstudio.com) 因为它是调试器支持的IDE。 可以将任何其他IDE用作代码编辑器，但是尚不支持高级用法（例如，调试器）
   * 安装最新的[[!DNL Adobe I/O] CLI](https://github.com/adobe/aio-cli) (`aio`)

   <!-- - install using `npm install -g @adobe/aio-cli@7.1.0` -->

1. 确保符合 [先决条件](/help/understand-extensibility.md#prerequisites-and-provisioning)

<!--
>[!NOTE]
>
>For now, use [!DNL Adobe I/O] CLI v7.1.0 of and do not use [!DNL Adobe I/O] CLI v8.
-->

## 设置App Builder项目 {#create-App-Builder-project}

1. 确保系统管理员或开发人员角色位于 [!DNL Experience Cloud] 组织。 这由系统管理员在 [Admin Console](https://adminconsole.adobe.com/overview).

1. 登录 [Adobe Developer控制台](https://console.adobe.io/). 确保您属于同一组 [!DNL Experience Cloud] 组织作为 [!DNL Experience Manager] as a [!DNL Cloud Service] 集成。 有关Adobe Developer控制台的更多信息，请参阅 [控制台文档](https://www.adobe.io/apis/experienceplatform/console/docs.html).

1. [创建App Builder项目](https://developer.adobe.com/app-builder/docs/getting_started/first_app/). 单击 **[!UICONTROL 创建新项目]** > **[!UICONTROL 模板中的项目]**. 选择App Builder。 它会创建一个新的App Builder项目，该项目有两个工作区： `Production` 和 `Stage`. 添加其他工作区，例如 `Development`，根据需要。

1. 在App Builder项目中，选择一个工作区并订阅Asset compute所需的服务。 单击 **添加到项目** > **API** 并添加 `Asset Compute`， `IO Events`、和 `IO Events Management` 服务。 添加第一个API时，它会提示创建私钥。 将此信息保存在计算机上，因为您需要此密钥才能使用开发人员工具测试自定义应用程序。

## 下一步 {#next-step}

现在您的环境已设置完毕，您可以 [创建自定义应用程序](develop-custom-application.md).

<!-- More ideas:
 
* Any steps in the beginning that lead to gotchas later should be called out for caution? For example,
  * don't change some defaults initially
  * know risks when deviating from standard path
  * naming conventions to follow
  * Retrieve and format credentials (YAML file details)

TBD: When aio-cli v8 bugs are resolved, update the AIO CLI install command to remove v7.x reference and instruct users to use the latest version. See CQDOC-18346.

-->
