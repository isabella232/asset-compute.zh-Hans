---
title: 测试和调试 [!DNL Asset Compute Service] 自定义应用程序
description: 测试和调试 [!DNL Asset Compute Service] 自定义应用程序。
exl-id: c2534904-0a07-465e-acea-3cb578d3bc08
source-git-commit: 2dde177933477dc9ac2ff5a55af1fd2366e18359
workflow-type: tm+mt
source-wordcount: '812'
ht-degree: 0%

---

# 测试和调试自定义应用程序 {#test-debug-custom-worker}

## 自定义应用程序的执行单元测试 {#test-custom-worker}

安装 [Docker Desktop](https://www.docker.com/get-started) 你的机器上。 要测试自定义工作程序，请在应用程序的根中执行以下命令：

```bash
$ aio app test
```

<!-- TBD
To run tests for a custom application, run `aio asset-compute test-worker` command at the root of the custom application application.

Document interactively running `adobe-asset-compute` commands `test-worker` and `run-worker`.
-->

这会为项目中的Asset compute应用程序操作运行自定义单元测试框架，如下所述。 它通过 `package.json` 文件。 也可以进行JavaScript单元测试，如Jest。 `aio app test` 都执行。

的 [aio-cli-plugin-asset-compute](https://github.com/adobe/aio-cli-plugin-asset-compute#install-as-local-devdependency) 插件会作为开发依赖项嵌入到自定义应用程序应用程序中，因此无需在构建/测试系统上安装插件。

### 应用单元测试框架 {#unit-test-framework}

该Asset compute应用单元测试框架允许测试应用而不编写任何代码。 它依赖于应用程序的源到再现文件原则。 必须设置特定的文件和文件夹结构，以通过测试源文件、可选参数、预期呈现版本和自定义验证脚本来定义测试用例。 默认情况下，将比较演绎版以实现字节等同。 此外，使用简单的JSON文件可以轻松模拟外部HTTP服务。

### 添加测试 {#add-tests}

在 `test` 的根级别上的文件夹 [!DNL Adobe I/O] 项目。 每个应用程序的测试用例应位于路径中 `test/asset-compute/<worker-name>`，每个测试案例都有一个文件夹：

```yaml
action/
manifest.yml
package.json
...
test/
  asset-compute/
    <worker-name>/
        <testcase1>/
            file.jpg
            params.json
            rendition.png
        <testcase2>/
            file.jpg
            params.json
            rendition.gif
            validate
        <testcase3>/
            file.jpg
            params.json
            rendition.png
            mock-adobe.com.json
            mock-console.adobe.io.json
```

看看 [示例自定义应用程序](https://github.com/adobe/asset-compute-example-workers/) 例。 下面是详细参考。

### 测试输出 {#test-output}

在 `build` 文件夹(位于Adobe Developer App Builder应用程序的根目录中)，如 `aio app test` 输出。

### 模拟外部服务 {#mock-external-services}

可以通过定义 `mock-<HOST_NAME>.json` 文件，其中HOST_NAME是要模拟的主机。 示例用例是一个对S3进行单独调用的应用程序。 新的测试结构将如下所示：

```json
test/
  asset-compute/
    <worker-name>/
      <testcase3>/
        file.jpg
        params.json
        rendition.png
        mock-<HOST_NAME1>.json
        mock-<HOST_NAME2>.json
```

模拟文件是JSON格式的http响应。 有关更多信息，请参阅 [本文档](https://www.mock-server.com/mock_server/creating_expectations.html). 如果要模拟多个主机名，请定义多个 `mock-<mocked-host>.json` 文件。 以下是的模拟文件示例 `google.com` 已命名 `mock-google.com.json`:

```json
[{
    "httpRequest": {
        "path": "/images/hello.txt"
        "method": "GET"
    },
    "httpResponse": {
        "statusCode": 200,
        "body": {
          "message": "hello world!"
        }
    }
}]
```

示例 `worker-animal-pictures` 包含a [模拟文件](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-animal-pictures/test/asset-compute/worker-animal-pictures/simple-test/mock-upload.wikimedia.org.json) 为维基媒体服务，它与之互动。

#### 跨测试案例共享文件 {#share-files-across-test-cases}

如果您共享 `file.*`, `params.json` 或 `validate` 脚本。 git支持这些功能。 请确保为共享文件提供唯一的名称，因为您可能有不同的名称。 在以下示例中，测试将混合和匹配一些共享文件以及它们自己的文件：

```json
tests/
    file-one.jpg
    params-resize.json
    params-crop.json
    validate-image-compare

    jpg-png-resize/
        file.jpg    - symlink: ../file-one.jpg
        params.json - symlink: ../params-resize.json
        rendition.png
        validate    - symlink: ../validate-image-compare

    jpg2-png-crop/
        file.jpg
        params.json - symlink: ../params-crop.json
        rendition.gif
        validate    - symlink: ../validate-image-compare

    jpg-gif-crop/
        file.jpg    - symlink: ../file-one.jpg
        params.json - symlink: ../params-crop.json
        rendition.gif
        validate
```

### 测试预期错误 {#test-unexpected-errors}

错误测试案例不应包含预期 `rendition.*` 文件，并应定义预期 `errorReason` 内部 `params.json` 文件。

>[!NOTE]
>
>如果测试用例不包含预期 `rendition.*` 文件，但未定义预期 `errorReason` 内部 `params.json` 文件，则假定它是任何 `errorReason`.

错误测试用例结构：

```json
<error_test_case>/
    file.jpg
    params.json
```

参数文件，错误原因如下：

```javascript
{
    "errorReason": "SourceUnsupported",
    // ... other params
}
```

请参阅 [asset compute错误原因](https://github.com/adobe/asset-compute-commons#error-reasons).

## 调试自定义应用程序 {#debug-custom-worker}

以下步骤显示如何使用Visual Studio代码调试自定义应用程序。 它允许查看实时日志、点击断点和分步代码，以及在每次激活时实时重新加载本地代码更改。

其中许多步骤通常由 `aio` 开箱即用，请参阅 [Adobe Developer App Builder文档](https://developer.adobe.com/app-builder/docs/getting_started/first_app). 目前，以下步骤包含一个解决方法。

1. 安装最新 [wsdebug](https://github.com/apache/openwhisk-wskdebug) 和可选 [恩格罗克](https://www.npmjs.com/package/ngrok).

   ```shell
   npm install -g @openwhisk/wskdebug
   npm install -g ngrok --unsafe-perm=true
   ```

1. 将添加到用户设置JSON文件。 它会持续使用旧的VS Code调试器，而新调试器具有 [一些问题](https://github.com/apache/openwhisk-wskdebug/issues/74) 使用wskdebug: `"debug.javascript.usePreview": false`.
1. 关闭通过打开的任何应用程序实例 `aio app run`.
1. 使用部署最新代码 `aio app deploy`.
1. 仅使用执行Asset compute开发工具 `aio asset-compute devtool`. 保持打开。
1. 在VS代码编辑器中，将以下调试配置添加到 `launch.json`:

   ```json
   {
     "type": "node",
     "request": "launch",
     "name": "wskdebug worker",
     "runtimeExecutable": "wskdebug",
     "args": [
       "PROJECT-0.0.1/__secured_worker",           // Replace this with your ACTION NAME
       "${workspaceFolder}/actions/worker/index.js",
       "-l",
       "--ngrok"
     ],
     "localRoot": "${workspaceFolder}",
     "remoteRoot": "/code",
     "outputCapture": "std",
     "timeout": 30000
   }
   ```

   获取 `ACTION NAME` 从 `aio app deploy`.

1. 选择 `wskdebug worker` 在运行/调试配置中，按“播放”图标。 等它开始，直到它显示 **[!UICONTROL 准备激活]** 在 **[!UICONTROL 调试控制台]** 窗口。

1. 单击 **[!UICONTROL 运行]** 中。 您可以在VS代码编辑器中看到正在执行的操作，并且日志开始显示。

1. 在代码中设置断点，然后再次运行并应该点击。

任何代码更改都会实时加载，并在下次激活时立即生效。

>[!NOTE]
>
>自定义应用程序中的每个请求都有两个激活。 第一个请求是一个Web操作，该操作会在SDK代码中异步调用自身。 第二个激活是命中您的代码的激活。
