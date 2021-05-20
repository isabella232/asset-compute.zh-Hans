---
title: 测试和调试 [!DNL Asset Compute Service] 自定义应用程序
description: 测试和调试 [!DNL Asset Compute Service] 自定义应用程序。
exl-id: c2534904-0a07-465e-acea-3cb578d3bc08
source-git-commit: ebc0d717b3f6fc4518f4a79cd44ebe8fdcf9ec6a
workflow-type: tm+mt
source-wordcount: '811'
ht-degree: 0%

---

# 测试和调试自定义应用程序{#test-debug-custom-worker}

## 自定义应用程序{#test-custom-worker}的执行单元测试

在计算机上安装[Docker Desktop](https://www.docker.com/get-started)。 要测试自定义工作程序，请在应用程序的根中执行以下命令：

```bash
$ aio app test
```

<!-- TBD
To run tests for a custom application, run `aio asset-compute test-worker` command at the root of the custom application application.

Document interactively running `adobe-asset-compute` commands `test-worker` and `run-worker`.
-->

这会为项目中的Asset compute应用程序操作运行自定义单元测试框架，如下所述。 它通过`package.json`文件中的配置挂接。 也可以进行JavaScript单元测试，如Jest。 `aio app test` 都执行。

[aio-cli-plugin-asset-compute](https://github.com/adobe/aio-cli-plugin-asset-compute#install-as-local-devdependency)插件作为开发依赖项嵌入到自定义应用程序应用程序中，因此无需在构建/测试系统上安装。

### 应用程序单元测试框架{#unit-test-framework}

该Asset compute应用单元测试框架允许测试应用而不编写任何代码。 它依赖于应用程序的源到再现文件原则。 必须设置特定的文件和文件夹结构，以通过测试源文件、可选参数、预期呈现版本和自定义验证脚本来定义测试用例。 默认情况下，将比较演绎版以实现字节等同。 此外，使用简单的JSON文件可以轻松模拟外部HTTP服务。

### 添加测试{#add-tests}

应在[!DNL Adobe I/O]项目根级别的`test`文件夹内进行测试。 每个应用程序的测试用例应位于路径`test/asset-compute/<worker-name>`中，每个测试用例都对应一个文件夹：

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

请查看[示例自定义应用程序](https://github.com/adobe/asset-compute-example-workers/)以了解一些示例。 下面是详细参考。

### 测试输出{#test-output}

Firefly应用程序根目录的`build`文件夹中提供了包括自定义应用程序日志在内的详细测试输出，如`aio app test`输出中所示。

### 模拟外部服务{#mock-external-services}

在测试案例中，可以通过定义`mock-<HOST_NAME>.json`文件来模拟操作中的外部服务调用，其中HOST_NAME是您要模拟的主机。 示例用例是一个对S3进行单独调用的应用程序。 新的测试结构将如下所示：

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

模拟文件是JSON格式的http响应。 有关更多信息，请参阅[本文档](https://www.mock-server.com/mock_server/creating_expectations.html)。 如果有多个要模拟的主机名，请定义多个`mock-<mocked-host>.json`文件。 以下是名为`mock-google.com.json`的`google.com`模拟文件示例：

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

示例`worker-animal-pictures`包含与之交互的维基媒体服务的[模拟文件](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-animal-pictures/test/asset-compute/worker-animal-pictures/simple-test/mock-upload.wikimedia.org.json)。

#### 跨测试案例{#share-files-across-test-cases}共享文件

如果您在多个测试中共享`file.*`、`params.json`或`validate`脚本，则建议使用相对符号链接。 git支持这些功能。 请确保为共享文件提供唯一的名称，因为您可能有不同的名称。 在以下示例中，测试将混合和匹配一些共享文件以及它们自己的文件：

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

### 测试预期错误{#test-unexpected-errors}

错误测试案例不应包含预期的`rendition.*`文件，而应在`params.json`文件中定义预期的`errorReason`。

>[!NOTE]
>
>如果测试用例不包含预期的`rendition.*`文件，并且未在`params.json`文件内定义预期的`errorReason`，则假定它是任何`errorReason`的错误用例。

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

请参阅[Asset compute错误原因](https://github.com/adobe/asset-compute-commons#error-reasons)的完整列表和说明。

## 调试自定义应用程序{#debug-custom-worker}

以下步骤显示如何使用Visual Studio代码调试自定义应用程序。 它允许查看实时日志、点击断点和分步代码，以及在每次激活时实时重新加载本地代码更改。

其中许多步骤通常由开箱即用的`aio`自动完成，请参阅[Firefly文档](https://www.adobe.io/apis/experienceplatform/project-firefly/docs.html#!AdobeDocs/project-firefly/master/getting_started/first_app.md)中的调试应用程序一节。 目前，以下步骤包含一个解决方法。

1. 从GitHub安装最新的[wskdebug](https://github.com/apache/openwhisk-wskdebug)和可选的[ngrok](https://www.npmjs.com/package/ngrok)。

   ```shell
   npm install -g @openwhisk/wskdebug
   npm install -g ngrok --unsafe-perm=true
   ```

1. 将添加到用户设置JSON文件。 它会一直使用旧的VS代码调试器，而新调试器与wskdebug之间存在[某些问题](https://github.com/apache/openwhisk-wskdebug/issues/74):`"debug.javascript.usePreview": false`。
1. 关闭通过`aio app run`打开的所有应用程序实例。
1. 使用`aio app deploy`部署最新代码。
1. 仅使用`aio asset-compute devtool`执行Asset compute开发工具。 保持打开。
1. 在VS代码编辑器中，将以下调试配置添加到您的`launch.json`中：

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

   从`aio app deploy`的输出中获取`ACTION NAME`。

1. 从运行/调试配置中选择`wskdebug worker` ，然后按播放图标。 等待启动，直到在&#x200B;**[!UICONTROL 调试控制台]**&#x200B;窗口中显示&#x200B;**[!UICONTROL 准备激活]**。

1. 在Devtool中单击&#x200B;**[!UICONTROL 运行]**。 您可以在VS代码编辑器中看到正在执行的操作，并且日志开始显示。

1. 在代码中设置断点，然后再次运行并应该点击。

任何代码更改都会实时加载，并在下次激活时立即生效。

>[!NOTE]
>
>自定义应用程序中的每个请求都有两个激活。 第一个请求是一个Web操作，该操作会在SDK代码中异步调用自身。 第二个激活是命中您的代码的激活。
