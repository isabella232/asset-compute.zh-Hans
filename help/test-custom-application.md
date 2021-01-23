---
title: 测试和调试 [!DNL Asset Compute Service] 自定义应用程序
description: 测试和调试 [!DNL Asset Compute Service] 自定义应用程序。
translation-type: tm+mt
source-git-commit: 95e384d2a298b3237d4f93673161272744e7f44a
workflow-type: tm+mt
source-wordcount: '787'
ht-degree: 0%

---


# 测试和调试自定义应用程序{#test-debug-custom-worker}

## 对自定义应用程序{#test-custom-worker}执行单元测试

在您的计算机上安装[Docker Desktop](https://www.docker.com/get-started)。 要测试自定义工作器，请在应用程序的根目录下执行以下命令：

```bash
$ aio app test
```

<!-- TBD
To run tests for a custom application, run `adobe-asset-compute test-worker` command in the root of the custom application application application.

Document interactively running `adobe-asset-compute` commands `test-worker` and `run-worker`.
-->

它运行一个自定义单元测试框架，用于Asset compute项目中的应用程序操作，如下所述。 它通过`package.json`文件中的配置进行连接。 还可以进行JavaScript单元测试，如Jest。 `aio app test` 同时执行。

[aio-cli-plugin-asset-compute](https://github.com/adobe/aio-cli-plugin-asset-compute#install-as-local-devdependency)插件作为开发依赖项嵌入到自定义应用程序应用程序中，因此无需在构建／测试系统上安装它。

### 应用程序单元测试框架{#unit-test-framework}

asset compute应用单元测试框架允许测试应用程序而无需编写任何代码。 它依赖于应用程序的源到再现文件原则。 必须设置特定的文件和文件夹结构，以使用测试源文件、可选参数、期望的再现和自定义验证脚本定义测试用例。 默认情况下，将比较演绎版以实现字节等式。 此外，使用简单的JSON文件可以轻松模拟外部HTTP服务。

### 添加测试{#add-tests}

测试应位于[!DNL Adobe I/O]项目的根级别的`test`文件夹中。 每个应用程序的测试用例应位于路径`test/asset-compute/<worker-name>`中，每个测试用例对应一个文件夹：

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

请查看[示例自定义应用程序](https://github.com/adobe/asset-compute-example-workers/)以了解一些示例。 以下是详细参考。

### 测试输出{#test-output}

包括自定义应用程序日志在内的详细测试输出在Firefly应用程序根目录的`build`文件夹中提供，如`aio app test`输出中所示。

### 模拟外部服务{#mock-external-services}

在测试案例中定义`mock-<HOST_NAME>.json`文件（其中HOST_NAME是您要模拟的主机），可以模拟操作中的外部服务调用。 一个示例用例是对S3进行单独调用的应用程序。 新的测试结构如下所示：

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

模型文件是JSON格式的http响应。 有关详细信息，请参阅[本文档](https://www.mock-server.com/mock_server/creating_expectations.html)。 如果有多个主机名要模拟，请定义多个`mock-<mocked-host>.json`文件。 下面是名为`mock-google.com.json`的`google.com`的示例模型文件：

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

示例`worker-animal-pictures`包含与之交互的Wikimedia服务的[模拟文件](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-animal-pictures/test/asset-compute/worker-animal-pictures/simple-test/mock-upload.wikimedia.org.json)。

#### 跨测试用例共享文件{#share-files-across-test-cases}

如果跨多个测试共享`file.*`、`params.json`或`validate`脚本，建议使用相对符号链接。 Git支持这些功能。 请确保为共享的文件提供唯一的名称，因为您可能具有不同的名称。 在下面的示例中，测试将混合和匹配几个共享文件，并且它们自己的文件：

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

错误测试案例不应包含预期的`rendition.*`文件，并应在`params.json`文件中定义预期的`errorReason`。

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

请参阅[列表错误原因](https://github.com/adobe/asset-compute-commons#error-reasons)的完整Asset compute和说明。

## 调试自定义应用程序{#debug-custom-worker}

以下步骤显示了如何使用Visual Studio代码调试自定义应用程序。 它允许查看实时日志、点击断点和遍历代码以及在每个激活实时重新加载本地代码更改。

这些步骤中的许多通常由开箱即用的`aio`自动完成，请参阅[Firefly文档](https://www.adobe.io/apis/experienceplatform/project-firefly/docs.html#!AdobeDocs/project-firefly/master/getting_started/first_app.md)中的调试应用程序一节。 目前，以下步骤包括解决方法。

1. 从GitHub安装最新的[wskdebug](https://github.com/apache/openwhisk-wskdebug)和可选的[ngrok](https://www.npmjs.com/package/ngrok)。

   ```shell
   npm install -g @openwhisk/wskdebug
   npm install -g ngrok --unsafe-perm=true
   ```

1. 添加到用户设置JSON文件。 它继续使用旧的VS代码调试器，新调试器的wskdebug有[某些问题](https://github.com/apache/openwhisk-wskdebug/issues/74):`"debug.javascript.usePreview": false`。
1. 关闭通过`aio app run`打开的应用程序的所有实例。
1. 使用`aio app deploy`部署最新代码。
1. 使用`npx adobe-asset-compute devtool`仅执行Asset compute开发工具。 保持打开。
1. 在VS代码编辑器中，将以下调试配置添加到`launch.json`:

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

   从`aio app deploy`的输出中提取ACTION NAME。 它看起来像`Your deployed actions -> TypicalCoffeeCat-0.0.1/__secured_worker`。

1. 从运行／调试配置中选择`wskdebug worker`，然后按播放图标。 等待它开始，直到它在&#x200B;**[!UICONTROL 调试控制台]**&#x200B;窗口中显示&#x200B;**[!UICONTROL 激活]**&#x200B;就绪。

1. 单击Devtool中的&#x200B;**[!UICONTROL 运行]**。 您可以看到在VS代码编辑器中执行的操作和日志开始显示。

1. 在代码中设置断点，再次运行并应点击它。

任何代码更改都会实时加载，并在下次激活发生时立即生效。

>[!NOTE]
>
>自定义应用程序中的每个请求都有两个激活。 第一个请求是在SDK代码中异步调用自身的Web操作。 第二个激活是命中您的代码的。
