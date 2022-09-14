---
title: 设置 [!DNL Asset Compute Service]
description: 的开发人员环境设置 [!DNL Asset Compute Service] 以开始创建和测试自定义代码。
exl-id: 91c12889-01d8-4757-9bdd-f73c491cd9d5
source-git-commit: a50a3bdb520cbe608c5710716df80ac6e3b486e5
workflow-type: tm+mt
source-wordcount: '364'
ht-degree: 1%

---

# 设置开发人员环境 {#create-dev-environment}

要创建设置，以便您能够 [!DNL Asset Compute Service]，请按照以下要求和说明操作。

1. [获取访问和凭据](https://www.adobe.io/project-firefly/docs/getting_started/#acquire-access-and-credentials) 表示 [!DNL Project Firefly].

1. [设置本地环境](https://www.adobe.io/project-firefly/docs/getting_started/#local-environment-set-up) 和所需工具。

1. 还有一些工具可帮助您开始顺利开发：

   * [Git](https://git-scm.com/)
   * [Docker Desktop](https://www.docker.com/get-started)
   * [NodeJS](https://nodejs.org) （v12 - v14 LTS，不推荐奇数版本）和 [NPM](https://www.npmjs.com). OSX HomeBrew的用户可以 `brew install node` 来安装这两个组件。 否则，请从 [NodeJS下载页面](https://nodejs.org/en/)
   * 对于适用于NodeJS的IDE，我们建议 [Visual Studio代码（VS代码）](https://code.visualstudio.com) 因为调试器支持IDE。 您可以将任何其他IDE用作代码编辑器，但尚不支持高级用法（例如调试器）
   * 安装最新[[!DNL Adobe I/O] CLI](https://github.com/adobe/aio-cli) (`aio`)

   <!-- - install using `npm install -g @adobe/aio-cli@7.1.0` -->

1. 确保遇到 [先决条件](/help/understand-extensibility.md#prerequisites-and-provisioning)

<!--
>[!NOTE]
>
>For now, use [!DNL Adobe I/O] CLI v7.1.0 of and do not use [!DNL Adobe I/O] CLI v8.
-->

## 设置应用程序生成器项目 {#create-App-Builder-project}

1. 确保在 [!DNL Experience Cloud] 组织。 这由系统管理员在 [Admin Console](https://adminconsole.adobe.com/overview).

1. 登录 [Adobe Developer控制台](https://console.adobe.io/). 确保您属于同一 [!DNL Experience Cloud] 组织 [!DNL Experience Manager] as a [!DNL Cloud Service] 集成。 有关Adobe Developer Console的更多信息，请参阅 [控制台文档](https://www.adobe.io/apis/experienceplatform/console/docs.html).

1. [创建应用程序生成器项目](https://www.adobe.io/apis/experienceplatform/project-firefly/docs.html#!AdobeDocs/project-firefly/master/getting_started/first_app.md). 单击 **[!UICONTROL 创建新项目]** > **[!UICONTROL 模板中的项目]**. 选择“应用程序生成器”。 它会创建一个具有两个工作区的新应用程序生成器项目： `Production` 和 `Stage`. 添加其他工作区，例如 `Development`，视需要。

1. 在应用程序生成器项目中，选择一个工作区并订阅Asset compute所需的服务。 单击 **添加到项目** > **API** 添加 `Asset Compute`, `IO Events`和 `IO Events Management` 服务。 添加第一个API时，将提示创建私钥。 在您需要此密钥时，将此信息保存在您的计算机上，以便使用开发人员工具测试您的自定义应用程序。

## 下一步 {#next-step}

现在，您的环境已设置完成，您可以 [创建自定义应用程序](develop-custom-application.md).

<!-- More ideas:
 
* Any steps in the beginning that lead to gotchas later should be called out for caution? For example,
  * don't change some defaults initially
  * know risks when deviating from standard path
  * naming conventions to follow
  * Retrieve and format credentials (YAML file details)

TBD: When aio-cli v8 bugs are resolved, update the AIO CLI install command to remove v7.x reference and instruct users to use the latest version. See CQDOC-18346.

-->
