---
title: 了解自定义应用程序的工作。
description: 自定义应 [!DNL Asset Compute Service] 用程序的内部工作，有助于了解其工作方式。
translation-type: tm+mt
source-git-commit: 54afa44d8d662ee1499a385f504fca073ab6c347
workflow-type: tm+mt
source-wordcount: '774'
ht-degree: 0%

---


# 自定义应用程序的内部 {#how-custom-application-works}

使用下图可了解当客户端使用自定义应用程序处理数字资产时的端到端工作流程。

![自定义应用程序工作流程](assets/customworker.png)

*图：使用处理资产时涉及的步骤 [!DNL Asset Compute Service]。*

## 注册 {#registration}

客户端必须在 [`/register`](api.md#register) 第一个请求前调用一次 [`/process`](api.md#process-request) ，以设置和检索日志URL，以便接收AdobeI/O事件，用于Adobe资产计算。

```sh
curl -X POST \
  https://asset-compute.adobe.io/register \
  -H "x-ims-org-id: $ORG_ID" \
  -H "x-gw-ims-org-id: $ORG_ID" \
  -H "Authorization: Bearer $JWT_TOKEN" \
  -H "x-api-key: $API_KEY"
```

JavaScript [`@adobe/asset-compute-client`](https://github.com/adobe/asset-compute-client#usage) 库可用于NodeJS应用程序中，以处理从注册、处理到异步事件处理的所有必要步骤。 有关所需标头的详细信息，请参 [阅身份验证和授权](api.md)。

## 正在处理 {#processing}

客户端发送处 [理请](api.md#process-request) 求。

```sh
curl -X POST \
  https://asset-compute.adobe.io/process \
  -H "x-ims-org-id: $ORG_ID" \
  -H "x-gw-ims-org-id: $ORG_ID" \
  -H "Authorization: Bearer $JWT_TOKEN" \
  -H "x-api-key: $API_KEY" \
  -d "<RENDITION_JSON>
```

客户端负责使用预签名URL正确设置演绎版格式。 JavaScript [`@adobe/node-cloud-blobstore-wrapper`](https://github.com/adobe/node-cloud-blobstore-wrapper#presigned-urls) 库可用于NodeJS应用程序中预签URL。 目前，该库仅支持Azure Blob存储和AWS S3容器。

处理请求返回 `requestId` 可用于轮询AdobeI/O事件的请求。

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

将自 [!DNL Asset Compute Service] 定义应用程序再现请求发送到自定义应用程序。 它对提供的应用程序URL使用HTTPPOST，该URL是来自Project Firefly的安全Web操作URL。 所有请求都使用HTTPS协议来最大化数据安全性。

自定 [义应用程序使](https://github.com/adobe/asset-compute-sdk#adobe-asset-compute-worker-sdk) 用的资产计算SDK可处理HTTPPOST请求。 它还处理源的下载、上传再现、发送I/O事件和错误处理。

<!-- TBD: Add the application diagram. -->

### Application code {#application-code}

自定义代码只需要提供一个回调，它获取本地可用的源文件(`source.path`)。 是 `rendition.path` 放置资产处理请求最终结果的位置。 自定义应用程序使用回调将本地可用的源文件转换为使用传入()的名称的再现`rendition.path`文件。 自定义应用程序必须写入才能 `rendition.path` 创建再现：

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

### 下载源文件 {#download-source}

自定义应用程序只处理本地文件。 下载源文件由资产计算 [SDK处理](https://github.com/adobe/asset-compute-sdk#adobe-asset-compute-worker-sdk)。

### 再现创建 {#rendition-creation}

SDK为每个再现调 [用异步再现](https://github.com/adobe/asset-compute-sdk#rendition-callback-for-worker-required) 回调函数。

回调函数有权访问源 [和再现](https://github.com/adobe/asset-compute-sdk#source)[对象](https://github.com/adobe/asset-compute-sdk#rendition) 。 源 `source.path` 文件的本地副本的路径已存在。 该 `rendition.path` 路径是必须存储已处理再现的路径。 除非 [设置了disableSourceDownload](https://github.com/adobe/asset-compute-sdk#worker-options-optional) 标志，否则应用程序必须完全使用 `rendition.path`。 否则，SDK无法找到或标识再现文件并失败。

对示例进行过度简化，以说明和重点介绍自定义应用程序的剖析。 应用程序只将源文件复制到再现目标。

有关再现回调参数的详细信息，请 [参阅资产计算SDK API](https://github.com/adobe/asset-compute-sdk#api-details)。

### 上传演绎版 {#upload-rendition}

创建每个再现并将其存储在文件中，其路径 `rendition.path`由提 [供，资产计算SDK](https://github.com/adobe/asset-compute-sdk#adobe-asset-compute-worker-sdk) 会将每个再现上传到云存储（AWS或Azure）。 当且仅当传入请求具有指向同一应用程序URL的多个再现时，自定义应用程序会同时获取多个再现。 上传到云存储在每个再现之后以及为下一个再现运行回调之前完成。

该演 `batchWorker()` 绎版具有不同的行为，因为它实际处理所有演绎版，并且只在所有演绎版都经过处理后，才上传这些演绎版。

## AdobeI/O事件 {#aio-events}

SDK会为每个再现发送AdobeI/O事件。 这些事件是类 `rendition_created` 型 `rendition_failed` 或取决于结果。 有关 [事件详细](api.md#asynchronous-events) 信息，请参阅资产计算异步事件。

## 接收AdobeI/O事件 {#receive-aio-events}

客户端根据 [AdobeI/O事件](https://www.adobe.io/apis/experienceplatform/events/ioeventsapi.html#/Journaling) ，根据其消耗逻辑轮询I/O日志。 初始日志URL是API响应中提供 `/register` 的URL。 事件可以使用 `requestId` 中存在且与中返回的事件相同的标识 `/process`。 每个再现都有一个单独的事件，在再现上传（或失败）后立即发送。 在收到匹配的事件后，客户端可以显示或以其他方式处理生成的演绎版。

JavaScript库使 [`asset-compute-client`](https://github.com/adobe/asset-compute-client#usage) 日志轮询更简单， `waitActivation()` 使用该方法获取所有事件。

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

有关如何获取日志事件的详细信息，请 [参阅AdobeI/O事件API](https://www.adobe.io/apis/experienceplatform/events/ioeventsapi.html#!adobedocs/adobeio-events/master/events-api-reference.yaml)。

<!-- TBD:
* Illustration of the controls/data flow.
* Basic overview, in text and not code, of how an application works.
-->
