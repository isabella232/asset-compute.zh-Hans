---
title: 针对 [!DNL Asset Compute Service]。
description: 使用创建自定义应用程序 [!DNL Asset Compute Service]。
translation-type: tm+mt
source-git-commit: 6de4e3cde9c38f2e23838f5d728dae23e15d2147
workflow-type: tm+mt
source-wordcount: '1559'
ht-degree: 0%

---


# 开发自定义应用程序 {#develop}

开始开发自定义应用程序之前：

* 确保满足所有 [先决条件](/help/understand-extensibility.md#prerequisites-and-provisioning) 。
* 安装所 [需的软件工具](/help/setup-environment.md#create-dev-environment)。
* 请参 [阅设置环境](setup-environment.md) ，以确保您已准备好创建自定义应用程序。

## 创建自定义应用程序 {#create-custom-application}

确保本地安 [装AdobeI/O](https://github.com/adobe/aio-cli) CLI。

1. 要创建自定义应用程序，请 [创建Firefly应用程序](https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/first_app.md#4-bootstrapping-new-app-using-the-cli)。 为此，请在终 `aio app init <app-name>` 端中执行。

   如果您尚未登录，此命令将提示您使用您的Adobe ID登录 [Adobe开发者控制](https://console.adobe.io/) 台。 有关 [从](https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/first_app.md#3-signing-in-from-cli) cli登录的详细信息，请参阅此处。

   Adobe建议您登录。 如果您遇到问题，请按照说 [明在不登录的情况下创建应用程序](https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/first_app.md#42-developer-is-not-logged-in-as-enterprise-organization-user)。

1. 登录后，按照CLI中的提示操作，并选 `Organization`择 `Project`、 `Workspace` 和用于应用程序。 选择您在设置环境时创建的 [项目和工作区](setup-environment.md)。

   ```sh
   $ aio app init <app-name>
   Retrieving information from Adobe I/O Console..
   ? Select Org My Adobe Org
   ? Select Project MyFireflyProject
   ? Select Workspace myworkspace
   create console.json
   ```

1. 提示时， `Which Adobe I/O App features do you want to enable for this project?`请至少选择 `Actions`:

   ```bash
   ? Which Adobe I/O App features do you want to enable for this project?
   select components to include (Press <space> to select, <a> to toggle all, <i> to invert selection)
   ❯◉ Actions: Deploy Runtime actions
   ◯ Events: Publish to Adobe I/O Events
   ◯ Web Assets: Deploy hosted static assets
   ◯ CI/CD: Include GitHub Actions based workflows for Build, Test and Deploy
   ```

1. 出现提示 `Which type of sample actions do you want to create?`时，请确保选择 `Adobe Asset Compute Worker`:

   ```bash
   ? Which type of sample actions do you want to create?
   Select type of actions to generate
   ❯◉ Adobe Asset Compute Worker
   ◯ Generic
   ```

1. 按照其余提示操作，在Visual Studio代码（或您最喜爱的代码编辑器）中打开新的应用程序。 它包含自定义应用程序的基架和示例代码。

   在此阅读有关 [Firefly应用程序的主要组件](https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/first_app.md#5-anatomy-of-a-project-firefly-application)。

   模板应用程序利用我 [们的Asset compute](https://github.com/adobe/asset-compute-sdk#asset-compute-sdk) SDK来上传、下载和安排应用程序再现，因此开发人员只需要实施自定义应用程序逻辑。 在文件 `actions/<worker-name>` 夹中，文 `index.js` 件是添加自定义应用程序代码的位置。

有关自 [定义应用程序的示例](#try-sample) 和构思，请参阅自定义应用程序示例。

### 添加凭据 {#add-credentials}

在创建应用程序时登录时，将在ENV文件中收集大多数Firefly凭据。 但是，使用开发人员工具需要其他凭据。

<!-- TBD: Check if manual setup of credentials is required.
Manual set up of credentials is removed from troubleshooting and best practices page. Link was broken.
If you did not log in, refer to our troubleshooting guide to [set up credentials manually](troubleshooting.md).
-->

#### 开发人员工具存储凭据 {#developer-tool-credentials}

该开发人员工具用于使用实际的应用程序测试自 [!DNL Asset Compute service] 定义应用程序，它需要一个云存储容器来托管测试文件以及接收和显示应用程序生成的演绎版。

>[!NOTE]
>
>这与作为存储的云 [!DNL Adobe Experience Manager] Cloud Service分开。 它仅适用于使用Asset compute开发者工具进行开发和测试。

确保有权访问受支 [持的云存储容器](https://github.com/adobe/asset-compute-devtool#prerequisites)。 此容器可以由多个开发人员根据需要跨不同项目进行共享。

#### 向ENV文件添加凭据 {#add-credentials-env-file}

将开发人员工具的以下凭据添加到Firefly项目根目录的ENV文件中：

1. 将服务添加到Firefly项目时创建的私钥文件的绝对路径：

   ```conf
   ASSET_COMPUTE_PRIVATE_KEY_FILE_PATH=
   ```

1. 如果Firefly `console.json` 应用程序不直接位于根目录中，请添加Adobe开发人员控制台集成JSON文件的绝对路径。 这是在您的项 [`console.json`](https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/first_app.md#42-developer-is-not-logged-in-as-enterprise-organization-user) 目工作区中下载的同一文件。 或者，您也可以使用该命 `aio app use <path_to_console_json>` 令，而不是将路径添加到ENV文件。

   ```conf
   ASSET_COMPUTE_INTEGRATION_FILE_PATH=
   ```

1. 添加S3或Azure存储凭据。 您只需访问一个云存储解决方案。

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

## 执行应用程序 {#run-custom-application}

使用Asset compute开发者工具执行应用程序之前，请正确配置 [凭据](#developer-tool-credentials)。

要在开发人员工具中运行应用程序，请使用 `aio app run` 命令。 它将动作部署到Adobe I/O Runtime，并在您的本地机器上开始开发工具。 此工具用于在开发过程中测试应用程序请求。 以下是一个再现请求示例：

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
>请勿将标志 `--local` 与命令一起 `run` 使用。 它不适用于自定义 [!DNL Asset Compute] 应用程序和Asset compute开发人员工具。 自定义应用程序由调用， [!DNL Asset Compute Service] 它无法访问在开发人员的本地计算机上运行的操作。

请参 [阅](test-custom-application.md) ：如何测试和调试应用程序。 开发完自定义应用程序后，请部 [署自定义应用程序](deploy-custom-application.md)。

## 试用Adobe提供的范例应用程序 {#try-sample}

以下是自定义应用程序的示例：

* [worker-basic](https://github.com/adobe/asset-compute-example-workers/tree/master/projects/worker-basic)
* [工人——动物图片](https://github.com/adobe/asset-compute-example-workers/tree/master/projects/worker-animal-pictures)

### 模板自定义应用程序 {#template-custom-application}

worker- [basic是模板应](https://github.com/adobe/asset-compute-example-workers/tree/master/projects/worker-basic) 用程序。 它只需复制源文件即可生成再现。 此应用程序的内容是在创建aio应用程 `Adobe Asset Compute` 序时收到的模板。

应用程序文 [`worker-basic.js`](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-basic/worker-basic.js) 件，使 [`asset-compute-sdk`](https://github.com/adobe/asset-compute-sdk#overview) 用它下载源文件、编排每个再现处理，并将生成的再现上传回云存储。

在应 [`renditionCallback`](https://github.com/adobe/asset-compute-sdk#rendition-callback-for-worker-required) 用程序代码中定义的是执行所有应用程序处理逻辑的位置。 中的再现回 `worker-basic` 调仅将源文件内容复制到再现文件。

```javascript
const { worker } = require('@adobe/asset-compute-sdk');
const fs = require('fs').promises;

exports.main = worker(async (source, rendition) => {
    // copy source to rendition to transfer 1:1
    await fs.copyFile(source.path, rendition.path);
});
```

## 调用外部API {#call-external-api}

在应用程序代码中，您可以发出外部API调用以帮助处理应用程序。 以下是调用外部API的应用程序文件示例。

```javascript
exports.main = worker(async function (source, rendition) {

    const response = await fetch('https://adobe.com', {
        method: 'GET',
        Authorization: params.AUTH_KEY
    })
});
```

例如，使用 [`worker-animal-pictures`](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-animal-pictures/worker-animal-pictures.js#L46) 库从维基媒体对静态URL发出提取请 [`node-httptransfer`](https://github.com/adobe/node-httptransfer#node-httptransfer) 求。

<!-- TBD: Revisit later to see if this note is required.
>[!NOTE]
>
>For extra authorization for these API calls, see [custom authorization checks](#custom-authorization-checks).
-->

### 传递自定义参数 {#pass-custom-parameters}

您可以通过再现对象传递自定义参数。 它们可以在应用程序内按说明进 [`rendition` 行引用](https://github.com/adobe/asset-compute-sdk#rendition)。 再现对象的示例如下：

```json
"renditions": [
    {
        "worker": "https://1234_my_namespace.adobeioruntime.net/api/v1/web/example-custom-worker-master/worker",
        "name": "image.jpg",
        "my-custom-parameter": "my-custom-parameter-value"
    }
]
```

访问自定义参数的应用程序文件的示例如下：

```javascript
exports.main = worker(async function (source, rendition) {

    const customParam = rendition.instructions['my-custom-parameter'];
    console.log('Custom paramter:', customParam);
    // should print out `Custom parameter: "my-custom-parameter-value"`
});
```

它 `example-worker-animal-pictures` 传递一个自定义 [`animal`](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-animal-pictures/worker-animal-pictures.js#L39) 参数，以确定从维基媒体获取哪个文件。

## 身份验证和授权支持 {#authentication-authorization-support}

默认情况下，Asset compute自定义应用程序随附Firefly应用程序的“授权”和“身份验证”检查。 可通过在中将注 `require-adobe-auth` 释设 `true` 置为启用 `manifest.yml`。

### 访问其他AdobeAPI {#access-adobe-apis}

<!-- TBD: Revisit this section. Where do we document console workspace creation?
-->

将API服务添加到在安装程序 [!DNL Asset Compute] 中创建的控制台工作区。 这些服务是由生成的JWT访问令牌的一部分 [!DNL Asset Compute Service]。 令牌和其他凭据可在应用程序操作对象中 `params` 访问。

```javascript
const accessToken = params.auth.accessToken; // JWT token for Technical Account with entitlements from the console workspace to the API service
const clientId = params.auth.clientId; // Technical Account client Id
const orgId = params.auth.orgId; // Experience Cloud Organization
```

### 为第三方系统传递凭据 {#pass-credentials-for-tp}

要处理其他外部服务的凭据，请将这些凭据作为操作的默认参数进行传递。 在传输中会自动加密这些文件。 有关详细信息，请参阅“运 [行时开发人员指南”中的“创建操作](https://www.adobe.io/apis/experienceplatform/runtime/docs.html#!adobedocs/adobeio-runtime/master/guides/creating_actions.md)”。 然后在部署过程中使用环境变量设置它们。 这些参数可以在操作内 `params` 的对象中访问。

在以下位置设置默认 `inputs` 参数： `manifest.yml`:

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

表达式 `$VAR` 从名为的环境变量中读取该值 `VAR`。

在开发过程中，除了从调用的外壳程序中设置的变量外， `aio` 还可以在本地ENV文件中设置该值，因为它会自动从ENV文件读取环境变量。 在本示例中，ENV文件如下所示：

```CONF
#...
SECRET_KEY=secret-value
```

对于生产部署，您可以在CI系统中设置环境变量，例如在GitHub Actions中使用机密。 最后，访问应用程序内的默认参数，如：

```javascript
const key = params.secretKey;
```

## 调整应用程序大小 {#sizing-workers}

应用程序在Adobe I/O Runtime的容器中 [执行](https://www.adobe.io/apis/experienceplatform/runtime/docs.html#!adobedocs/adobeio-runtime/master/guides/system_settings.md) ，限制可通过 `manifest.yml`:

```yaml
    actions:
      myworker:
        function: /actions/myworker/index.js
        limits:
          timeout: 300000
          memorySize: 512
          concurrency: 1
```

由于Asset compute应用程序通常进行更广泛的处理，因此更可能必须调整这些限制以获得最佳性能（足够大以处理二进制资产）和效率(不会因未使用的容器内存而浪费资源)。

运行时中操作的默认超时为一分钟，但可以通过设置限制(以毫 `timeout` 秒为单位)来增加。 如果希望处理较大的文件，请增加此次处理。 请考虑下载源、处理文件和上传演绎版所花费的总时间。 如果操作超时，即在指定超时限制之前不返回激活，运行时将放弃容器，而不重用它。

Asset compute应用程序从本质上来说往往是网络和磁盘IO绑定。 必须先下载源文件，处理过程通常占用大量IO，然后重新上传生成的演绎版。

操作容器可用的内存以MB为 `memorySize` 单位。 目前，它还定义了容器获得的CPU访问量，最重要的是，它是使用运行时成本的一个关键要素(较大的容器成本更高)。 当您的处理需要更多内存或CPU时，请在此处使用较大的值，但不要浪费资源，因为容器越大，总体吞吐量越低。

此外，可以使用该设置控制容器内的动作并发 `concurrency` 性。 这是单个激活（同一操作）获取的并发容器数。 在此模型中，操作容器类似于Node.js服务器，它接收多个并发请求，最高可达此限制。 如果未设置，则运行时中的默认值为200，这对于较小的Firefly动作很有用，但对于Asset compute应用程序来说通常太大，因为它们的本地处理和磁盘活动更加密集。 某些应用程序可能也无法与并发活动配合使用，具体取决于其实现。 asset computeSDK通过将文件写入不同的唯一文件夹来确保激活分开。

测试应用程序以找到和的最 `concurrency` 佳数 `memorySize`量。 较大的容器=较高的内存限制可能允许更多并发，但对于较低的流量也可能是浪费。
