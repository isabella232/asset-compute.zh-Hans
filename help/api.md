---
title: "[!DNL Asset Compute Service] HTTP API"
description: '"[!DNL Asset Compute Service] HTTP API可建立自訂應用程式。」'
exl-id: 4b63fdf9-9c0d-4af7-839d-a95e07509750
source-git-commit: 93d3b407c8875888f03bec673d0a677a3205cfbb
workflow-type: tm+mt
source-wordcount: '2906'
ht-degree: 3%

---

# [!DNL Asset Compute Service] HTTP API {#asset-compute-http-api}

API的使用僅限於開發目的。 此API在開發自訂應用程式時作為上下文提供。 [!DNL Adobe Experience Manager] as a [!DNL Cloud Service] 使用API將處理資訊傳遞至自訂應用程式。 如需詳細資訊，請參閱 [使用資產微服務和處理設定檔](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html).

>[!NOTE]
>
>[!DNL Asset Compute Service] 僅供搭配使用 [!DNL Experience Manager] as a [!DNL Cloud Service].

的任何使用者端 [!DNL Asset Compute Service] HTTP API必須遵循此高階流程：

1. 使用者端布建為 [!DNL Adobe Developer Console] ims組織中的專案。 每個單獨的使用者端（系統或環境）需要其自己的單獨專案，以分離事件資料流。

