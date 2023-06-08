---
title: "[!DNL Asset Compute Service] HTTP API"
description: ”[!DNL Asset Compute Service] 用于创建自定义应用程序的HTTP API。”
exl-id: 4b63fdf9-9c0d-4af7-839d-a95e07509750
source-git-commit: 5257e091730f3672c46dfbe45c3e697a6555e6b1
workflow-type: tm+mt
source-wordcount: '2906'
ht-degree: 3%

---

# [!DNL Asset Compute Service] HTTP API {#asset-compute-http-api}

API的使用仅限于开发目的。 API在开发自定义应用程序时作为上下文提供。 [!DNL Adobe Experience Manager] as a [!DNL Cloud Service] 使用API将处理信息传递给自定义应用程序。 有关更多信息，请参阅 [使用资源微服务和处理配置文件](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html).

>[!NOTE]
>
>[!DNL Asset Compute Service] 仅适用于以下情况： [!DNL Experience Manager] as a [!DNL Cloud Service].

的任何客户端 [!DNL Asset Compute Service] HTTP API必须遵循以下高级流程：

1. 客户端配置为 [!DNL Adobe Developer Console] IMS组织中的项目。 为了分离事件数据流，每个单独的客户端（系统或环境）需要其自己的独立项目。

1. 客户端使用为技术帐户生成访问令牌 [JWT（服务帐户）身份验证](https://www.adobe.io/authentication/auth-methods.html).

1. 客户端调用 [`/register`](#register) 仅检索一次日志URL。

1. 客户端调用 [`/process`](#process-request) 要为其生成演绎版的每个资源。 调用是异步调用。

1. 客户定期将日志轮询到 [接收事件](#asynchronous-events). 成功处理呈现版本后，它会接收每个所请求呈现版本的事件(`rendition_created` 事件类型)或是否存在错误(`rendition_failed` 事件类型)。

此 [@adobe/asset-compute-client](https://github.com/adobe/asset-compute-client) 通过模块，可以轻松地使用Node.js代码中的API。

## 身份验证和授权 {#authentication-and-authorization}

所有API都需要访问令牌身份验证。 请求必须设置以下标头：

1. `Authorization` 标头带有持有者令牌，它是通过接收的技术帐户令牌 [JWT交换](https://www.adobe.io/authentication/auth-methods.html) 从Adobe Developer Console项目。 此 [范围](#scopes) 记录如下。

<!-- TBD: Change the existing URL to a new path when a new path for docs is available. The current path contains master word that is not an inclusive term. Logged ticket in Adobe I/O's GitHub repo to get a new URL.
-->

1. `x-gw-ims-org-id` 标头具有IMS组织ID。

1. `x-api-key` 客户端ID来自 [!DNL Adobe Developers Console] 项目。

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

这些需要 [!DNL Adobe Developer Console] 要订阅的项目 `Asset Compute`， `I/O Events`、和 `I/O Management API` 服务。 各个范围的划分如下：

* 基本
   * 范围： `openid,AdobeID`

* asset compute
   * metascope： `asset_compute_meta`
   * 范围： `asset_compute,read_organizations`

* [!DNL Adobe I/O] 事件
   * metascope： `event_receiver_api`
   * 范围： `event_receiver,event_receiver_api`

* [!DNL Adobe I/O] 管理API
   * metascope： `ent_adobeio_sdk`
   * 范围： `adobeio_api,additional_info.roles,additional_info.projectedProductContext`

## 注册 {#register}

每个客户 [!DNL Asset Compute service]  — 唯一 [!DNL Adobe Developer Console] 订阅服务的项目 — 必须 [注册](#register-request) ，然后再提出处理请求。 注册步骤将返回从演绎版处理中检索异步事件所需的唯一事件日志。

在其生命周期结束时，客户可以 [取消注册](#unregister-request).

### 注册请求 {#register-request}

此API调用设置 [!DNL Asset Compute] 客户端并提供事件日志URL。 这是幂等操作，每个客户端只需调用一次。 可以再次调用它以检索日志URL。

| 参数 | 价值 |
|--------------------------|------------------------------------------------------|
| 方法 | `POST` |
| 路径 | `/register` |
| 页眉 `Authorization` | 全部 [授权相关标头](#authentication-and-authorization). |
| 页眉 `x-request-id` | 可选，可由客户端跨系统设置处理请求的唯一端到端标识符。 |
| 请求正文 | 必须为空。 |

### 注册响应 {#register-response}

| 参数 | 价值 |
|-----------------------|------------------------------------------------------|
| MIME类型 | `application/json` |
| 页眉 `X-Request-Id` | 与 `X-Request-Id` 请求标头或唯一生成的标头。 用于标识跨系统的请求和/或支持请求。 |
| 响应正文 | 包含的JSON对象 `journal`， `ok` 和/或 `requestId` 字段。 |

HTTP状态代码包括：

* **200项成功**：请求成功时。 它包含 `journal` 通过触发的异步处理的任何结果均通知的URL `/process` (作为事件类型 `rendition_created` 成功时，或 `rendition_failed` 失败时)。

  ```json
  {
      "ok": true,
      "journal": "https://api.adobe.io/events/organizations/xxxxx/integrations/xxxx/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "requestId": "1234567890"
  }
  ```

* **401未授权**：在请求无效时发生 [身份验证](#authentication-and-authorization). 例如，访问令牌或API密钥无效。

* **403禁止访问**：在请求无效时发生 [授权](#authentication-and-authorization). 例如，可能是一个有效的访问令牌，但Adobe Developer Console项目（技术帐户）未订阅所有必需的服务。

* **429请求太多**：当系统被此客户端或其他客户端重载时发生。 客户端应使用 [指数回退](https://en.wikipedia.org/wiki/Exponential_backoff). 正文为空。
* **4xx错误**：当出现任何其他客户端错误并注册失败时。 通常，会返回诸如此类的JSON响应，尽管不能保证在所有错误中都会出现这种情况：

  ```json
  {
      "ok": false,
      "requestId": "1234567890",
      "message": "error message"
  }
  ```

* **5xx错误**：当存在任何其他服务器端错误且注册失败时发生。 通常，会返回诸如此类的JSON响应，尽管不能保证在所有错误中都会出现这种情况：

  ```json
  {
      "ok": false,
      "requestId": "1234567890",
      "message": "error message"
  }
  ```

### 注销请求 {#unregister-request}

此API调用取消注册 [!DNL Asset Compute] 客户。 之后，无法再调用 `/process`. 对未注册的客户端或尚未注册的客户端使用API调用时，会返回 `404` 错误。

| 参数 | 价值 |
|--------------------------|------------------------------------------------------|
| 方法 | `POST` |
| 路径 | `/unregister` |
| 页眉 `Authorization` | 全部 [授权相关标头](#authentication-and-authorization). |
| 页眉 `x-request-id` | 可选，可由客户端跨系统设置处理请求的唯一端到端标识符。 |
| 请求正文 | 空. |

### 注销响应 {#unregister-response}

| 参数 | 价值 |
|-----------------------|------------------------------------------------------|
| MIME类型 | `application/json` |
| 页眉 `X-Request-Id` | 与 `X-Request-Id` 请求标头或唯一生成的标头。 用于标识跨系统的请求和/或支持请求。 |
| 响应正文 | 包含的JSON对象 `ok` 和 `requestId` 字段。 |

状态代码为：

* **200项成功**：找到并删除注册和日志时发生。

  ```json
  {
      "ok": true,
      "requestId": "1234567890"
  }
  ```

* **401未授权**：在请求无效时发生 [身份验证](#authentication-and-authorization). 例如，访问令牌或API密钥无效。

* **403禁止访问**：在请求无效时发生 [授权](#authentication-and-authorization). 例如，可能是一个有效的访问令牌，但Adobe Developer Console项目（技术帐户）未订阅所有必需的服务。

* **404未找到**：当给定凭据当前未注册时发生。

  ```json
  {
      "ok": true,
      "requestId": "1234567890"
  }
  ```

* **429请求太多**：在系统过载时发生。 客户端应使用 [指数回退](https://en.wikipedia.org/wiki/Exponential_backoff). 正文为空。

* **4xx错误**：在出现任何其他客户端错误且注销失败时发生。 通常，会返回诸如此类的JSON响应，尽管不能保证在所有错误中都会出现这种情况：

  ```json
  {
      "ok": false,
      "requestId": "1234567890",
      "message": "error message"
  }
  ```

* **5xx错误**：当存在任何其他服务器端错误且注册失败时发生。 通常，会返回诸如此类的JSON响应，尽管不能保证在所有错误中都会出现这种情况：

  ```json
  {
      "ok": false,
      "requestId": "1234567890",
      "message": "error message"
  }
  ```

## 流程请求 {#process-request}

此 `process` 操作会根据请求中的说明提交一个作业，以将源资源转换为多个演绎版。 成功完成的通知（事件类型） `rendition_created`)或任何错误(事件类型 `rendition_failed`)发送到事件日志，必须使用进行检索 [/register](#register) 在生成任意数量之前执行一次 `/process` 请求。 格式不正确的请求立即失败，并显示400错误代码。

二进制文件是使用URL引用的，例如Amazon AWS S3预签名URL或Azure Blob Storage SAS URL，两者均读取 `source` 资源(`GET` URL)和写入演绎版(`PUT` URL)。 客户端负责生成这些预签名的URL。

| 参数 | 价值 |
|--------------------------|------------------------------------------------------|
| 方法 | `POST` |
| 路径 | `/process` |
| MIME类型 | `application/json` |
| 页眉 `Authorization` | 全部 [授权相关标头](#authentication-and-authorization). |
| 页眉 `x-request-id` | 可选，可由客户端跨系统设置处理请求的唯一端到端标识符。 |
| 请求正文 | 必须采用如下所述的流程请求JSON格式。 它提供有关要处理的资产以及要生成的演绎版的说明。 |

### 处理请求JSON {#process-request-json}

的请求正文 `/process` 是一个具有此高级模式的JSON对象：

```json
{
    "source": "",
    "renditions" : []
}
```

可用字段包括：

| 名称 | 类型 | 描述 | 示例 |
|--------------|----------|-------------|---------|
| `source` | `string` | 要处理的源资产的URL。 基于请求的演绎版格式的可选(例如， `fmt=zip`)。 | `"http://example.com/image.jpg"` |
| `source` | `object` | 描述要处理的源资产。 查看描述 [源对象字段](#source-object-fields) 下面的。 基于请求的演绎版格式的可选(例如， `fmt=zip`)。 | `{"url": "http://example.com/image.jpg", "mimeType": "image/jpeg" }` |
| `renditions` | `array` | 要从源文件生成的演绎版。 每个演绎版对象支持 [演绎版指令](#rendition-instructions). 必需. | `[{ "target": "https://....", "fmt": "png" }]` |

此 `source` 可以是 `<string>` 该URL可以是 `<object>` 带有一个附加字段。 以下变量类似：

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
| `url` | `string` | 要处理的源资产的URL。 必需. | `"http://example.com/image.jpg"` |
| `name` | `string` | 源资源文件名。 如果检测不到MIME类型，则可以使用名称中的文件扩展名。 优先于URL路径中的文件名或中的文件名 `content-disposition` 二进制资源的标头。 默认为“file”。 | `"image.jpg"` |
| `size` | `number` | 源资源文件大小（字节）。 优先于 `content-length` 二进制资源的标头。 | `10234` |
| `mimetype` | `string` | 源资源文件MIME类型。 优先于 `content-type` 二进制资源的标头。 | `"image/jpeg"` |

### 完整 `process` 请求示例 {#complete-process-request-example}

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

此 `/process` 根据基本请求验证，请求会立即返回成功或失败。 实际资产处理是异步进行的。

| 参数 | 价值 |
|-----------------------|------------------------------------------------------|
| MIME类型 | `application/json` |
| 页眉 `X-Request-Id` | 与 `X-Request-Id` 请求标头或唯一生成的标头。 用于标识跨系统的请求和/或支持请求。 |
| 响应正文 | 包含的JSON对象 `ok` 和 `requestId` 字段。 |

状态代码：

* **200项成功**：表示已成功提交请求。 响应JSON包含 `"ok": true`：

  ```json
  {
      "ok": true,
      "requestId": "1234567890"
  }
  ```

* **400无效的请求**：如果请求的格式不正确，例如请求JSON中缺少必填字段。 响应JSON包含 `"ok": false`：

  ```json
  {
      "ok": false,
      "requestId": "1234567890",
      "message": "error message"
  }
  ```

* **401未授权**：当请求无效时 [身份验证](#authentication-and-authorization). 例如，访问令牌或API密钥无效。
* **403禁止访问**：当请求无效时 [授权](#authentication-and-authorization). 例如，可能是一个有效的访问令牌，但Adobe Developer Console项目（技术帐户）未订阅所有必需的服务。
* **429请求太多**：当系统被此客户端重载或通常重载时。 客户端可以使用 [指数回退](https://en.wikipedia.org/wiki/Exponential_backoff). 正文为空。
* **4xx错误**：当出现任何其他客户端错误时。 通常，会返回诸如此类的JSON响应，尽管不能保证在所有错误中都会出现这种情况：

  ```json
  {
      "ok": false,
      "requestId": "1234567890",
      "message": "error message"
  }
  ```

* **5xx错误**：当出现任何其他服务器端错误时。 通常，会返回诸如此类的JSON响应，尽管不能保证在所有错误中都会出现这种情况：

  ```json
  {
      "ok": false,
      "requestId": "1234567890",
      "message": "error message"
  }
  ```

大多数客户可能倾向于使用重试完全相同的请求 [指数回退](https://en.wikipedia.org/wiki/Exponential_backoff) 发生任何错误时 *排除* 配置问题（如401或403）或无效请求（如400）。 除了通过429响应进行常规速率限制外，临时服务中断或限制可能会导致5xx错误。 然后，建议您在一段时间后重试。

所有JSON响应（如果存在）包括 `requestId` ，该值与 `X-Request-Id` 标头。 建议从标头读取，因为它始终存在。 此 `requestId` 也会在与处理请求相关的所有事件中作为 `requestId`. 客户端不得对此字符串的格式进行任何假设，它是不透明的字符串标识符。

## 选择加入后处理 {#opt-in-to-post-processing}

此 [ASSET COMPUTESDK](https://github.com/adobe/asset-compute-sdk) 支持一组基本的图像后处理选项。 自定义工作程序可以通过设置字段来明确选择启用后处理 `postProcess` 将节目对象设置为 `true`.

支持的用例包括：

* 将演绎版裁切为矩形，其限制由crop.w、crop.h、crop.x和crop.y定义。它由以下定义 `instructions.crop` 在节目对象中。
* 使用宽度、高度或两者来调整图像大小。 它由以下定义 `instructions.width` 和 `instructions.height` 在节目对象中。 要仅使用宽度或高度来调整大小，请仅设置一个值。 计算服务可节省宽高比。
* 设置JPEG图像的品质。 它由以下定义 `instructions.quality` 在节目对象中。 最佳品质表示为 `100` 较小的值表示质量降低。
* 创建交错图像。 它由以下定义 `instructions.interlace` 在节目对象中。
* 设置DPI可通过调整应用于像素的比例来调整渲染大小，以用于桌面出版目的。 它由以下定义 `instructions.dpi` 更改dpi分辨率。 但是，要调整图像大小以便以不同的分辨率显示相同的大小，请使用 `convertToDpi` 说明。
* 调整图像大小，使其渲染的宽度或高度在指定的目标分辨率(DPI)下与原始图像保持相同。 它由以下定义 `instructions.convertToDpi` 在节目对象中。

## 为资源添加水印 {#add-watermark}

此 [ASSET COMPUTESDK](https://github.com/adobe/asset-compute-sdk) 支持向PNG、JPEG、TIFF和GIF图像文件添加水印。 水印将按照演绎版中的说明添加 `watermark` 呈现版本上的对象。

水印在演绎版后处理期间完成。 要对资产添加水印，自定义工作程序 [选择进行后处理](#opt-in-to-post-processing) 通过设置字段 `postProcess` 将节目对象设置为 `true`. 如果辅助进程不选择加入，则不会应用水印，即使对请求中的演绎版对象设置了水印对象也是如此。

## 节目说明 {#rendition-instructions}

这些选项适用于 `renditions` 数组 [/process](#process-request).

### 常用字段 {#common-fields}

| 名称 | 类型 | 描述 | 示例 |
|-------------------|----------|-------------|---------|
| `fmt` | `string` | 演绎版目标格式，也可以 `text` 用于文本提取和 `xmp` 用于将XMP元数据提取为xml。 参见 [支持的格式](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/file-format-support.html) | `png` |
| `worker` | `string` | 的URL [自定义应用程序](develop-custom-application.md). 必须为 `https://` URL。 如果此字段存在，则呈现版本由自定义应用程序创建。 然后，在自定义应用程序中使用任何其他设置演绎版字段。 | `"https://1234.adobeioruntime.net`<br>`/api/v1/web`<br>`/example-custom-worker-master/worker"` |
| `target` | `string` | 应使用HTTPPUT将生成的演绎版上传到的URL。 | `http://w.com/img.jpg` |
| `target` | `object` | 生成的演绎版的多部分预签名URL上传信息。 这是用于 [AEM/Oak直接二进制上传](https://jackrabbit.apache.org/oak/docs/features/direct-binary-access.html) 使用此 [多部分上传行为](https://jackrabbit.apache.org/oak/docs/apidocs/org/apache/jackrabbit/api/binary/BinaryUpload.html).<br>字段:<ul><li>`urls`：字符串数组，每个预签名部分URL对应一个字符串</li><li>`minPartSize`：用于一个部分的最小大小= url</li><li>`maxPartSize`：用于一个部分的最大大小= url</li></ul> | `{ "urls": [ "https://part1...", "https://part2..." ], "minPartSize": 10000, "maxPartSize": 100000 }` |
| `userData` | `object` | 由客户端控制的可选保留空间，按原样传递给演绎版事件。 允许客户端添加自定义信息以标识演绎版事件。 在自定义应用程序中不得修改或依赖，因为客户端随时可以更改。 | `{ ... }` |

### 节目特定字段 {#rendition-specific-fields}

有关当前支持的文件格式的列表，请参阅 [支持的文件格式](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/file-format-support.html).

| 名称 | 类型 | 描述 | 示例 |
|-------------------|----------|-------------|---------|
| `*` | `*` | 可添加高级自定义字段，以便 [自定义应用程序](develop-custom-application.md) 理解。 | |
| `embedBinaryLimit` | `number` 以字节为单位 | 如果设置了此值，并且演绎版的文件大小小于此值，则演绎版将嵌入到演绎版生成完成后发送的事件中。 允许嵌入的最大大小为32 KB （32 x 1024字节）。 如果演绎版的大小大于 `embedBinaryLimit` 限制，则它被放置在云存储中的某个位置，并且不会嵌入事件中。 | `3276` |
| `width` | `number` | 以像素为单位的宽度. 仅适用于图像演绎版。 | `200` |
| `height` | `number` | 高度（以像素为单位）. 仅适用于图像演绎版。 | `200` |
|                   |          | 在以下情况下，始终保持宽高比： <ul> <li> 两者 `width` 和 `height` 指定，则在保持宽高比的情况下图像大小适合 </li><li> 仅 `width` 或仅限 `height` 指定，则生成的图像会使用相应的尺寸，同时保持纵横比</li><li> 如果两者都不 `width` 也不 `height` 指定，则使用原始图像像素大小。 它取决于源类型。 对于某些格式(如PDF文件)，将使用默认大小。 可以有最大大小限制。</li></ul> | |
| `quality` | `number` | 在以下范围中指定jpeg品质： `1` 到 `100`. 仅适用于图像演绎版。 | `90` |
| `xmp` | `string` | 仅由XMP元数据写回使用，它是base64编码的XMP，用于写回指定的演绎版。 | |
| `interlace` | `bool` | 通过将隔行PNG、GIF或渐进式JPEG设置为，可创建隔行PNG `true`. 它对其他文件格式没有影响。 | |
| `jpegSize` | `number` | JPEG文件的大致大小（字节）。 它会覆盖任何 `quality` 设置。 对其他格式没有影响。 | |
| `dpi` | `number` 或 `object` | 设置x和y DPI。 为简单起见，还可以将其设置为用于x和y的单个数字。它对图像本身没有影响。 | `96` 或 `{ xdpi: 96, ydpi: 96 }` |
| `convertToDpi` | `number` 或 `object` | x和y DPI重新采样值，同时保持物理大小。 为简单起见，还可以将其设置为用于x和y的单个数字。 | `96` 或 `{ xdpi: 96, ydpi: 96 }` |
| `files` | `array` | 要包含在ZIP存档中的文件列表(`fmt=zip`)。 每个条目可以是URL字符串或包含字段的对象：<ul><li>`url`：下载文件的URL</li><li>`path`：将文件存储在此路径下的ZIP文件中</li></ul> | `[{ "url": "https://host/asset.jpg", "path": "folder/location/asset.jpg" }]` |
| `duplicate` | `string` | ZIP存档的重复处理(`fmt=zip`)。 默认情况下，存储在ZIP中同一路径下的多个文件会生成错误。 设置 `duplicate` 到 `ignore` 导致仅存储第一个资产，而忽略其余资产。 | `ignore` |
| `watermark` | `object` | 包含关于 [水印](#watermark-specific-fields). |  |

### 特定于水印的字段 {#watermark-specific-fields}

PNG格式用作水印。

| 名称 | 类型 | 描述 | 示例 |
|-------------------|----------|-------------|---------|
| `scale` | `number` | 水印的比例，介于 `0.0` 和 `1.0`. `1.0` 表示水印具有其原始比例(1:1)，较低的值会减小水印大小。 | 值 `0.5` 表示原始大小的一半。 |
| `image` | `url` | 用于水印的PNG文件的URL。 | |

## 异步事件 {#asynchronous-events}

处理完节目或发生错误后，会将事件发送到 [[!DNL Adobe I/O] 事件日志](https://www.adobe.io/apis/experienceplatform/events/documentation.html#!adobedocs/adobeio-events/master/intro/journaling_api.md). 客户端必须侦听通过提供的日志URL [/register](#register). 日志响应包括 `event` 数组，每个事件由一个对象组成，其中 `event` 字段包含实际事件有效负载。

此 [!DNL Adobe I/O] 的所有事件的事件类型 [!DNL Asset Compute Service] 是 `asset_compute`. 日志仅自动订阅此事件类型，无需根据 [!DNL Adobe I/O] 事件类型。 特定于服务的事件类型请参见 `type` 事件的属性。

### 事件类型 {#event-types}

| Event | 描述 |
|---------------------|-------------|
| `rendition_created` | 已针对每个成功处理和上传的呈现版本发送。 |
| `rendition_failed` | 为处理或上传失败的每个演绎版发送。 |

### 事件属性 {#event-attributes}

| 属性 | 类型 | Event | 描述 |
|-------------|----------|---------------|-------------|
| `date` | `string` | `*` | 以简化的扩展方式发送事件时的时间戳 [ISO-8601](https://en.wikipedia.org/wiki/ISO_8601) 格式，由JavaScript定义 [Date.toISOString()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date/toISOString). |
| `requestId` | `string` | `*` | 对的原始请求的请求ID `/process`，与 `X-Request-Id` 标头。 |
| `source` | `object` | `*` | 此 `source` 的 `/process` 请求。 |
| `userData` | `object` | `*` | 此 `userData` 的格式副本 `/process` 请求（如果已设置）。 |
| `rendition` | `object` | `rendition_*` | 传入的相应演绎版对象 `/process`. |
| `metadata` | `object` | `rendition_created` | 此 [元数据](#metadata) 演绎版的属性。 |
| `errorReason` | `string` | `rendition_failed` | 节目失败 [原因](#error-reasons) 如果有。 |
| `errorMessage` | `string` | `rendition_failed` | 提供有关演绎版失败的更多详细信息（如果有）的文本。 |

### 元数据 {#metadata}

| 属性 | 描述 |
|--------|-------------|
| `repo:size` | 演绎版的大小，以字节为单位。 |
| `repo:sha1` | 演绎版的sha1摘要。 |
| `dc:format` | 演绎版的 MIME 类型。 |
| `repo:encoding` | 演绎版的charset编码（如果它是基于文本的格式）。 |
| `tiff:ImageWidth` | 演绎版的宽度（像素）。 仅对于图像演绎版存在。 |
| `tiff:ImageLength` | 演绎版的长度（以像素为单位）。 仅对于图像演绎版存在。 |

### 错误原因 {#error-reasons}

| 原因 | 描述 |
|---------|-------------|
| `RenditionFormatUnsupported` | 给定的源不支持所请求的演绎版格式。 |
| `SourceUnsupported` | 特定源不受支持，即使该类型受支持也是如此。 |
| `SourceCorrupt` | 源数据已损坏。 包含空文件。 |
| `RenditionTooLarge` | 无法使用中提供的预签名URL上传节目 `target`. 实际演绎版大小可用作中的元数据 `repo:size` 和可供客户端使用重新处理此演绎版，并具有正确数量的预签名URL。 |
| `GenericError` | 任何其他意外错误。 |
