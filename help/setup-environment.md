---
title: 设置 [!DNL Asset Compute Service]所需的开发环境。
description: ' [!DNL Asset Compute Service] 的开发人员环境设置用于开始创建和测试自定义代码。'
translation-type: tm+mt
source-git-commit: 0ecf2c044c7847029fb10ce37f4b831c58b93fe9
workflow-type: tm+mt
source-wordcount: '373'
ht-degree: 0%

---


# 设置开发者环境{#create-dev-environment}

要创建允许您为[!DNL Asset Compute Service]进行开发的设置，请按照以下要求和说明操作。

1. [获取Project Firefly的](https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/setup.md#acquire-access-and-credentials) 访问权限和凭据。

1. [设置本地环](https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/setup.md#local-environment-set-up) 境和所需的工具。

1. 帮助您顺利开始开发的其他工具有：

   * [蠢货](https://git-scm.com/)。
   * [Docker Desktop](https://www.docker.com/get-started)。
   * [NodeJS](https://nodejs.org) （v10到v12 LTS，不推荐奇数版本）和 [NPM](https://www.npmjs.com)。OSX HomeBrew的用户可以执行`brew install node`以安装这两个软件。 否则，从[NodeJS下载页面](https://nodejs.org/en/)下载。
   * 对于NodeJS，我们建议使用[Visual Studio代码（VS代码）](https://code.visualstudio.com)作为调试器支持的IDE，因为它是IDE。 您可以将任何其他IDE用作代码编辑器，但尚不支持高级用法（如调试器）。
   * [AIO CLI](https://github.com/adobe/aio-cli) (`aio`)-使用安装 `npm install -g @adobe/aio-cli`。

1. 确保满足[先决条件](/help/understand-extensibility.md#prerequisites-and-provisioning)。

## 设置Firefly项目{#create-firefly-project}

1. 在体验组织中授予系统管理员或开发人员角色访问权限。 这可由系统管理员在[Admin Console](https://adminconsole.adobe.com/overview)中设置。

1. 登录[Adobe开发者控制台](https://console.adobe.io/)。 确保您是作为[!DNL Cloud Service]集成的[!DNL Experience Manager]的同一Adobe Experience Cloud组织的一部分。 有关Adobe开发者控制台的详细信息，请参阅[控制台文档](https://www.adobe.io/apis/experienceplatform/console/docs.html)。

1. [创建Firefly项目](https://www.adobe.io/apis/experienceplatform/project-firefly/docs.html#!AdobeDocs/project-firefly/master/getting_started/first_app.md)。单击&#x200B;**[!UICONTROL 从模板]**&#x200B;创建新项目&#x200B;]**>**[!UICONTROL &#x200B;项目。 选择“萤火虫”。 它创建一个新的Firefly项目，其中具有两个工作区：`Production`和`Stage`。 根据需要添加其他工作区，例如`Development`。

1. 在Firefly项目中，选择一个工作区并订阅Asset compute所需的服务。 单击&#x200B;**添加到Project** > **API**&#x200B;并添加`Asset Compute`、`IO Events`和`IO Events Management`服务。 添加第一个API时，将提示创建私钥。 在您需要此密钥时将此信息保存到您的计算机上，以便使用开发人员工具测试您的自定义应用程序。

## 下一步{#next-step}

环境设置完毕后，您便可以[创建自定义应用程序](develop-custom-application.md)。

<!-- TBD items for later:
 
* Any steps in the beginning that lead to gotchas later should be called out for caution? For example,
  * don't change some defaults initially
  * know risks when deviating from standard path
  * naming conventions to follow
  * Retrieve and format credentials (YAML file details)
-->