1. 使用者端使用為技術帳戶產生存取權杖 [JWT （服務帳戶）驗證](https://www.adobe.io/authentication/auth-methods.html).

1. 使用者端呼叫 [`/register`](#register) 僅擷取日誌URL一次。

1. 使用者端呼叫 [`/process`](#process-request) 針對每個要產生轉譯的資產。 呼叫為非同步。

1. 使用者端定期輪詢日誌 [接收事件](#asynchronous-events). 成功處理轉譯時，它會接收每個請求轉譯的事件(`rendition_created` 事件型別)或是否有錯誤(`rendition_failed` 事件型別)。

此 [@adobe/asset-compute-client](https://github.com/adobe/asset-compute-client) 模組可讓您在Node.js程式碼中輕鬆使用API。

## 驗證和授權 {#authentication-and-authorization}

所有API都需要存取權杖驗證。 請求必須設定以下標頭：

1. `Authorization` 標頭具有持有人權杖，這是技術帳戶權杖，透過接收 [JWT交換](https://www.adobe.io/authentication/auth-methods.html) 從Adobe Developer Console專案。 此 [範圍](#scopes) 記錄如下。

<!-- TBD: Change the existing URL to a new path when a new path for docs is available. The current path contains master word that is not an inclusive term. Logged ticket in Adobe I/O's GitHub repo to get a new URL.
-->

1. `x-gw-ims-org-id` IMS組織ID的標頭。

1. `x-api-key` ，使用者端ID來自 [!DNL Adobe Developers Console] 專案。

### 范围 {#scopes}

確定以下存取權杖範圍：

* `openid`
* `AdobeID`
* `asset_compute`
* `read_organizations`
* `event_receiver`
* `event_receiver_api`
* `adobeio_api`
* `additional_info.roles`
* `additional_info.projectedProductContext`

這些需要 [!DNL Adobe Developer Console] 要訂閱的專案 `Asset Compute`， `I/O Events`、和 `I/O Management API` 服務。 個別範圍的劃分如下：

* 基本
   * 範圍： `openid,AdobeID`

* asset compute
   * metascope： `asset_compute_meta`
   * 範圍： `asset_compute,read_organizations`

* [!DNL Adobe I/O] 事件
   * metascope： `event_receiver_api`
   * 範圍： `event_receiver,event_receiver_api`

* [!DNL Adobe I/O] 管理API
   * metascope： `ent_adobeio_sdk`
   * 範圍： `adobeio_api,additional_info.roles,additional_info.projectedProductContext`

## 注册 {#register}

每個使用者端 [!DNL Asset Compute service]  — 不重複 [!DNL Adobe Developer Console] 訂閱服務的專案 — 必須 [註冊](#register-request) 進行處理要求之前。 註冊步驟會傳回從轉譯處理中擷取非同步事件所需的唯一事件日誌。

在其生命週期結束時，使用者端可以 [取消註冊](#unregister-request).

### 註冊要求 {#register-request}

此API呼叫會設定 [!DNL Asset Compute] 使用者端並提供事件日誌URL。 這是等冪運算，每個使用者端只需要呼叫一次。 可以再次呼叫它以擷取日誌URL。

| 参数 | 价值 |
|--------------------------|------------------------------------------------------|
| 方法 | `POST` |
| 路径 | `/register` |
| 页眉 `Authorization` | 全部 [授權相關標頭](#authentication-and-authorization). |
| 页眉 `x-request-id` | 可選，可由使用者端設定為跨系統處理要求的唯一端對端識別碼。 |
| 要求內文 | 必須為空白。 |

### 註冊回應 {#register-response}

| 参数 | 价值 |
|-----------------------|------------------------------------------------------|
| MIME型別 | `application/json` |
| 页眉 `X-Request-Id` | 與 `X-Request-Id` 請求標頭或唯一產生的標頭。 用於識別跨系統的請求和/或支援請求。 |
| 回應內文 | 具有的JSON物件 `journal`， `ok` 和/或 `requestId` 欄位。 |

HTTP狀態碼為：

* **200項成功**：要求成功時。 它包含 `journal` 透過觸發的非同步處理任何結果通知的URL `/process` (作為事件型別 `rendition_created` 成功時，或 `rendition_failed` 失敗時)。

   ```json
   {
       "ok": true,
       "journal": "https://api.adobe.io/events/organizations/xxxxx/integrations/xxxx/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
       "requestId": "1234567890"
   }
   ```

* **401未獲授權**：當請求無效時發生 [驗證](#authentication-and-authorization). 例如，存取權杖無效或API金鑰無效。

* **403禁止**：當請求無效時發生 [授權](#authentication-and-authorization). 一個範例可能是有效的存取Token，但Adobe Developer Console專案（技術帳戶）未訂閱所有必要的服務。

* **429請求太多**：當系統被此使用者端或其他方式多載時發生。 使用者端應使用 [指數輪詢](https://en.wikipedia.org/wiki/Exponential_backoff). 內文是空的。
* **4xx錯誤**：發生任何其他使用者端錯誤且註冊失敗時。 通常會傳回類似的JSON回應，但並非所有錯誤都有保證：

   ```json
   {
       "ok": false,
       "requestId": "1234567890",
       "message": "error message"
   }
   ```

* **5xx錯誤**：發生於發生任何其他伺服器端錯誤且註冊失敗時。 通常會傳回類似的JSON回應，但並非所有錯誤都有保證：

   ```json
   {
       "ok": false,
       "requestId": "1234567890",
       "message": "error message"
   }
   ```

### 取消註冊請求 {#unregister-request}

此API呼叫會取消註冊 [!DNL Asset Compute] 使用者端。 之後就不能再呼叫 `/process`. 對未註冊的使用者端或尚未註冊的使用者端使用API呼叫會傳回 `404` 錯誤。

| 参数 | 价值 |
|--------------------------|------------------------------------------------------|
| 方法 | `POST` |
| 路径 | `/unregister` |
| 页眉 `Authorization` | 全部 [授權相關標頭](#authentication-and-authorization). |
| 页眉 `x-request-id` | 可選，可由使用者端設定為跨系統處理要求的唯一端對端識別碼。 |
| 要求內文 | 空. |

### 取消登入回應 {#unregister-response}

| 参数 | 价值 |
|-----------------------|------------------------------------------------------|
| MIME型別 | `application/json` |
| 页眉 `X-Request-Id` | 與 `X-Request-Id` 請求標頭或唯一產生的標頭。 用於識別跨系統的請求和/或支援請求。 |
| 回應內文 | 具有的JSON物件 `ok` 和 `requestId` 欄位。 |

狀態代碼為：

* **200項成功**：找到並移除註冊和日誌時發生。

   ```json
   {
       "ok": true,
       "requestId": "1234567890"
   }
   ```

* **401未獲授權**：當請求無效時發生 [驗證](#authentication-and-authorization). 例如，存取權杖無效或API金鑰無效。

* **403禁止**：當請求無效時發生 [授權](#authentication-and-authorization). 一個範例可能是有效的存取Token，但Adobe Developer Console專案（技術帳戶）未訂閱所有必要的服務。

* **404找不到**：當指定認證目前沒有註冊時發生。

   ```json
   {
       "ok": true,
       "requestId": "1234567890"
   }
   ```

* **429請求太多**：在系統超載時發生。 使用者端應使用 [指數輪詢](https://en.wikipedia.org/wiki/Exponential_backoff). 內文是空的。

* **4xx錯誤**：當發生任何其他使用者端錯誤且取消註冊失敗時發生。 通常會傳回類似的JSON回應，但並非所有錯誤都有保證：

   ```json
   {
       "ok": false,
       "requestId": "1234567890",
       "message": "error message"
   }
   ```

* **5xx錯誤**：發生於發生任何其他伺服器端錯誤且註冊失敗時。 通常會傳回類似的JSON回應，但並非所有錯誤都有保證：

   ```json
   {
       "ok": false,
       "requestId": "1234567890",
       "message": "error message"
   }
   ```

## 處理要求 {#process-request}

此 `process` 作業會根據請求中的指示，提交將來源資產轉換為多個轉譯的工作。 成功完成的通知（事件型別） `rendition_created`)或任何錯誤(事件型別 `rendition_failed`)傳送至事件日誌，而事件日誌必須透過以下方式擷取： [/register](#register) 在生成任何數量之前執行一次 `/process` 要求。 格式錯誤的請求會立即失敗，並產生400錯誤代碼。

二進位檔是使用URL來參照，例如Amazon AWS S3預先簽署的URL或Azure Blob儲存SAS URL，兩者都會讀取 `source` 資產(`GET` URL)和撰寫轉譯(`PUT` URL)。 使用者端負責產生這些預先簽署的URL。

| 参数 | 价值 |
|--------------------------|------------------------------------------------------|
| 方法 | `POST` |
| 路径 | `/process` |
| MIME型別 | `application/json` |
| 页眉 `Authorization` | 全部 [授權相關標頭](#authentication-and-authorization). |
| 页眉 `x-request-id` | 可選，可由使用者端設定為跨系統處理要求的唯一端對端識別碼。 |
| 要求內文 | 必須採用如下所述的程式請求JSON格式。 它會提供要處理的資產以及要產生哪些轉譯的相關指示。 |

### 處理請求JSON {#process-request-json}

的請求內文 `/process` 是具備此高階結構描述的JSON物件：

```json
{
    "source": "",
    "renditions" : []
}
```

可用的欄位包括：

| 名称 | 类型 | 描述 | 示例 |
|--------------|----------|-------------|---------|
| `source` | `string` | 要處理的來源資產的URL。 根據要求的轉譯格式(例如 `fmt=zip`)。 | `"http://example.com/image.jpg"` |
| `source` | `object` | 說明要處理的來源資產。 請參閱「 」的說明 [來源物件欄位](#source-object-fields) 下方的。 根據要求的轉譯格式(例如 `fmt=zip`)。 | `{"url": "http://example.com/image.jpg", "mimeType": "image/jpeg" }` |
| `renditions` | `array` | 從來源檔案產生的轉譯。 每個轉譯物件都支援 [轉譯指示](#rendition-instructions). 必填. | `[{ "target": "https://....", "fmt": "png" }]` |

此 `source` 可以是 `<string>` 視為URL或可以是 `<object>` 含一個額外欄位。 下列變體類似：

```json
"source": "http://example.com/image.jpg"
```

```json
"source": {
    "url": "http://example.com/image.jpg"
}
```

### 來源物件欄位 {#source-object-fields}

| 名称 | 类型 | 描述 | 示例 |
|-----------|----------|-------------|---------|
| `url` | `string` | 要處理的來源資產的URL。 必填. | `"http://example.com/image.jpg"` |
| `name` | `string` | 來源資產檔案名稱。 如果偵測不到MIME型別，則可能會使用名稱中的副檔名。 優先於URL路徑中的檔案名稱或中的檔案名稱 `content-disposition` 二進位資源的標頭。 預設為「file」。 | `"image.jpg"` |
| `size` | `number` | 來源資產檔案大小（位元組）。 優先於 `content-length` 二進位資源的標頭。 | `10234` |
| `mimetype` | `string` | 來源資產檔案MIME型別。 優先於 `content-type` 二進位資源的標頭。 | `"image/jpeg"` |

### 完整 `process` 請求範例 {#complete-process-request-example}

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

## 處理序回應 {#process-response}

此 `/process` 根據基本請求驗證，請求會立即傳回成功或失敗。 實際資產處理作業會非同步進行。

| 参数 | 价值 |
|-----------------------|------------------------------------------------------|
| MIME型別 | `application/json` |
| 页眉 `X-Request-Id` | 與 `X-Request-Id` 請求標頭或唯一產生的標頭。 用於識別跨系統的請求和/或支援請求。 |
| 回應內文 | 具有的JSON物件 `ok` 和 `requestId` 欄位。 |

狀態代碼：

* **200項成功**：若已成功提交請求。 回應JSON包含 `"ok": true`：

   ```json
   {
       "ok": true,
       "requestId": "1234567890"
   }
   ```

* **400無效的請求**：如果請求的格式不正確，例如請求JSON中缺少必填欄位。 回應JSON包含 `"ok": false`：

   ```json
   {
       "ok": false,
       "requestId": "1234567890",
       "message": "error message"
   }
   ```

* **401未獲授權**：當請求無效時 [驗證](#authentication-and-authorization). 例如，存取權杖無效或API金鑰無效。
* **403禁止**：當請求無效時 [授權](#authentication-and-authorization). 一個範例可能是有效的存取Token，但Adobe Developer Console專案（技術帳戶）未訂閱所有必要的服務。
* **429請求太多**：此使用者端或一般而言系統超載時。 使用者端可使用 [指數輪詢](https://en.wikipedia.org/wiki/Exponential_backoff). 內文是空的。
* **4xx錯誤**：發生任何其他使用者端錯誤時。 通常會傳回類似的JSON回應，但並非所有錯誤都有保證：

   ```json
   {
       "ok": false,
       "requestId": "1234567890",
       "message": "error message"
   }
   ```

* **5xx錯誤**：發生任何其他伺服器端錯誤時。 通常會傳回類似的JSON回應，但並非所有錯誤都有保證：

   ```json
   {
       "ok": false,
       "requestId": "1234567890",
       "message": "error message"
   }
   ```

大多數使用者端都可能傾向使用來重試完全相同的請求 [指數輪詢](https://en.wikipedia.org/wiki/Exponential_backoff) 發生任何錯誤時 *例外* 設定問題（例如401或403）或無效請求（例如400）。 除了透過429回應來限制一般速率之外，暫時性的服務中斷或限制可能會導致5xx錯誤。 之後，建議您在一段時間後重試。

所有JSON回應（如果存在）包含 `requestId` ，此值與 `X-Request-Id` 標頭。 建議從標頭讀取，因為它永遠存在。 此 `requestId` 也會在與處理請求相關的所有事件中傳回 `requestId`. 使用者端不得對此字串的格式做任何假設，此字串是不透明的字串識別碼。

## 選擇加入後續處理 {#opt-in-to-post-processing}

此 [ASSET COMPUTESDK](https://github.com/adobe/asset-compute-sdk) 支援一組基本影像後處理選項。 自訂背景工作程式可以透過設定欄位明確選擇加入後續處理 `postProcess` 在轉譯物件上 `true`.

支援的使用案例包括：

* 將轉譯裁切為矩形，其限制由crop.w、crop.h、crop.x和crop.y定義。它由以下定義： `instructions.crop` 在轉譯物件中。
* 使用寬度、高度或兩者來調整影像大小。 它由以下定義： `instructions.width` 和 `instructions.height` 在轉譯物件中。 若要僅使用寬度或高度來調整大小，請僅設定一個值。 運算服務可節省外觀比例。
* 設定JPEG影像的品質。 它由以下定義： `instructions.quality` 在轉譯物件中。 最佳品質表示為 `100` 和較小的值表示品質降低。
* 建立交錯影像。 它由以下定義： `instructions.interlace` 在轉譯物件中。
* 設定DPI可調整套用至畫素的比例，以調整案頭出版所需的演算大小。 它由以下定義： `instructions.dpi` 變更dpi解析度。 不過，若要調整影像大小，使其以不同解析度呈現相同大小，請使用 `convertToDpi` 指示。
* 調整影像大小，使其演算後的寬度或高度與指定目標解析度(DPI)的原始影像保持相同。 它由以下定義： `instructions.convertToDpi` 在轉譯物件中。

## 为资源添加水印 {#add-watermark}

此 [ASSET COMPUTESDK](https://github.com/adobe/asset-compute-sdk) 支援在PNG、JPEG、TIFF和GIF影像檔案中新增浮水印。 浮水印是依照中的轉譯指示新增的 `watermark` 物件。

浮水印會在轉譯後處理期間完成。 若要為資產加上浮水印，自訂背景工作 [選擇加入後續處理](#opt-in-to-post-processing) 藉由設定欄位 `postProcess` 在轉譯物件上 `true`. 如果Worker未選擇加入，則不會套用浮水印，即使浮水印物件已設定在請求中的轉譯物件上。

## 轉譯指示 {#rendition-instructions}

以下是「 」的可用選項 `renditions` 中的陣列 [/process](#process-request).

### 常見欄位 {#common-fields}

| 名称 | 类型 | 描述 | 示例 |
|-------------------|----------|-------------|---------|
| `fmt` | `string` | 轉譯目標格式，也可以 `text` 文字擷取和 `xmp` 用於將XMP中繼資料擷取為xml。 另請參閱 [支援的格式](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/file-format-support.html) | `png` |
| `worker` | `string` | 的URL [自訂應用程式](develop-custom-application.md). 必須是 `https://` URL。 如果此欄位存在，轉譯會由自訂應用程式建立。 然後，任何其他集合轉譯欄位都會用於自訂應用程式中。 | `"https://1234.adobeioruntime.net`<br>`/api/v1/web`<br>`/example-custom-worker-master/worker"` |
| `target` | `string` | 應使用HTTPPUT將產生的轉譯上傳到的URL。 | `http://w.com/img.jpg` |
| `target` | `object` | 所產生轉譯的多部分預先簽署URL上傳資訊。 這是用於 [AEM/Oak直接二進位上傳](https://jackrabbit.apache.org/oak/docs/features/direct-binary-access.html) 與此 [多部分上傳行為](https://jackrabbit.apache.org/oak/docs/apidocs/org/apache/jackrabbit/api/binary/BinaryUpload.html).<br>字段:<ul><li>`urls`：字串陣列，每個預先簽署部分URL各一個</li><li>`minPartSize`：用於一個部分的最小大小= url</li><li>`maxPartSize`：用於一個部分的大小上限= url</li></ul> | `{ "urls": [ "https://part1...", "https://part2..." ], "minPartSize": 10000, "maxPartSize": 100000 }` |
| `userData` | `object` | 使用者端控制的可選保留空間，依原樣傳遞至轉譯事件。 允許使用者端新增自訂資訊以識別轉譯事件。 自訂應用程式不可修改或依賴，因為使用者端隨時可以變更。 | `{ ... }` |

### 轉譯特定欄位 {#rendition-specific-fields}

如需目前支援的檔案格式清單，請參閱 [支援的檔案格式](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/file-format-support.html).

| 名称 | 类型 | 描述 | 示例 |
|-------------------|----------|-------------|---------|
| `*` | `*` | 進階、自訂欄位可新增 [自訂應用程式](develop-custom-application.md) 瞭解。 |  |
| `embedBinaryLimit` | `number` 位元組 | 如果設定了此值，且轉譯的檔案大小小於此值，則轉譯會內嵌在當轉譯產生完成時所傳送的事件中。 允許內嵌的大小上限為32 KB （32 x 1024位元組）。 如果轉譯的大小大於 `embedBinaryLimit` limit，會放置在雲端儲存空間中的某個位置，而不會內嵌在事件中。 | `3276` |
| `width` | `number` | 以像素为单位的宽度. 僅適用於影像轉譯。 | `200` |
| `height` | `number` | 高度（以像素为单位）. 僅適用於影像轉譯。 | `200` |
|  |  | 如果符合下列條件，則一律會維持外觀比例： <ul> <li> 兩者 `width` 和 `height` 會指定，然後影像會符合大小，同時維持外觀比例 </li><li> 僅限 `width` 或僅限 `height` 指定，產生的影像會使用對應的尺寸，同時保持外觀比例</li><li> 如果兩者都不 `width` 也不 `height` 指定，則會使用原始影像畫素大小。 這取決於來源型別。 對於某些格式(例如PDF檔案)，會使用預設大小。 可以有大小上限。</li></ul> |  |
| `quality` | `number` | 在以下範圍中指定jpeg品質： `1` 至 `100`. 僅適用於影像轉譯。 | `90` |
| `xmp` | `string` | 只有XMP中繼資料回寫可使用此功能，它是base64編碼的XMP，用於回寫指定的轉譯。 |  |
| `interlace` | `bool` | 建立交錯式PNG或GIF或漸進式JPEG，方法是將其設定為 `true`. 它對其他檔案格式沒有影響。 |  |
| `jpegSize` | `number` | JPEG檔案的大致大小（位元組）。 它會覆寫任何 `quality` 設定。 對其他格式沒有影響。 |  |
| `dpi` | `number` 或 `object` | 設定x和y DPI。 為簡單起見，它也可以設定為用於x和y的單一數字。這對影像本身沒有影響。 | `96` 或 `{ xdpi: 96, ydpi: 96 }` |
| `convertToDpi` | `number` 或 `object` | x和y DPI會重新取樣值，同時維持實體大小。 為簡單起見，它也可以設定為用於x和y的單一數字。 | `96` 或 `{ xdpi: 96, ydpi: 96 }` |
| `files` | `array` | 要包含在ZIP封存檔中的檔案清單(`fmt=zip`)。 每個專案可以是URL字串或具有欄位的物件：<ul><li>`url`：下載檔案的URL</li><li>`path`：將此路徑下的檔案儲存在ZIP中</li></ul> | `[{ "url": "https://host/asset.jpg", "path": "folder/location/asset.jpg" }]` |
| `duplicate` | `string` | ZIP封存檔的重複處理(`fmt=zip`)。 根據預設，儲存在ZIP中相同路徑下的多個檔案會產生錯誤。 設定 `duplicate` 至 `ignore` 只會儲存第一個資產，而忽略其餘資產。 | `ignore` |
| `watermark` | `object` | 包含關於 [浮水印](#watermark-specific-fields). |  |

### 浮水印特定欄位 {#watermark-specific-fields}

PNG格式會用作浮水印。

| 名称 | 类型 | 描述 | 示例 |
|-------------------|----------|-------------|---------|
| `scale` | `number` | 浮水印的比例，介於 `0.0` 和 `1.0`. `1.0` 表示浮水印具有原始比例(1:1)，較低的值會縮小浮水印大小。 | 值 `0.5` 代表原始大小的一半。 |
| `image` | `url` | 用於浮水印的PNG檔案的URL。 |  |

## 非同步事件 {#asynchronous-events}

轉譯的處理完成或發生錯誤時，事件會傳送至 [[!DNL Adobe I/O] 事件日誌](https://www.adobe.io/apis/experienceplatform/events/documentation.html#!adobedocs/adobeio-events/master/intro/journaling_api.md). 使用者端必須監聽透過提供的日誌URL [/register](#register). 日誌回應包括 `event` 陣列，每個事件由一個物件組成，其中 `event` 欄位包含實際事件裝載。

此 [!DNL Adobe I/O] 「 」的所有事件的事件型別 [!DNL Asset Compute Service] 是 `asset_compute`. 分錄只會自動訂閱此事件型別，不需要根據 [!DNL Adobe I/O] 事件型別。 此服務專屬的事件型別位於 `type` 事件的屬性。

### 事件型別 {#event-types}

| Event | 描述 |
|---------------------|-------------|
| `rendition_created` | 已針對每個成功處理和上傳的轉譯傳送。 |
| `rendition_failed` | 針對無法處理或上傳的每個轉譯傳送。 |

### 事件屬性 {#event-attributes}

| 属性 | 类型 | Event | 描述 |
|-------------|----------|---------------|-------------|
| `date` | `string` | `*` | 以簡化延伸傳送事件時的時間戳記 [ISO-8601](https://en.wikipedia.org/wiki/ISO_8601) 格式，由JavaScript定義 [Date.toISOString()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date/toISOString). |
| `requestId` | `string` | `*` | 原始請求的請求ID `/process`，與 `X-Request-Id` 標頭。 |
| `source` | `object` | `*` | 此 `source` 的 `/process` 要求。 |
| `userData` | `object` | `*` | 此 `userData` 的轉譯 `/process` 要求（若已設定）。 |
| `rendition` | `object` | `rendition_*` | 傳入的對應轉譯物件 `/process`. |
| `metadata` | `object` | `rendition_created` | 此 [中繼資料](#metadata) 轉譯的屬性。 |
| `errorReason` | `string` | `rendition_failed` | 轉譯失敗 [原因](#error-reasons) 如果有。 |
| `errorMessage` | `string` | `rendition_failed` | 提供轉譯失敗詳細資訊的文字（如果有的話）。 |

### 元数据 {#metadata}

| 属性 | 描述 |
|--------|-------------|
| `repo:size` | 轉譯的大小（位元組）。 |
| `repo:sha1` | 轉譯的sha1摘要。 |
| `dc:format` | 轉譯的MIME型別。 |
| `repo:encoding` | 轉譯的charset編碼（若是文字型格式）。 |
| `tiff:ImageWidth` | 轉譯的寬度（畫素）。 僅供影像轉譯使用。 |
| `tiff:ImageLength` | 轉譯的長度（畫素）。 僅供影像轉譯使用。 |

### 錯誤原因 {#error-reasons}

| 原因 | 描述 |
|---------|-------------|
| `RenditionFormatUnsupported` | 給定的來源不支援要求的轉譯格式。 |
| `SourceUnsupported` | 即使支援型別，也不支援特定來源。 |
| `SourceCorrupt` | 來源資料已損毀。 包含空白檔案。 |
| `RenditionTooLarge` | 無法使用中提供的預先簽署URL上傳轉譯 `target`. 實際轉譯大小在中會以中繼資料的形式提供 `repo:size` 使用者端可使用和以正確數量的預先簽署URL重新處理此轉譯。 |
| `GenericError` | 任何其他非預期的錯誤。 |
