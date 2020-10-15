---
title: 设置所需的开发环境 [!DNL Asset Compute Service]。
description: 开发人员环境 [!DNL Asset Compute Service] 设置，用于开始创建和测试自定义代码。
translation-type: tm+mt
source-git-commit: 1c2a1dc41296bf26c432c51b5afa20cb07a4c5c5
workflow-type: tm+mt
source-wordcount: '376'
ht-degree: 0%

---


# 设置开发人员环境 {#create-dev-environment}

要创建允许您进行开发的设置，请 [!DNL Asset Compute Service]按照以下要求和说明操作。

1. [获取Project Firefly的访问](https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/setup.md#acquire-access-and-credentials) 和凭据。

1. [设置本地环境](https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/setup.md#local-environment-set-up) 和所需的工具。

1. 帮助您顺利开始开发的其他工具有：

   * [蠢货](https://git-scm.com/)。
   * [Docker Desktop](https://www.docker.com/get-started)。
   * [NodeJS](https://nodejs.org) （v10到v12 LTS，不推荐奇数版本）和 [NPM](https://www.npmjs.com)。 OSX HomeBrew的用户可以安 `brew install node` 装这两种软件。 否则，从NodeJS下 [载页面下载](https://nodejs.org/en/)。
   * 作为一个适用于NodeJS的IDE，我们建议使 [用Visual Studio代码（VS代码）](https://code.visualstudio.com) ，因为它是调试器支持的IDE。 您可以将任何其他IDE用作代码编辑器，但尚不支持高级用法（如调试器）。
   * [AIO CLI](https://github.com/adobe/aio-cli) (`aio`)-使用安装 `npm install -g @adobe/aio-cli`。

1. 确保满足入门 [条件](/help/understand-extensibility.md#prerequisites-and-provisioning)。

## 设置Firefly项目 {#create-firefly-project}

1. 在体验组织中授予系统管理员或开发人员角色访问权限。 这可由Admin Console中的系统管理员设 [置](https://adminconsole.adobe.com/overview)。

1. 登录Adobe开 [发人员控制台](https://console.adobe.io/)。 确保您是与AEM作为Cloud Service集成的同一个Adobe Experience Cloud组织的一部分。 有关Adobe开发者控制台的详细信息，请参阅 [控制台文档](https://www.adobe.io/apis/experienceplatform/console/docs.html)。

1. [创建Firefly项目](https://www.adobe.io/apis/experienceplatform/project-firefly/docs.html#!AdobeDocs/project-firefly/master/getting_started/first_app.md)。 单击 **[!UICONTROL “新建项目]** ”>“ **[!UICONTROL 从模板创建项目”]**。 选择“萤火虫”。 它创建一个新的Firefly项目，其中具有两个工作区： `Production` 和 `Stage`。 添加其他工作区， `Development`例如，根据需要。

1. 在Firefly项目中，选择一个工作区并订阅资产计算所需的服务。 单 **击“添加到** Project **” >** API `Asset Compute`，然后 `IO Events`添加 `IO Events Management` 、和服务。 添加第一个API时，将提示创建私钥。 在您需要此密钥时将此信息保存到您的计算机上，以便使用开发人员工具测试您的自定义应用程序。

## Next step {#next-step}

环境设置完毕后，您便可以创建自 [定义应用程序](develop-custom-application.md)。

<!-- TBD items for later:
 
* Any steps in the beginning that lead to gotchas later should be called out for caution? For example,
  * don't change some defaults initially
  * know risks when deviating from standard path
  * naming conventions to follow
  * Retrieve and format credentials (YAML file details)
-->
