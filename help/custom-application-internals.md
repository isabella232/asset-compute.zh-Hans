---
title: 了解自定义应用程序的工作情况
description: ' [!DNL Asset Compute Service] 自定义应用程序的内部工作，以帮助了解其工作方式。'
exl-id: a3ee6549-9411-4839-9eff-62947d8f0e42
source-git-commit: 187a788d036f33b361a0fd1ca34a854daeb4a101
workflow-type: tm+mt
source-wordcount: '751'
ht-degree: 0%

---

# 自定义应用程序{#how-custom-application-works}的内部

通过下图，可了解当客户使用自定义应用程序处理数字资产时的端对端工作流程。

![自定义应用程序工作流](assets/customworker.png)

*图：使用处理资产时涉及的步 [!DNL Asset Compute Service]骤。*

## 注册 {#registration}

客户端必须在向[`/process`](api.md#process-request)发出第一个请求之前调用[`/register`](api.md#register)一次，以设置并检索日志URL，以接收[!DNL Adobe I/O]事件以进行AdobeAsset compute。

```sh
curl -X POST \
  https://asset-compute.adobe.io/register \
  -H "x-ims-org-id: $ORG_ID" \
  -H "x-gw-ims-org-id: $ORG_ID" \
  -H "Authorization: Bearer $JWT_TOKEN" \
  -H "x-api-key: $API_KEY"
```

[`@adobe/asset-compute-client`](https://github.com/adobe/asset-compute-client#usage) JavaScript库可在NodeJS应用程序中使用，以处理从注册、处理到异步事件处理等所有必需步骤。 有关所需标头的更多信息，请参阅[Authentication and Authorization](api.md)。

## 正在处理 {#processing}

客户端发送一个[正在处理的](api.md#process-request)请求。

```sh
curl -X POST \
  https://asset-compute.adobe.io/process \
  -H "x-ims-org-id: $ORG_ID" \
  -H "x-gw-ims-org-id: $ORG_ID" \
  -H "Authorization: Bearer $JWT_TOKEN" \
  -H "x-api-key: $API_KEY" \
  -d "<RENDITION_JSON>
```

客户端负责使用预签名URL正确设置演绎版的格式。 [`@adobe/node-cloud-blobstore-wrapper`](https://github.com/adobe/node-cloud-blobstore-wrapper#presigned-urls) JavaScript库可在NodeJS应用程序中用于预签URL。 目前，库仅支持Azure Blob Storage和AWS S3容器。

处理请求返回一个`requestId`，可用于轮询[!DNL Adobe I/O]事件。

下面是自定义应用程序处理请求的示例。

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

[!DNL Asset Compute Service]将自定义应用程序呈现请求发送到自定义应用程序。 它对提供的应用程序URL使用HTTPPOST，该URL是Project Firefly提供的安全Web操作URL。 所有请求都使用HTTPS协议来最大限度地提高数据安全性。

自定义应用程序使用的[Asset computeSDK](https://github.com/adobe/asset-compute-sdk#adobe-asset-compute-worker-sdk)可处理HTTPPOST请求。 它还可处理源的下载、上传演绎版、发送[!DNL Adobe I/O]事件和错误处理。

<!-- TBD: Add the application diagram. -->

### 应用程序代码{#application-code}

自定义代码只需提供一个回调即可获取本地可用的源文件(`source.path`)。 `rendition.path`是放置资产处理请求最终结果的位置。 自定义应用程序使用回调将本地可用的源文件转换为使用传入的名称(`rendition.path`)的演绎版文件。 自定义应用程序必须写入`rendition.path`才能创建演绎版：

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

自定义应用程序仅处理本地文件。 下载源文件由[Asset computeSDK](https://github.com/adobe/asset-compute-sdk#adobe-asset-compute-worker-sdk)处理。

### 再现创建{#rendition-creation}

SDK会为每个演绎版调用异步[演绎版回调函数](https://github.com/adobe/asset-compute-sdk#rendition-callback-for-worker-required)。

回调函数对[source](https://github.com/adobe/asset-compute-sdk#source)和[rendition](https://github.com/adobe/asset-compute-sdk#rendition)对象具有访问权限。 `source.path`已存在，是源文件的本地副本的路径。 `rendition.path`是必须存储已处理的再现的路径。 除非设置了[disableSourceDownload标志](https://github.com/adobe/asset-compute-sdk#worker-options-optional)，否则应用程序必须完全使用`rendition.path`。 否则，SDK无法找到或识别呈现版本文件，并且失败。

该示例的过度简化是为了说明和重点介绍自定义应用程序的结构。 应用程序只会将源文件复制到演绎版目标。

有关演绎版回调参数的更多信息，请参阅[Asset computeSDK API](https://github.com/adobe/asset-compute-sdk#api-details)。

### 上传演绎版{#upload-rendition}

创建每个呈现版本并将其存储在具有`rendition.path`提供路径的文件中后，[Asset computeSDK](https://github.com/adobe/asset-compute-sdk#adobe-asset-compute-worker-sdk)会将每个呈现版本上传到云存储（AWS或Azure）。 当且仅当传入请求具有指向同一应用程序URL的多个演绎版时，自定义应用程序才会同时获取多个演绎版。 将上传到云存储的操作在每个演绎版之后，并在为下一个演绎版运行回调之前完成。

`batchWorker()`的行为不同，因为它实际上会处理所有演绎版，并且仅在所有演绎版都经过处理后，才上传这些演绎版。

## [!DNL Adobe I/O] 事件 {#aio-events}

SDK会为每个演绎版发送[!DNL Adobe I/O]事件。 这些事件的类型为`rendition_created`或`rendition_failed`，具体取决于结果。 有关事件详细信息，请参阅[Asset compute异步事件](api.md#asynchronous-events)。

## 接收[!DNL Adobe I/O]事件{#receive-aio-events}

客户端根据其使用逻辑轮询[[!DNL Adobe I/O] 事件日志](https://www.adobe.io/apis/experienceplatform/events/ioeventsapi.html#/Journaling)。 初始日志URL是在`/register` API响应中提供的日志URL。 事件可以使用事件中存在的`requestId`进行标识，该事件与`/process`中返回的事件相同。 每个演绎版都有一个单独的事件，该事件会在上传演绎版后立即发送（或失败）。 当客户端收到匹配事件后，便可以显示或以其他方式处理生成的演绎版。

JavaScript库[`asset-compute-client`](https://github.com/adobe/asset-compute-client#usage)使用`waitActivation()`方法可简化日志轮询以获取所有事件。

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
