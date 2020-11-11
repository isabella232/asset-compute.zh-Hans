---
title: '[!DNL Asset Compute Service] HTTP API。'
description: '[!DNL Asset Compute Service] 用于创建自定义应用程序的HTTP API。'
translation-type: tm+mt
source-git-commit: 79630efa8cee2c8919d11e9bb3c14ee4ef54d0f3
workflow-type: tm+mt
source-wordcount: '2925'
ht-degree: 2%

---


# [!DNL Asset Compute Service] HTTP API {#asset-compute-http-api}

API的使用仅限于开发目的。 在开发自定义应用程序时，API作为上下文提供。 [!DNL Adobe Experience Manager] 因为Cloud Service使用API将处理信息传递给自定义应用程序。 有关详细信息，请参 [阅使用资产微服务和处理用户档案](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html)。

>[!NOTE]
>
>[!DNL Asset Compute Service] 只能与Cloud Service [!DNL Experience Manager] 一起使用。

HTTP API的任何客 [!DNL Asset Compute Service] 户端都必须遵循以下高级流：

1. 在IMS组织中，将 [!DNL Adobe Developer Console] 客户端设置为项目。 每个单独的客户端(系统或环境)都需要其自己的单独项目，以分离事件数据流。

1. 客户端使用JWT（服务帐户）身份验证 [为技术帐户生成访问令牌](https://www.adobe.io/authentication/auth-methods.html)。

1. 客户端仅 [`/register`](#register) 调用一次以检索日志URL。

1. 客户端会 [`/process`](#process-request) 为每个要为其生成演绎版的资产调用。 呼叫是异步的。

1. 客户端定期轮询日志以 [接收事件](#asynchronous-events)。 当再现成功处理(事件)或出现错误(事件类型`rendition_created` )时，它会接收每个请求的再现的`rendition_failed` (事件类型)。

@ [adobe/asset-compute-client](https://github.com/adobe/asset-compute-client) module使API在Node.js代码中更易于使用。

## 身份验证和授权 {#authentication-and-authorization}

所有API都需要访问令牌身份验证。 请求必须设置以下标题：

1. `Authorization` header with bearer token，它是技术帐户令牌，通过JWT [exchange从Adobe开发](https://www.adobe.io/authentication/auth-methods.html) 者控制台项目接收。 范 [围](#scopes) 见下文。

<!-- TBD: Change the existing URL to a new path when a new path for docs is available. The current path contains master word that is not an inclusive term. Logged ticket in AIO's GitHub repo to get a new URL.
-->

1. `x-gw-ims-org-id` 头和IMS组织ID。

1. `x-api-key` 和项目的客户端 [!DNL Adobe Developers Console] ID。

### 范围 {#scopes}

确保访问令牌的以下范围：

* `openid`
* `AdobeID`
* `asset_compute`
* `read_organizations`
* `event_receiver`
* `event_receiver_api`
* `adobeio_api`
* `additional_info.roles`
* `additional_info.projectedProductContext`

这些要求 [!DNL Adobe Developer Console] 项目需要订阅 `Asset Compute`、 `I/O Events`和服 `I/O Management API` 务。 各个范围的细分是：

* 基本
   * scopes: `openid,AdobeID`

* asset compute
   * metascope: `asset_compute_meta`
   * scopes: `asset_compute,read_organizations`

* AdobeI/O事件
   * metascope: `event_receiver_api`
   * scopes: `event_receiver,event_receiver_api`

* AdobeI/O管理API
   * metascope: `ent_adobeio_sdk`
   * scopes: `adobeio_api,additional_info.roles,additional_info.projectedProductContext`

## 注册 {#register}

每个客户端( [!DNL Asset Compute service] 订阅服务的 [!DNL Adobe Developer Console] 唯一项目)必须在 [发出处理请](#register-request) 求之前进行注册。 注册步骤返回唯一事件日志，该事件是从再现处理中检索异步所必需的。

在客户端生命周期结束时，可以取消注 [册](#unregister-request)。

### 注册请求 {#register-request}

此API调用设置客 [!DNL Asset Compute] 户端并提供事件日志URL。 这是一个幂等运算，每个客户端只需调用一次。 可再次调用它以检索日志URL。

| 参数 | 值 |
|--------------------------|------------------------------------------------------|
| 方法 | `POST` |
| 路径 | `/register` |
| 标题 `Authorization` | 所有与 [授权相关的标头](#authentication-and-authorization)。 |
| 标题 `x-request-id` | 可选，可由客户端为跨系统处理请求的唯一端到端标识符进行设置。 |
| 请求主体 | 必须为空。 |

### 注册响应 {#register-response}

| 参数 | 值 |
|-----------------------|------------------------------------------------------|
| MIME类型 | `application/json` |
| 标题 `X-Request-Id` | 与请求标头相 `X-Request-Id` 同，或是唯一生成的标头。 用于识别跨系统和／或支持请求的请求。 |
| 响应体 | 包含、 `journal`和/ `ok` 或字段的 `requestId` JSON对象。 |

HTTP状态代码为：

* **200次成功**:请求成功时。 它包含通 `journal` 过(成功时或失败时作为事件类型)触发的异步处理 `/process` 的任何结果 `rendition_created` 的通知URL `rendition_failed` 。

   ```json
   {
       "ok": true,
       "journal": "https://api.adobe.io/events/organizations/xxxxx/integrations/xxxx/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
       "requestId": "1234567890"
   }
   ```

* **401未经授权**:当请求没有有效身份验证时 [发生](#authentication-and-authorization)。 示例可能是无效访问令牌或无效API密钥。

* **403禁止**:当请求没有有效授权时 [发生](#authentication-and-authorization)。 示例可能是有效访问令牌，但Adobe开发者控制台项目（技术帐户）未订阅所有必需的服务。

* **429请求太多**:当系统由此客户端过载时或以其他方式发生。 客户端应在出现指数退 [避的情况下重试](https://en.wikipedia.org/wiki/Exponential_backoff)。 尸体是空的。
* **4xx错误**:出现任何其他客户端错误且注册失败时。 通常会返回此类JSON响应，但并不能保证所有错误都会返回：

   ```json
   {
       "ok": false,
       "requestId": "1234567890",
       "message": "error message"
   }
   ```

* **5xx错误**:出现任何其他服务器端错误且注册失败时发生。 通常会返回此类JSON响应，但并不能保证所有错误都会返回：

   ```json
   {
       "ok": false,
       "requestId": "1234567890",
       "message": "error message"
   }
   ```

### 注销请求 {#unregister-request}

此API调用取消注册客 [!DNL Asset Compute] 户端。 之后，再也无法打电话 `/process`。 对未注册的客户端或尚未注册的客户端使用API调用会返回错 `404` 误。

| 参数 | 值 |
|--------------------------|------------------------------------------------------|
| 方法 | `POST` |
| 路径 | `/unregister` |
| 标题 `Authorization` | 所有与 [授权相关的标头](#authentication-and-authorization)。 |
| 标题 `x-request-id` | 可选，可由客户端为跨系统处理请求的唯一端到端标识符进行设置。 |
| 请求主体 | 空. |

### 注销响应 {#unregister-response}

| 参数 | 值 |
|-----------------------|------------------------------------------------------|
| MIME类型 | `application/json` |
| 标题 `X-Request-Id` | 与请求标头相 `X-Request-Id` 同，或是唯一生成的标头。 用于识别跨系统和／或支持请求的请求。 |
| 响应体 | 包含和字段 `ok` 的JSON `requestId` 对象。 |

状态代码为：

* **200次成功**:在找到并删除注册和日志时发生。

   ```json
   {
       "ok": true,
       "requestId": "1234567890"
   }
   ```

* **401未经授权**:当请求没有有效身份验证时 [发生](#authentication-and-authorization)。 示例可能是无效访问令牌或无效API密钥。

* **403禁止**:当请求没有有效授权时 [发生](#authentication-and-authorization)。 示例可能是有效访问令牌，但Adobe开发者控制台项目（技术帐户）未订阅所有必需的服务。

* **404未找到**:当给定凭据没有当前注册时发生。

   ```json
   {
       "ok": true,
       "requestId": "1234567890"
   }
   ```

* **429请求太多**:在系统过载时发生。 客户端应在出现指数退 [避的情况下重试](https://en.wikipedia.org/wiki/Exponential_backoff)。 尸体是空的。

* **4xx错误**:发生任何其他客户端错误并注销失败的情况。 通常会返回此类JSON响应，但并不能保证所有错误都会返回：

   ```json
   {
       "ok": false,
       "requestId": "1234567890",
       "message": "error message"
   }
   ```

* **5xx错误**:出现任何其他服务器端错误且注册失败时发生。 通常会返回此类JSON响应，但并不能保证所有错误都会返回：

   ```json
   {
       "ok": false,
       "requestId": "1234567890",
       "message": "error message"
   }
   ```

## 处理请求 {#process-request}

此操 `process` 作会根据请求中的说明，提交将源资产转换为多个演绎版的作业。 有关成功完成(事件类型 `rendition_created`)或任何错误(事件类型 `rendition_failed`)的通知会发送到事件日志，在发出任意数量的请求之前，必须使用／注册一次 [才能检索该](#register)`/process` 。 格式不正确的请求会立即失败，并显示400错误代码。

二进制文件使用URL(如AmazonAWS S3预签名URL或Azure Blob存储SAS URL)进行引用，以读取资 `source` 产(`GET` URL)和写入再现(`PUT` URL)。 客户端负责生成这些预签名的URL。

| 参数 | 值 |
|--------------------------|------------------------------------------------------|
| 方法 | `POST` |
| 路径 | `/process` |
| MIME类型 | `application/json` |
| 标题 `Authorization` | 所有与 [授权相关的标头](#authentication-and-authorization)。 |
| 标题 `x-request-id` | 可选，可由客户端为跨系统处理请求的唯一端到端标识符进行设置。 |
| 请求主体 | 必须采用流程请求JSON格式，如下所述。 它提供了有关要处理的资产和要生成的演绎版的说明。 |

### 进程请求JSON {#process-request-json}

请求主体 `/process` 是具有此高级模式的JSON对象：

```json
{
    "source": "",
    "renditions" : []
}
```

可用字段包括：

| 名称 | 类型 | 描述 | 示例 |
|--------------|----------|-------------|---------|
| `source` | `string` | 要处理的源资产的URL。 根据请求的再现格式(例如， `fmt=zip`)。 | `"http://example.com/image.jpg"` |
| `source` | `object` | 描述要处理的源资产。 请参阅下面 [的源对象字段](#source-object-fields) 说明。 根据请求的再现格式(例如， `fmt=zip`)。 | `{"url": "http://example.com/image.jpg", "mimeType": "image/jpeg" }` |
| `renditions` | `array` | 要从源文件生成的演绎版。 每个再现对象都支持 [再现说明](#rendition-instructions)。 必填. | `[{ "target": "https://....", "fmt": "png" }]` |

它 `source` 可以是 `<string>` 被视为URL的，也可以是带有 `<object>` 附加字段的。 以下变体类似：

```json
"source": "http://example.com/image.jpg"
```

```json
"source": {
    "url": "http://example.com/image.jpg"
}
```

### 源对象字段 {#source-object-fields}

| 名称 | 类型 | 描述 | 示例 |
|-----------|----------|-------------|---------|
| `url` | `string` | 要处理的源资产的URL。 必填. | `"http://example.com/image.jpg"` |
| `name` | `string` | 源资产文件名。 如果无法检测到MIME类型，则可以使用名称中的文件扩展名。 优先于URL路径中的文件名或二进制资 `content-disposition` 源标题中的文件名。 默认为“文件”。 | `"image.jpg"` |
| `size` | `number` | 源资产文件大小（以字节为单位）。 优先于二 `content-length` 进制资源的标头。 | `10234` |
| `mimetype` | `string` | 源资产文件MIME类型。 优先于二进制 `content-type` 资源的标头。 | `"image/jpeg"` |

### 完整的请 `process` 求示例 {#complete-process-request-example}

```json
{
    "source": "https://www.adobe.com/content/dam/acom/en/lobby/lobby-bg-bts2017-logged-out-1440x860.jpg",
    "renditions" : [{
            "name": "image.48x48.png",
            "target": "https://some-presigned-put-url-for-image.48x48.png",
            "fmt": "png",
            "width": 48,
            "height": 48
        },{
            "name": "image.200x200.jpg",
            "target": "https://some-presigned-put-url-for-image.200x200.jpg",
            "fmt": "jpg",
            "width": 200,
            "height": 200
        },{
            "name": "cqdam.xmp.xml",
            "target": "https://some-presigned-put-url-for-cqdam.xmp.xml",
            "fmt": "xmp"
        },{
            "name": "cqdam.text.txt",
            "target": "https://some-presigned-put-url-for-cqdam.text.txt",
            "fmt": "text"
    }]
}
```

## 进程响应 {#process-response}

请 `/process` 求会立即返回，并基于基本请求验证成功或失败。 实际资产处理以异步方式进行。

| 参数 | 值 |
|-----------------------|------------------------------------------------------|
| MIME类型 | `application/json` |
| 标题 `X-Request-Id` | 与请求标头相 `X-Request-Id` 同，或是唯一生成的标头。 用于识别跨系统和／或支持请求的请求。 |
| 响应体 | 包含和字段 `ok` 的JSON `requestId` 对象。 |

状态代码：

* **200次成功**:如果请求已成功提交。 响应JSON包括 `"ok": true`:

   ```json
   {
       "ok": true,
       "requestId": "1234567890"
   }
   ```

* **400无效请求**:如果请求的格式不正确，例如请求JSON中缺少必填字段。 响应JSON包括 `"ok": false`:

   ```json
   {
       "ok": false,
       "requestId": "1234567890",
       "message": "error message"
   }
   ```

* **401未经授权**:当请求没有有效的身份 [验证](#authentication-and-authorization)。 示例可能是无效访问令牌或无效API密钥。
* **403禁止**:当请求没有有效的授权 [时](#authentication-and-authorization)。 示例可能是有效访问令牌，但Adobe开发者控制台项目（技术帐户）未订阅所有必需的服务。
* **429请求太多**:当系统由此客户端过载时，或者通常。 客户端可以在指数退 [出时重试](https://en.wikipedia.org/wiki/Exponential_backoff)。 尸体是空的。
* **4xx错误**:出现任何其他客户端错误时。 通常会返回此类JSON响应，但并不能保证所有错误都会返回：

   ```json
   {
       "ok": false,
       "requestId": "1234567890",
       "message": "error message"
   }
   ```

* **5xx错误**:发生任何其他服务器端错误时。 通常会返回此类JSON响应，但并不能保证所有错误都会返回：

   ```json
   {
       "ok": false,
       "requestId": "1234567890",
       "message": "error message"
   }
   ```

大多数客户端可能会在出现任何错误时， [重试与同一请求](https://en.wikipedia.org/wiki/Exponential_backoff) ，但 ** 配置问题（如401或403）或无效请求（如400）除外。 除了通过429个响应进行常规速率限制外，临时服务中断或限制可能导致5xx错误。 然后，建议在一段时间后重试。

所有JSON响应（如果存在） `requestId` 都包含与标题相同的 `X-Request-Id` 值。 建议从标题中读取，因为它始终存在。 在与 `requestId` 处理请求相关的所有事件中，也会返回 `requestId`。 客户端不能对此字符串的格式做出任何假设，它是一个不透明的字符串标识符。

## 选择加入后期处理 {#opt-in-to-post-processing}

asset compute [SDK](https://github.com/adobe/asset-compute-sdk) 支持一组基本的图像后处理选项。 通过将再现对选择加入象上的字段设置为，自定 `postProcess` 义Worker可显式地进行后处理 `true`。

支持的用例有：

* 将演绎版裁切为限制由crop.w、crop.h、crop.x和crop.y定义的矩形。它由再现 `instructions.crop` 对象中定义。
* 使用宽度、高度或两者调整图像大小。 它由再现对 `instructions.width` 象 `instructions.height` 定义，并在再现对象中定义。 要仅使用宽度或高度调整大小，请仅设置一个值。 Compute Service可以节省宽高比。
* 设置JPEG图像的质量。 它由再现 `instructions.quality` 对象中定义。 最佳质量由表示，较 `100` 小的值表示质量降低。
* 创建隔行扫描图像。 它由再现 `instructions.interlace` 对象中定义。
* 设置DPI，通过调整应用于像素的比例来调整呈现大小以用于桌面发布。 它由再现对 `instructions.dpi` 象中定义，以更改dpi分辨率。 但是，要调整图像大小，使其在不同的分辨率下大小相同，请使用说 `convertToDpi` 明。
* 调整图像大小，使其呈现的宽度或高度在指定的目标分辨率(DPI)下保持与原始图像相同。 它由再现 `instructions.convertToDpi` 对象中定义。

## 水印资产 {#add-watermark}

asset compute [SDK](https://github.com/adobe/asset-compute-sdk) 支持向PNG、JPEG、TIFF和GIF图像文件添加水印。 将按照再现对象中的再现说明 `watermark` 添加水印。

在再现后处理过程中，将执行水印操作。 要对资产设置水印，自定 [义工作者可通过](#opt-in-to-post-processing) 将演绎版对象 `postProcess` 上的字段设置为进行后处理 `true`。 如果工作者不选择加入，则不应用水印，即使在请求中的再现对象上设置了水印对象也是如此。

## 再现说明 {#rendition-instructions}

这些选项是／处理中 `renditions` 的阵列 [的可用选项](#process-request)。

### 常用字段 {#common-fields}

| 名称 | 类型 | 描述 | 示例 |
|-------------------|----------|-------------|---------|
| `fmt` | `string` | 演绎版目标格式也可用于文 `text` 本提取和将XMP元 `xmp` 元数据提取为xml。 查看支 [持的格式](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/file-format-support.html) | `png` |
| `worker` | `string` | 自定义应用 [程序的URL](develop-custom-application.md)。 必须是 `https://` URL。 如果此字段存在，则再现由自定义应用程序创建。 然后，自定义应用程序中将使用任何其他设置的再现字段。 | `"https://1234.adobeioruntime.net`<br>`/api/v1/web`<br>`/example-custom-worker-master/worker"` |
| `target` | `string` | 应使用HTTPPUT将生成的演绎版上传到的URL。 | `http://w.com/img.jpg` |
| `target` | `object` | 生成的再现的多部分预签名URL上传信息。 这适用于具 [有此多部件上传行为的AEM](https://jackrabbit.apache.org/oak/docs/features/direct-binary-access.html) / [Oak Direct Binary Upload](http://jackrabbit.apache.org/oak/docs/apidocs/org/apache/jackrabbit/api/binary/BinaryUpload.html)。<br>字段:<ul><li>`urls`:字符串数组，每个预签名部件URL对应一个</li><li>`minPartSize`:用于一个部件的最小大小= url</li><li>`maxPartSize`:用于一个部件的最大大小= url</li></ul> | `{ "urls": [ "https://part1...", "https://part2..." ], "minPartSize": 10000, "maxPartSize": 100000 }` |
| `userData` | `object` | 由客户端控制并按原样传递到再现事件的可选保留空间。 允许客户端添加自定义信息以标识再现事件。 在自定义应用程序中不得修改或依赖客户端，因为客户端随时可以自由更改。 | `{ ... }` |

### 演绎版特定字段 {#rendition-specific-fields}

有关当前支持的文件格式的列表，请参 [阅支持的文件格式](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/file-format-support.html)。

| 名称 | 类型 | 描述 | 示例 |
|-------------------|----------|-------------|---------|
| `*` | `*` | 可以添加自定义应用程序所理解的 [高级自定](develop-custom-application.md) 义字段。 |  |
| `embedBinaryLimit` | `number` 字节 | 如果设置此值，且再现的文件大小小于此值，则再现将嵌入在再现生成完成后发送的事件中。 嵌入允许的最大大小为32 KB（32 x 1024字节）。 如果再现的大小大于限 `embedBinaryLimit` 制，则它将放在云存储中的某个位置，而不嵌入事件。 | `3276` |
| `width` | `number` | 宽度（以像素为单位）. 仅适用于图像再现。 | `200` |
| `height` | `number` | 高度（以像素为单位）. 仅适用于图像再现。 | `200` |
|  |  | 如果： <ul> <li> 同 `width` 时指 `height` 定和，然后在保持长宽比的同时，图像适合大小 </li><li> 仅 `width` 指定 `height` 或仅指定，结果图像在保持宽高比的同时使用相应的尺寸</li><li> 如果既未 `width` 指 `height` 定也未指定，则使用原始图像像素大小。 它取决于源类型。 对于某些格式（如PDF文件），使用默认大小。 可以存在最大大小限制。</li></ul> |  |
| `quality` | `number` | 在至范围内指定jpeg `1` 品 `100`质。 仅适用于图像演绎版。 | `90` |
| `xmp` | `string` | 只供XMP元数据写回使用，它是base64编码的XMP以写回到指定的再现。 |  |
| `interlace` | `bool` | 通过将其设置为，创建隔行扫描PNG或GIF或逐行JPEG `true`。 它对其他文件格式没有任何影响。 |  |
| `jpegSize` | `number` | JPEG文件的大小（以字节为单位）。 它将覆盖任何 `quality` 设置。 对其他格式没有影响。 |  |
| `dpi` | `number` 或 `object` | 设置x和y DPI。 为了简单起见，还可以将它设置为单个数字，该数字同时用于x和y。它对图像本身没有影响。 | `96` 或 `{ xdpi: 96, ydpi: 96 }` |
| `convertToDpi` | `number` 或 `object` | x和y DPI在保持物理大小的同时重新取样值。 为了简单起见，还可以将它设置为单个数字，该数字同时用于x和y。 | `96` 或 `{ xdpi: 96, ydpi: 96 }` |
| `files` | `array` | 要包含在ZIP归档中的文件列表(`fmt=zip`)。 每个条目可以是URL字符串或包含以下字段的对象：<ul><li>`url`:下载文件的URL</li><li>`path`:将文件存储在ZIP中此路径下</li></ul> | `[{ "url": "https://host/asset.jpg", "path": "folder/location/asset.jpg" }]` |
| `duplicate` | `string` | 重复处理ZIP存档(`fmt=zip`)。 默认情况下，在ZIP中存储在同一路径下的多个文件会生成错误。 设置 `duplicate` 为 `ignore` 仅会生成要存储的第一个资产和要忽略的其余资产。 | `ignore` |
| `watermark` | `object` | 包含有关水印的 [说明](#watermark-specific-fields)。 |  |

### 特定于水印的字段 {#watermark-specific-fields}

PNG格式用作水印。

| 名称 | 类型 | 描述 | 示例 |
|-------------------|----------|-------------|---------|
| `scale` | `number` | 在和之间缩放水 `0.0` 印 `1.0`。 `1.0` 表示水印具有其原始比例(1:1)，且较低的值会减小水印大小。 | 值表示 `0.5` 原始大小的一半。 |
| `image` | `url` | 用于水印的PNG文件的URL。 |  |

## 异步事件 {#asynchronous-events}

再现处理完成或出现错误时，事件会发送到 [AdobeI/O事件日志](https://www.adobe.io/apis/experienceplatform/events/documentation.html#!adobedocs/adobeio-events/master/intro/journaling_api.md)。 客户端必须侦听通过／注册提供的日志 [URL](#register)。 日志响应包括由 `event` 每个事件的一个对象组成的数组，该字段 `event` 包括实际事件有效负荷。

所有Adobe的事件I/O事件类型 [!DNL Asset Compute Service] 为 `asset_compute`。 日志仅自动订阅此事件类型，并且不再需要基于AdobeI/O事件类型进行筛选。 服务特定事件类型在事件 `type` 的属性中可用。

### Event types {#event-types}

| 事件 | 描述 |
|---------------------|-------------|
| `rendition_created` | 为每个成功处理和上传的演绎版发送。 |
| `rendition_failed` | 已针对每个无法处理或上传的再现发送。 |

### 事件属性 {#event-attributes}

| 属性 | 类型 | 事件 | 描述 |
|-------------|----------|---------------|-------------|
| `date` | `string` | `*` | 以简化的扩展ISO-8601格 [式发送事件的时间戳](https://en.wikipedia.org/wiki/ISO_8601) ，格式由JavaScript [Date.toISOString()定义](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date/toISOString)。 |
| `requestId` | `string` | `*` | 原始请求的请求ID与标 `/process`头相同 `X-Request-Id` 。 |
| `source` | `object` | `*` | 请 `source` 求 `/process` 的。 |
| `userData` | `object` | `*` | 请求 `userData` 中的演绎版(如 `/process` 果已设置)。 |
| `rendition` | `object` | `rendition_*` | 传入的相应再现对象 `/process`。 |
| `metadata` | `object` | `rendition_created` | 再 [现的](#metadata) 元数据属性。 |
| `errorReason` | `string` | `rendition_failed` | 再现失败 [原因](#error-reasons) （如果有）。 |
| `errorMessage` | `string` | `rendition_failed` | 文本提供了有关再现失败（如果有）的更详细信息。 |

### 元数据 {#metadata}

| 属性 | 描述 |
|--------|-------------|
| `repo:size` | 再现的大小（以字节为单位）。 |
| `repo:sha1` | 再现的sha1摘要。 |
| `dc:format` | 再现的MIME类型。 |
| `repo:encoding` | 再现的字符集编码（如果它是基于文本的格式）。 |
| `tiff:ImageWidth` | 再现的宽度（以像素为单位）。 仅适用于图像再现。 |
| `tiff:ImageLength` | 再现的长度（以像素为单位）。 仅适用于图像再现。 |

### 错误原因 {#error-reasons}

| 原因 | 描述 |
|---------|-------------|
| `RenditionFormatUnsupported` | 给定源不支持请求的再现格式。 |
| `SourceUnsupported` | 不支持特定源，即使支持类型。 |
| `SourceCorrupt` | 源数据已损坏。 包括空文件。 |
| `RenditionTooLarge` | 无法使用中提供的预签名URL上传再现 `target`。 实际的再现大小在中以元数 `repo:size` 据形式可用，并且客户端可以使用正确数量的预签名URL重新处理此再现。 |
| `GenericError` | 任何其他意外错误。 |
