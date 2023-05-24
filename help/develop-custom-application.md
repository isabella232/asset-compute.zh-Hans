---
title: 为以下对象开发 [!DNL Asset Compute Service]
description: 创建自定义应用程序，使用 [!DNL Asset Compute Service].
exl-id: a0c59752-564b-4bb6-9833-ab7c58a7f38e
source-git-commit: 2dde177933477dc9ac2ff5a55af1fd2366e18359
workflow-type: tm+mt
source-wordcount: '1618'
ht-degree: 0%

---

# 开发自定义应用程序 {#develop}

在开始开发自定义应用程序之前：

* 确保所有 [先决条件](/help/understand-extensibility.md#prerequisites-and-provisioning) 符合。
* 安装 [所需的软件工具](/help/setup-environment.md#create-dev-environment).
* 参见 [设置环境](setup-environment.md) 以确保您准备好创建自定义应用程序。

## 创建自定义应用程序 {#create-custom-application}

确保具有 [[!DNL Adobe I/O] CLI](https://github.com/adobe/aio-cli) 本地安装。

1. 要创建自定义应用程序， [创建App Builder项目](https://developer.adobe.com/app-builder/docs/getting_started/first_app/#4-bootstrapping-new-app-using-the-cli). 为此，请执行 `aio app init <app-name>` 在你的终端机里。

   如果您尚未登录，此命令会提示您登录 [Adobe Developer控制台](https://console.adobe.io/) 使用您的Adobe ID。 参见 [此处](https://developer.adobe.com/app-builder/docs/getting_started/first_app/#3-signing-in-from-cli) 有关从cli登录的详细信息。

   Adobe建议您登录。 如果您遇到问题，请按照相关说明操作 [创建应用程序而不登录](https://developer.adobe.com/app-builder/docs/getting_started/first_app/#42-developer-is-not-logged-in-as-enterprise-organization-user).

1. 登录后，按照CLI中的提示操作，然后选择 `Organization`， `Project`、和 `Workspace` 用于应用程序。 选择您在创建项目和工作区时 [设置环境](setup-environment.md). 出现提示时 `Which extension point(s) do you wish to implement ?`，确保选择 `DX Asset Compute Worker`：

   ```sh
   $ aio app init <app-name>
   Retrieving information from [!DNL Adobe I/O] Console.
   ? Select Org My Adobe Org
   ? Select Project MyAdobe Developer App BuilderProject
   ? Which extension point(s) do you wish to implement ? (Press <space> to select, <a>
   to toggle all, <i> to invert selection)
   ❯◯ DX Experience Cloud SPA
   ◯ DX Asset Compute Worker
   ```

1. 出现以下提示时 `Which Adobe I/O App features do you want to enable for this project?`，选择 `Actions`. 确保取消选择 `Web Assets` 选项，因为Web资产使用不同的身份验证和授权检查。

   ```bash
   ? Which Adobe I/O App features do you want to enable for this project?
   select components to include (Press <space> to select, <a> to toggle all, <i> to invert selection)
   ❯◉ Actions: Deploy Runtime actions
   ◯ Events: Publish to Adobe I/O Events
   ◯ Web Assets: Deploy hosted static assets
   ◯ CI/CD: Include GitHub Actions based workflows for Build, Test and Deploy
   ```

1. 出现提示时 `Which type of sample actions do you want to create?`，确保选择 `Adobe Asset Compute Worker`：

   ```bash
   ? Which type of sample actions do you want to create?
   Select type of actions to generate
   ❯◉ Adobe Asset Compute Worker
   ◯ Generic
   ```

1. 按照其余提示进行操作，并在Visual Studio Code（或您喜爱的代码编辑器）中打开新的应用程序。 它包含自定义应用程序的基架和示例代码。

   请在此处阅读 [App Builder应用程序的主要组件](https://developer.adobe.com/app-builder/docs/getting_started/first_app/#5-anatomy-of-an-app-builder-application).

   模板应用程序利用我们的 [ASSET COMPUTESDK](https://github.com/adobe/asset-compute-sdk#asset-compute-sdk) 对于应用程序演绎版的上传、下载和编排，因此开发人员只需实施自定义应用程序逻辑。 内部 `actions/<worker-name>` 文件夹， `index.js` 文件是添加自定义应用程序代码的位置。

参见 [自定义应用程序示例](#try-sample) 有关自定义应用程序的示例和想法。

### 添加凭据 {#add-credentials}

在创建应用程序时登录时，系统会在ENV文件中收集大多数App Builder凭据。 但是，使用开发人员工具需要其他凭据。

<!-- TBD: Check if manual setup of credentials is required.
Manual set up of credentials is removed from troubleshooting and best practices page. Link was broken.
If you did not log in, refer to our troubleshooting guide to [set up credentials manually](troubleshooting.md).
-->

#### 开发人员工具存储凭据 {#developer-tool-credentials}

用于测试自定义应用程序的开发人员工具，具有实际 [!DNL Asset Compute service] 需要云存储容器来托管测试文件，以及接收和显示应用程序生成的演绎版。

>[!NOTE]
>
>这与的云存储不同 [!DNL Adobe Experience Manager] as a [!DNL Cloud Service]. 它仅适用于使用Asset compute开发人员工具进行开发和测试。

确保有权访问 [支持的云存储容器](https://github.com/adobe/asset-compute-devtool#prerequisites). 如果需要，此容器可以由多个开发人员跨不同项目共享。

#### 将凭据添加到环境文件 {#add-credentials-env-file}

将开发人员工具的以下凭据添加到App Builder项目根目录中的环境文件：

1. 将绝对路径添加到向App Builder项目添加服务时创建的私钥文件：

   ```conf
   ASSET_COMPUTE_PRIVATE_KEY_FILE_PATH=
   ```

1. 从Adobe Developer控制台下载文件。 转到项目的根目录并单击右上角的“全部下载”。 文件下载方式 `<namespace>-<workspace>.json` 作为文件名。 执行下列操作之一：

   * 将文件重命名为 `console.json` 并将其移动到项目的根目录中。
   * 或者，您也可以将绝对路径添加到Adobe Developer控制台集成JSON文件。 这是相同的 [`console.json`](https://developer.adobe.com/app-builder/docs/getting_started/first_app/#42-developer-is-not-logged-in-as-enterprise-organization-user) 在您的项目工作区中下载的文件。

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
>此 `config.json` 文件包含凭据。 在项目中，将JSON文件添加到您的 `.gitignore` 文件阻止其共享。 这同样适用于.env和.aio文件。

## 执行应用程序 {#run-custom-application}

在使用Asset compute开发人员工具执行应用程序之前，请正确配置 [凭据](#developer-tool-credentials).

要在开发人员工具中运行应用程序，请使用 `aio app run` 命令。 它将操作部署到 [!DNL Adobe I/O] 在本地计算机上运行并启动开发工具。 此工具用于在开发期间测试应用程序请求。 以下是格式副本请求示例：

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
>请勿使用 `--local` 标记和 `run` 命令。 它不适用于 [!DNL Asset Compute] 自定义应用程序和Asset compute开发人员工具。 自定义应用程序由调用 [!DNL Asset Compute Service] 无法访问在开发人员的本地计算机上运行的操作。

参见 [此处](test-custom-application.md) 如何测试和调试应用程序。 完成自定义应用程序的开发后， [部署自定义应用程序](deploy-custom-application.md).

## 尝试使用Adobe提供的示例应用程序 {#try-sample}

以下是自定义应用程序示例：

* [工作基本](https://github.com/adobe/asset-compute-example-workers/tree/master/projects/worker-basic)
* [工人 — 动物 — 图片](https://github.com/adobe/asset-compute-example-workers/tree/master/projects/worker-animal-pictures)

### 模板自定义应用程序 {#template-custom-application}

此 [工作基本](https://github.com/adobe/asset-compute-example-workers/tree/master/projects/worker-basic) 是一个模板应用程序。 它只通过复制源文件来生成演绎版。 此应用程序的内容是选择时收到的模板 `Adobe Asset Compute` 创建aio应用程序时。

应用程序文件， [`worker-basic.js`](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-basic/worker-basic.js) 使用 [`asset-compute-sdk`](https://github.com/adobe/asset-compute-sdk#overview) 要下载源文件，请编排每个演绎版处理，并将生成的演绎版上传回云存储。

此 [`renditionCallback`](https://github.com/adobe/asset-compute-sdk#rendition-callback-for-worker-required) 在应用程序代码中定义的位置，用于执行所有应用程序处理逻辑。 中的演绎版回调 `worker-basic` 只需将源文件内容复制到演绎版文件。

```javascript
const { worker } = require('@adobe/asset-compute-sdk');
const fs = require('fs').promises;

exports.main = worker(async (source, rendition) => {
    // copy source to rendition to transfer 1:1
    await fs.copyFile(source.path, rendition.path);
});
```

## 调用外部API {#call-external-api}

在应用程序代码中，您可以进行外部API调用以帮助处理应用程序。 下面是调用外部API的示例应用程序文件。

```javascript
exports.main = worker(async function (source, rendition) {

    const response = await fetch('https://adobe.com', {
        method: 'GET',
        Authorization: params.AUTH_KEY
    })
});
```

例如， [`worker-animal-pictures`](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-animal-pictures/worker-animal-pictures.js#L46) 使用向Wikimedia中的静态URL发出提取请求 [`node-httptransfer`](https://github.com/adobe/node-httptransfer#node-httptransfer) 库。

<!-- TBD: Revisit later to see if this note is required.
>[!NOTE]
>
>For extra authorization for these API calls, see [custom authorization checks](#custom-authorization-checks).
-->

### 传递自定义参数 {#pass-custom-parameters}

您可以通过演绎版对象传递自定义参数。 它们可在应用程序内部引用，位置如下： [`rendition` 说明](https://github.com/adobe/asset-compute-sdk#rendition). 呈现对象的示例为：

```json
"renditions": [
    {
        "worker": "https://1234_my_namespace.adobeioruntime.net/api/v1/web/example-custom-worker-master/worker",
        "name": "image.jpg",
        "my-custom-parameter": "my-custom-parameter-value"
    }
]
```

访问自定义参数的应用程序文件的示例为：

```javascript
exports.main = worker(async function (source, rendition) {

    const customParam = rendition.instructions['my-custom-parameter'];
    console.log('Custom paramter:', customParam);
    // should print out `Custom parameter: "my-custom-parameter-value"`
});
```

此 `example-worker-animal-pictures` 传递自定义参数 [`animal`](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-animal-pictures/worker-animal-pictures.js#L39) 以确定要从维基媒体中获取哪个文件。

## 身份验证和授权支持 {#authentication-authorization-support}

默认情况下，Asset compute自定义应用程序会随App Builder项目的授权和身份验证检查提供。 这可通过设置 `require-adobe-auth` 注释至 `true` 在 `manifest.yml`.

### 访问其他AdobeAPI {#access-adobe-apis}

<!-- TBD: Revisit this section. Where do we document console workspace creation?
-->

将API服务添加到 [!DNL Asset Compute] 控制台工作区已在设置中创建。 这些服务是由生成的JWT访问令牌的一部分 [!DNL Asset Compute Service]. 可在应用程序操作中访问令牌和其他凭据 `params` 对象。

```javascript
const accessToken = params.auth.accessToken; // JWT token for Technical Account with entitlements from the console workspace to the API service
const clientId = params.auth.clientId; // Technical Account client Id
const orgId = params.auth.orgId; // Experience Cloud Organization
```

### 为第三方系统传递凭据 {#pass-credentials-for-tp}

要处理其他外部服务的凭据，请将这些凭据作为默认参数传递给操作。 这些在传输过程中会自动加密。 有关更多信息，请参阅 [运行时开发人员指南中的“创建操作”](https://www.adobe.io/apis/experienceplatform/runtime/docs.html#!adobedocs/adobeio-runtime/master/guides/creating_actions.md). 然后在部署期间使用环境变量设置它们。 这些参数可在以下位置访问： `params` 操作内的对象。

在中设置默认参数 `inputs` 在 `manifest.yml`：

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

此 `$VAR` 表达式从名为的环境变量中读取值 `VAR`.

在开发期间，可以在本地ENV文件中将该值设置为 `aio` 除了从调用shell设置的变量之外，还会自动从ENV文件中读取环境变量。 在此示例中，ENV文件如下所示：

```CONF
#...
SECRET_KEY=secret-value
```

对于生产部署，您可以在CI系统中设置环境变量，例如使用GitHub操作中的密钥。 最后，访问应用程序内的默认参数，如下所示：

```javascript
const key = params.secretKey;
```

## 调整应用程序大小 {#sizing-workers}

应用程序在中的容器中执行 [!DNL Adobe I/O] 运行时间 [限制](https://www.adobe.io/apis/experienceplatform/runtime/docs.html#!adobedocs/adobeio-runtime/master/guides/system_settings.md) 这些资源可通过 `manifest.yml`：

```yaml
    actions:
      myworker:
        function: /actions/myworker/index.js
        limits:
          timeout: 300000
          memorySize: 512
          concurrency: 1
```

由于通常由Asset compute应用程序完成的更全面的处理，因此更有可能需要调整这些限制以实现最佳性能（足够大以处理二进制资源）和效率（不会由于未使用的容器内存而浪费资源）。

运行时操作的默认超时为一分钟，但可以通过设置 `timeout` 限制（毫秒）。 如果您希望处理较大的文件，请增加此时间。 考虑下载源、处理文件和上传演绎版所需的总时间。 如果操作超时（即在指定的超时限制之前未返回激活），运行时将放弃容器而不重新使用它。

asset compute应用程序本质上是网络和磁盘输入或输出绑定。 必须先下载源文件，处理过程通常占用大量资源，然后会再次上传生成的演绎版。

操作容器可用的内存由指定 `memorySize` 以MB为单位。 目前，这也定义了容器获得的CPU访问量，最重要的是，这是使用运行时成本的关键元素（容器越大，成本越高）。 当您的处理需要更多的内存或CPU时，请在此处使用更大的值，但请注意不要像容器越大，总吞吐量越低那样浪费资源。

此外，可以使用以下项控制容器内的操作并发 `concurrency` 设置。 这是单个容器（属于同一操作）获得的并发激活数。 在此模型中，操作容器类似于Node.js服务器，可接收多个并发请求，最大可达该限制。 如果未设置，则运行时中的默认值为200，这非常适合较小的App Builder操作，但通常对于Asset compute应用程序来说太大了，因为它们具有更密集的本地处理和磁盘活动。 某些应用程序可能也无法很好地与并发活动配合使用，具体取决于其实施。 asset computeSDK通过将文件写入不同的唯一文件夹来确保激活分开。

测试应用程序以找出最佳数量 `concurrency` 和 `memorySize`. 更大的容器=更高的内存限制可能会允许更多的并发性，但也可能对较低的流量造成浪费。
