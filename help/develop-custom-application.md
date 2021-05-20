---
title: 为 [!DNL Asset Compute Service]开发
description: 使用 [!DNL Asset Compute Service]创建自定义应用程序。
exl-id: a0c59752-564b-4bb6-9833-ab7c58a7f38e
source-git-commit: 187a788d036f33b361a0fd1ca34a854daeb4a101
workflow-type: tm+mt
source-wordcount: '1615'
ht-degree: 0%

---

# 开发自定义应用程序 {#develop}

在开始开发自定义应用程序之前：

* 确保满足所有[先决条件](/help/understand-extensibility.md#prerequisites-and-provisioning)。
* 安装所需的[软件工具](/help/setup-environment.md#create-dev-environment)。
* 请参阅[设置环境](setup-environment.md)以确保您已准备好创建自定义应用程序。

## 创建自定义应用程序{#create-custom-application}

确保在本地安装[[!DNL Adobe I/O] CLI](https://github.com/adobe/aio-cli)。

1. 要创建自定义应用程序，请[创建Firefly应用程序](https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/first_app.md#4-bootstrapping-new-app-using-the-cli)。 为此，请在终端中执行`aio app init <app-name>`。

   如果您尚未登录，此命令将提示浏览器要求您使用Adobe ID登录到[Adobe开发人员控制台](https://console.adobe.io/)。 有关从cli登录的详细信息，请参阅[此处](https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/first_app.md#3-signing-in-from-cli)。

   Adobe建议您登录。 如果您遇到问题，请按照说明[创建应用程序，而无需登录](https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/first_app.md#42-developer-is-not-logged-in-as-enterprise-organization-user)。

1. 登录后，按照CLI中的提示操作，选择`Organization`、`Project`和`Workspace`以用于应用程序。 选择在[设置环境](setup-environment.md)时创建的项目和工作区。

   ```sh
   $ aio app init <app-name>
   Retrieving information from [!DNL Adobe I/O] Console.
   ? Select Org My Adobe Org
   ? Select Project MyFireflyProject
   ? Select Workspace myworkspace
   create console.json
   ```

1. 出现`Which Adobe I/O App features do you want to enable for this project?`提示时，选择`Actions`。 请确保取消选择`Web Assets`选项，因为Web资产使用不同的身份验证和授权检查。

   ```bash
   ? Which Adobe I/O App features do you want to enable for this project?
   select components to include (Press <space> to select, <a> to toggle all, <i> to invert selection)
   ❯◉ Actions: Deploy Runtime actions
   ◯ Events: Publish to Adobe I/O Events
   ◯ Web Assets: Deploy hosted static assets
   ◯ CI/CD: Include GitHub Actions based workflows for Build, Test and Deploy
   ```

1. 出现提示`Which type of sample actions do you want to create?`时，请确保选择`Adobe Asset Compute Worker`:

   ```bash
   ? Which type of sample actions do you want to create?
   Select type of actions to generate
   ❯◉ Adobe Asset Compute Worker
   ◯ Generic
   ```

1. 按照其余提示操作，在Visual Studio代码（或您喜爱的代码编辑器）中打开新应用程序。 它包含自定义应用程序的基架和示例代码。

   请在此处阅读有关Firefly应用程序](https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/first_app.md#5-anatomy-of-a-project-firefly-application)的[主要组件。

   模板应用程序利用我们的[Asset computeSDK](https://github.com/adobe/asset-compute-sdk#asset-compute-sdk)来上传、下载和编排应用程序演绎版，因此开发人员只需实施自定义应用程序逻辑。 在`actions/<worker-name>`文件夹内， `index.js`文件是添加自定义应用程序代码的位置。

有关自定义应用程序的示例和想法，请参阅[示例自定义应用程序](#try-sample)。

### 添加凭据{#add-credentials}

当您在创建应用程序时登录时，大多数Firefly凭据都会在您的ENV文件中收集。 但是，使用开发人员工具需要其他凭据。

<!-- TBD: Check if manual setup of credentials is required.
Manual set up of credentials is removed from troubleshooting and best practices page. Link was broken.
If you did not log in, refer to our troubleshooting guide to [set up credentials manually](troubleshooting.md).
-->

#### 开发人员工具存储凭据{#developer-tool-credentials}

使用实际的[!DNL Asset Compute service]测试自定义应用程序的开发人员工具需要一个云存储容器，用于托管测试文件以及接收和显示应用程序生成的演绎版。

>[!NOTE]
>
>这与作为[!DNL Cloud Service]的[!DNL Adobe Experience Manager]的云存储分开。 它仅适用于使用Asset compute开发人员工具进行开发和测试。

确保有权访问[支持的云存储容器](https://github.com/adobe/asset-compute-devtool#prerequisites)。 此容器可由多个开发人员根据需要跨不同项目共享。

#### 将凭据添加到ENV文件{#add-credentials-env-file}

将开发人员工具的以下凭据添加到Firefly项目根目录的ENV文件：

1. 将服务添加到Firefly项目时创建的私钥文件的绝对路径添加到：

   ```conf
   ASSET_COMPUTE_PRIVATE_KEY_FILE_PATH=
   ```

1. 从Adobe开发人员控制台下载文件。 转到项目的根目录，然后单击右上角的“Download All”（全部下载）。 下载的文件文件名为`<namespace>-<workspace>.json`。 执行下列操作之一：

   * 将文件重命名为`console.json`，并将其移到项目的根中。
   * 或者，您也可以选择将绝对路径添加到Adobe开发人员控制台集成JSON文件。 这与在项目工作区中下载的[`console.json`](https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/first_app.md#42-developer-is-not-logged-in-as-enterprise-organization-user)文件相同。

      ```conf
      ASSET_COMPUTE_INTEGRATION_FILE_PATH=
      ```

1. 添加S3或Azure存储凭据。 您只需要访问一个云存储解决方案。

   ```conf
   # S3 credentials
   S3_BUCKET=
   AWS_ACCESS_KEY_ID=
   AWS_SECRET_ACCESS_KEY=
   AWS_REGION=
   
   # Azure Storage credentials
   AZURE_STORAGE_ACCOUNT=
   AZURE_STORAGE_KEY=
   AZURE_STORAGE_CONTAINER_NAME=
   ```

>[!TIP]
>
>`config.json`文件包含凭据。 从项目中，将JSON文件添加到`.gitignore`文件中，以阻止共享该文件。 这同样适用于.env和.aio文件。

## 执行应用程序{#run-custom-application}

在使用Asset compute开发人员工具执行应用程序之前，请正确配置[凭据](#developer-tool-credentials)。

要在开发人员工具中运行应用程序，请使用`aio app run`命令。 它将操作部署到[!DNL Adobe I/O]运行时，并在本地计算机上启动开发工具。 此工具用于在开发过程中测试应用程序请求。 以下是演绎版请求示例：

```json
"renditions": [
    {
        "worker": "https://1234_my_namespace.adobeioruntime.net/api/v1/web/example-custom-worker-master/worker",
        "name": "image.jpg"
    }
]
```

>[!NOTE]
>
>请勿在`run`命令中使用`--local`标记。 它不适用于[!DNL Asset Compute]自定义应用程序和Asset compute开发人员工具。 自定义应用程序由[!DNL Asset Compute Service]调用，该应用程序无法访问开发人员本地计算机上运行的操作。

请参阅[此处](test-custom-application.md)如何测试和调试应用程序。 完成自定义应用程序的开发后，[部署自定义应用程序](deploy-custom-application.md)。

## 尝试Adobe{#try-sample}提供的示例应用程序

以下是示例自定义应用程序：

* [工作人员 — 基本](https://github.com/adobe/asset-compute-example-workers/tree/master/projects/worker-basic)
* [工人 — 动物 — 图片](https://github.com/adobe/asset-compute-example-workers/tree/master/projects/worker-animal-pictures)

### 模板自定义应用程序{#template-custom-application}

[worker-basic](https://github.com/adobe/asset-compute-example-workers/tree/master/projects/worker-basic)是模板应用程序。 它只需复制源文件即可生成演绎版。 此应用程序的内容是在创建aio应用程序时选择`Adobe Asset Compute`时收到的模板。

应用程序文件[`worker-basic.js`](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-basic/worker-basic.js)使用[`asset-compute-sdk`](https://github.com/adobe/asset-compute-sdk#overview)下载源文件、编排每个演绎版处理，并将生成的演绎版上传回云存储。

应用程序代码内定义的[`renditionCallback`](https://github.com/adobe/asset-compute-sdk#rendition-callback-for-worker-required)是执行所有应用程序处理逻辑的位置。 `worker-basic`中的演绎版回调仅将源文件内容复制到演绎版文件。

```javascript
const { worker } = require('@adobe/asset-compute-sdk');
const fs = require('fs').promises;

exports.main = worker(async (source, rendition) => {
    // copy source to rendition to transfer 1:1
    await fs.copyFile(source.path, rendition.path);
});
```

## 调用外部API {#call-external-api}

在应用程序代码中，您可以进行外部API调用以帮助进行应用程序处理。 以下是调用外部API的示例应用程序文件。

```javascript
exports.main = worker(async function (source, rendition) {

    const response = await fetch('https://adobe.com', {
        method: 'GET',
        Authorization: params.AUTH_KEY
    })
});
```

例如， [`worker-animal-pictures`](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-animal-pictures/worker-animal-pictures.js#L46)使用[`node-httptransfer`](https://github.com/adobe/node-httptransfer#node-httptransfer)库从Wikimedia向静态URL发出获取请求。

<!-- TBD: Revisit later to see if this note is required.
>[!NOTE]
>
>For extra authorization for these API calls, see [custom authorization checks](#custom-authorization-checks).
-->

### 传递自定义参数{#pass-custom-parameters}

您可以通过演绎版对象传递自定义参数。 可以在[`rendition`说明](https://github.com/adobe/asset-compute-sdk#rendition)的应用程序内引用这些参数。 呈现版本对象的示例如下：

```json
"renditions": [
    {
        "worker": "https://1234_my_namespace.adobeioruntime.net/api/v1/web/example-custom-worker-master/worker",
        "name": "image.jpg",
        "my-custom-parameter": "my-custom-parameter-value"
    }
]
```

应用程序文件访问自定义参数的示例如下：

```javascript
exports.main = worker(async function (source, rendition) {

    const customParam = rendition.instructions['my-custom-parameter'];
    console.log('Custom paramter:', customParam);
    // should print out `Custom parameter: "my-custom-parameter-value"`
});
```

`example-worker-animal-pictures`传递一个自定义参数[`animal`](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-animal-pictures/worker-animal-pictures.js#L39)，以确定要从Wiki媒体获取的文件。

## 身份验证和授权支持{#authentication-authorization-support}

默认情况下，Asset compute自定义应用程序附带Firefly应用程序的授权和身份验证检查。 可通过在`manifest.yml`中将`require-adobe-auth`注释设置为`true`来启用此功能。

### 访问其他AdobeAPI {#access-adobe-apis}

<!-- TBD: Revisit this section. Where do we document console workspace creation?
-->

将API服务添加到在安装程序中创建的[!DNL Asset Compute]控制台工作区。 这些服务是[!DNL Asset Compute Service]生成的JWT访问令牌的一部分。 令牌和其他凭据可在应用程序操作`params`对象中访问。

```javascript
const accessToken = params.auth.accessToken; // JWT token for Technical Account with entitlements from the console workspace to the API service
const clientId = params.auth.clientId; // Technical Account client Id
const orgId = params.auth.orgId; // Experience Cloud Organization
```

### 传递第三方系统{#pass-credentials-for-tp}的凭据

要处理其他外部服务的凭据，请在操作中将这些作为默认参数传递。 在传输中会自动加密这些数据。 有关更多信息，请参阅运行时开发人员指南](https://www.adobe.io/apis/experienceplatform/runtime/docs.html#!adobedocs/adobeio-runtime/master/guides/creating_actions.md)中的[创建操作。 然后，在部署期间使用环境变量进行设置。 可以在操作内的`params`对象中访问这些参数。

在`manifest.yml`的`inputs`内设置默认参数：

```yaml
packages:
  __APP_PACKAGE__:
    actions:
      worker:
        function: 'index.js'
        runtime: 'nodejs:10'
        web: true
        inputs:
           secretKey: $SECRET_KEY
        annotations:
          require-adobe-auth: true
```

`$VAR`表达式从名为`VAR`的环境变量中读取值。

在开发过程中，该值可以在本地ENV文件中设置为：除了从调用外壳中设置的变量之外，`aio`还会自动从ENV文件中读取环境变量。 在此示例中，ENV文件如下所示：

```CONF
#...
SECRET_KEY=secret-value
```

对于生产部署，您可以在CI系统中设置环境变量，例如在GitHub操作中使用密钥。 最后，访问应用程序内的默认参数，如下所示：

```javascript
const key = params.secretKey;
```

## 调整应用程序大小{#sizing-workers}

应用程序在[!DNL Adobe I/O]运行时的容器中执行，并且[limits](https://www.adobe.io/apis/experienceplatform/runtime/docs.html#!adobedocs/adobeio-runtime/master/guides/system_settings.md)可通过`manifest.yml`进行配置：

```yaml
    actions:
      myworker:
        function: /actions/myworker/index.js
        limits:
          timeout: 300000
          memorySize: 512
          concurrency: 1
```

由于Asset compute应用程序通常进行的处理范围更广，因此更有可能必须调整这些限制以获得最佳性能（足够大以处理二进制资产）和效率（不会因未使用的容器内存而浪费资源）。

运行时中操作的默认超时为一分钟，但可以通过设置`timeout`限制（以毫秒为单位）来增加。 如果希望处理较大的文件，请增加此次。 考虑下载源、处理文件和上传演绎版所花费的总时间。 如果某个操作超时，即在指定的超时限制前不返回激活，则运行时会丢弃该容器且不会重复使用。

Asset compute应用程序从本质上讲往往是网络和磁盘输入或输出绑定。 必须先下载源文件，处理过程通常占用大量资源，然后会再次上载生成的演绎版。

操作容器可用的内存由`memorySize`指定(MB)。 目前，它还定义容器获得的CPU访问量，最重要的是，它是使用运行时成本的关键因素（容器较大，成本更高）。 当您的处理需要更多内存或CPU时，请在此处使用较大的值，但请注意不要浪费资源，因为容器越大，整体吞吐量就越低。

此外，还可以使用`concurrency`设置控制容器内的操作并发。 这是单个容器（具有相同操作）获取的并发激活数。 在此模型中，操作容器与接收多个并发请求的Node.js服务器类似，只有此限制。 如果未设置，则运行时中的默认值为200，这对于较小的Firefly操作非常有用，但对于Asset compute应用程序来说通常太大，因为它们的本地处理和磁盘活动更为密集。 某些应用程序（取决于其实施）在并发活动中也可能无法正常工作。 asset computeSDK通过将文件写入不同的唯一文件夹来确保激活的分隔。

测试应用程序以找到`concurrency`和`memorySize`的最佳数字。 较大的容器=内存限制越高，可能会允许更多并发，但对于较低的流量也可能是浪费。
