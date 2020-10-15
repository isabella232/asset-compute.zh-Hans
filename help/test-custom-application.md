---
title: 测试和调试 [!DNL Asset Compute Service] 自定义应用程序。
description: 测试和调试 [!DNL Asset Compute Service] 自定义应用程序。
translation-type: tm+mt
source-git-commit: 54afa44d8d662ee1499a385f504fca073ab6c347
workflow-type: tm+mt
source-wordcount: '788'
ht-degree: 0%

---


# 测试和调试自定义应用程序 {#test-debug-custom-worker}

## 自定义应用程序的执行单元测试 {#test-custom-worker}

在您 [的计算机上](https://www.docker.com/get-started) ，安装Docker Desktop。 要测试自定义工作器，请在应用程序的根目录下执行以下命令：

```bash
$ aio app test
```

<!-- TBD
To run tests for a custom application, run `adobe-asset-compute test-worker` command in the root of the custom application application application.

Document interactively running `adobe-asset-compute` commands `test-worker` and `run-worker`.
-->

此操作将为项目中的资产计算应用程序操作运行自定义单元测试框架，如下所述。 它通过文件中的配置进行 `package.json` 连接。 还可以进行JavaScript单元测试，如Jest。 `aio app test` 同时执行。

aio- [cli-plugin-asset-compute](https://github.com/adobe/aio-cli-plugin-asset-compute#install-as-local-devdependency) plugin作为开发依赖项嵌入在自定义应用程序应用程序中，因此无需在构建／测试系统上安装它。

### 应用单元测试框架 {#unit-test-framework}

资产计算应用程序单元测试框架允许测试应用程序而无需编写任何代码。 它依赖于应用程序的源到再现文件原则。 必须设置特定的文件和文件夹结构，以使用测试源文件、可选参数、期望的再现和自定义验证脚本定义测试用例。 默认情况下，将比较演绎版以实现字节等式。 此外，使用简单的JSON文件可以轻松模拟外部HTTP服务。

### 添加测试 {#add-tests}

测试应位于AIO `test` 项目根级别的文件夹内。 每个应用程序的测试用例应位于路径中， `test/asset-compute/<worker-name>`每个测试用例对应一个文件夹：

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

查看一些示 [例的自定义](https://github.com/adobe/asset-compute-example-workers/) 应用程序示例。 以下是详细参考。

### Test output {#test-output}

详细的测试输出（包括自定义应用程序的日志）可在Firefly应 `build` 用程序根目录的文件夹中提供，如输出中所 `aio app test` 示。

### 模拟外部服务 {#mock-external-services}

在测试用例中定义文件(HOST_NAME是您要模拟的 `mock-<HOST_NAME>.json` 主机)，从而模拟操作中的外部服务调用。 一个示例用例是对S3进行单独调用的应用程序。 新的测试结构如下所示：

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

模型文件是JSON格式的http响应。 有关详细信息，请参 [阅此文档](https://www.mock-server.com/mock_server/creating_expectations.html)。 如果有多个主机名要模拟，请定义多个 `mock-<mocked-host>.json` 文件。 以下是名为的示例模 `google.com` 型文 `mock-google.com.json`件：

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

该示例 `worker-animal-pictures` 包含 [它与之交互](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-animal-pictures/test/asset-compute/worker-animal-pictures/simple-test/mock-upload.wikimedia.org.json) 的Wikimedia服务的模拟文件。

#### 跨测试用例共享文件 {#share-files-across-test-cases}

如果跨多个测试共享或脚本，建议使 `file.*`用 `params.json` 相对 `validate` 符号链接。 Git支持这些功能。 请确保为共享的文件提供唯一的名称，因为您可能具有不同的名称。 在下面的示例中，测试将混合和匹配几个共享文件，并且它们自己的文件：

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

错误测试案例不应包含预期的 `rendition.*` 文件，并且应在文件 `errorReason` 中定义 `params.json` 预期。

错误测试用例结构：

```json
<error_test_case>/
    file.jpg
    params.json
```

参数文件，错误原因：

```javascript
{
    "errorReason": "SourceUnsupported",
    // ... other params
}
```

请参阅资产计算错误原因 [的完整列表和说明](https://github.com/adobe/asset-compute-commons#error-reasons)。

## 调试自定义应用程序 {#debug-custom-worker}

以下步骤显示了如何使用Visual Studio代码调试自定义应用程序。 它允许查看实时日志、点击断点和遍历代码以及在每个激活实时重新加载本地代码更改。

其中许多步骤通常是开箱即 `aio` 用的自动化步骤，请参阅Firefly文档中的调试应 [用程序一节](https://www.adobe.io/apis/experienceplatform/project-firefly/docs.html#!AdobeDocs/project-firefly/master/getting_started/first_app.md)。 目前，以下步骤包括解决方法。

1. 从GitHub和 [可选](https://github.com/apache/openwhisk-wskdebug) ngrok安装最新的wskdebug [程序](https://www.npmjs.com/package/ngrok)。

   ```shell
   npm install -g @openwhisk/wskdebug
   npm install -g ngrok --unsafe-perm=true
   ```

1. 添加到用户设置JSON文件。 它一直使用旧的VS代码调试器，新调试器有 [一些wskdebug](https://github.com/apache/openwhisk-wskdebug/issues/74) 问题： `"debug.javascript.usePreview": false`.
1. 关闭通过打开的任何应用程序实 `aio app run`例。
1. 使用部署最新代码 `aio app deploy`。
1. 仅使用执行资产计算开发工具 `npx adobe-asset-compute devtool`。 保持打开。
1. 在VS代码编辑器中，将以下调试配置添加到您的 `launch.json`:

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

   从的输出中获取操作名称 `aio app deploy`。 看起来象 `Your deployed actions -> TypicalCoffeeCat-0.0.1/__secured_worker`。

1. 从运 `wskdebug worker` 行／调试配置中进行选择，然后按播放图标。 等待它开始，直到它在“调试控 **[!UICONTROL 制台”窗口中显]** 示“准 **[!UICONTROL 备激活]** ”。

1. 在“ **[!UICONTROL Devtool]** ”中单击“运行”。 您可以看到在VS代码编辑器中执行的操作和日志开始显示。

1. 在代码中设置断点，再次运行并应点击它。

任何代码更改都会实时加载，并在下次激活发生时立即生效。

>[!NOTE]
>
>自定义应用程序中的每个请求都有两个激活。 第一个请求是在SDK代码中异步调用自身的Web操作。 第二个激活是命中您的代码的。
