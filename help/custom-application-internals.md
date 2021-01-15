---
title: 了解自定义应用程序的工作。
description: 内部工作 [!DNL Asset Compute Service] 自定义应用程序有助于了解其工作方式。
translation-type: tm+mt
source-git-commit: d26ae470507e187249a472ececf5f08d803a636c
workflow-type: tm+mt
source-wordcount: '751'
ht-degree: 0%

---


# 自定义应用程序{#how-custom-application-works}的内部

使用下图可了解当客户端使用自定义应用程序处理数字资产时的端到端工作流程。

![自定义应用程序工作流程](assets/customworker.png)

*图：使用处理资产时涉及的步骤 [!DNL Asset Compute Service]。*

## 注册{#registration}

客户端必须在向[`/process`](api.md#process-request)发出第一个请求之前调用[`/register`](api.md#register)一次，以设置和检索日志URL，以接收[!DNL Adobe I/O]的AdobeAsset compute事件。

```sh
curl -X POST \
  https://asset-compute.adobe.io/register \
  -H "x-ims-org-id: $ORG_ID" \
  -H "x-gw-ims-org-id: $ORG_ID" \
  -H "Authorization: Bearer $JWT_TOKEN" \
  -H "x-api-key: $API_KEY"
```

[`@adobe/asset-compute-client`](https://github.com/adobe/asset-compute-client#usage) JavaScript库可用于NodeJS应用程序中，以处理从注册、处理到异步事件处理的所有必要步骤。 有关所需标头的详细信息，请参阅[身份验证和授权](api.md)。

## 正在处理 {#processing}

客户端发送一个[处理](api.md#process-request)请求。

```sh
curl -X POST \
  https://asset-compute.adobe.io/process \
  -H "x-ims-org-id: $ORG_ID" \
  -H "x-gw-ims-org-id: $ORG_ID" \
  -H "Authorization: Bearer $JWT_TOKEN" \
  -H "x-api-key: $API_KEY" \
  -d "<RENDITION_JSON>
```

客户端负责使用预签名URL正确设置演绎版格式。 [`@adobe/node-cloud-blobstore-wrapper`](https://github.com/adobe/node-cloud-blobstore-wrapper#presigned-urls) JavaScript库可用于NodeJS应用程序中预签URL。 目前，该库仅支持Azure Blob存储和AWS S3容器。

处理请求返回`requestId`，它可用于轮询[!DNL Adobe I/O]事件。

以下是自定义应用程序处理请求示例。

```json
{
    "source": "https://www.adobe.com/some-source-file.jpg",
    "renditions" : [
        {
            "worker": "https://my-project-namespace.adobeioruntime.net/api/v1/web/my-namespace-version/my-worker",
            "name": "rendition1.jpg",
            "target": "https://some-presigned-put-url-for-rendition1.jpg",
        }
    ],
    "userData": {
        "my-asset-id": "1234567890"
    }
}
```

[!DNL Asset Compute Service]会向自定义应用程序发送自定义应用程序再现请求。 它对提供的应用程序URL使用HTTPPOST，该URL是来自Project Firefly的安全Web操作URL。 所有请求都使用HTTPS协议来最大化数据安全性。

自定义应用程序使用的[Asset computeSDK](https://github.com/adobe/asset-compute-sdk#adobe-asset-compute-worker-sdk)处理HTTPPOST请求。 它还处理源的下载、上传演绎版、发送[!DNL Adobe I/O]事件和错误处理。

<!-- TBD: Add the application diagram. -->

### 应用程序代码{#application-code}

自定义代码只需提供一个回调，它获取本地可用的源文件(`source.path`)。 `rendition.path`是放置资产处理请求最终结果的位置。 自定义应用程序使用回调将本地可用的源文件转换为使用传入的名称(`rendition.path`)的再现文件。 自定义应用程序必须写入`rendition.path`才能创建再现：

```javascript
const { worker } = require('@adobe/asset-compute-sdk');
const fs = require('fs').promises;

// worker() is the entry point in the SDK "framework".
// The asynchronous function defined is the rendition callback.
exports.main = worker(async (source, rendition) => {

    // Tip: custom worker parameters are available in rendition.instructions.
    console.log(rendition.instructions.name); // should print out `rendition.jpg`.

    // Simplest example: copy the source file to the rendition file destination so as to transfer the asset as is without processing.
    await fs.copyFile(source.path, rendition.path);
});
```

### 下载源文件{#download-source}

自定义应用程序只处理本地文件。 下载源文件由[Asset computeSDK](https://github.com/adobe/asset-compute-sdk#adobe-asset-compute-worker-sdk)处理。

### 再现创建{#rendition-creation}

SDK为每个再现调用异步[再现回调函数](https://github.com/adobe/asset-compute-sdk#rendition-callback-for-worker-required)。

回调函数可访问[源](https://github.com/adobe/asset-compute-sdk#source)和[再现](https://github.com/adobe/asset-compute-sdk#rendition)对象。 `source.path`已存在，是源文件的本地副本的路径。 `rendition.path`是必须存储已处理再现的路径。 除非设置[disableSourceDownload标志](https://github.com/adobe/asset-compute-sdk#worker-options-optional)，否则应用程序必须完全使用`rendition.path`。 否则，SDK无法找到或标识再现文件并失败。

对示例进行过度简化，以说明和重点介绍自定义应用程序的剖析。 应用程序只将源文件复制到再现目标。

有关再现回调参数的详细信息，请参阅[Asset computeSDK API](https://github.com/adobe/asset-compute-sdk#api-details)。

### 上传演绎版{#upload-rendition}

创建每个再现并将其存储在具有`rendition.path`提供的路径的文件中后，[Asset computeSDK](https://github.com/adobe/asset-compute-sdk#adobe-asset-compute-worker-sdk)将每个再现上传到云存储（AWS或Azure）。 当且仅当传入请求具有指向同一应用程序URL的多个再现时，自定义应用程序会同时获取多个再现。 上传到云存储在每个再现之后以及为下一个再现运行回调之前完成。

`batchWorker()`具有不同的行为，因为它实际处理所有再现，并且只有在所有已处理的演绎版都上载了这些演绎版。

## [!DNL Adobe I/O] 事件 {#aio-events}

SDK会为每个再现发送[!DNL Adobe I/O]事件。 根据结果，这些事件为`rendition_created`或`rendition_failed`类型。 有关Asset compute详细信息，请参见[事件异步事件](api.md#asynchronous-events)。

## 接收[!DNL Adobe I/O]事件{#receive-aio-events}

客户机根据其消耗逻辑轮询[[!DNL Adobe I/O] 事件日志](https://www.adobe.io/apis/experienceplatform/events/ioeventsapi.html#/Journaling)。 初始日志URL是`/register` API响应中提供的URL。 事件可以使用事件中存在的`requestId`进行标识，该值与`/process`中返回的值相同。 每个再现都有一个单独的事件，在再现上传（或失败）后立即发送。 在收到匹配的事件后，客户端可以显示或以其他方式处理生成的演绎版。

JavaScript库[`asset-compute-client`](https://github.com/adobe/asset-compute-client#usage)使用`waitActivation()`方法简化日志轮询以获取所有事件。

```javascript
const events = await assetCompute.waitActivation(requestId);
await Promise.all(events.map(event => {
    if (event.type === "rendition_created") {
        // get rendition from cloud storage location
    }
    else if (event.type === "rendition_failed") {
        // failed to process
    }
    else {
        // other event types
        // (could be added in the future)
    }
}));
```

有关如何获取日志事件的详细信息，请参阅[[!DNL Adobe I/O] 事件API](https://www.adobe.io/apis/experienceplatform/events/ioeventsapi.html#!adobedocs/adobeio-events/master/events-api-reference.yaml)。

<!-- TBD:
* Illustration of the controls/data flow.
* Basic overview, in text and not code, of how an application works.
-->
