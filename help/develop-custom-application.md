---
title: 為以下專案開發 [!DNL Asset Compute Service]
description: 建立自訂應用程式，使用 [!DNL Asset Compute Service].
exl-id: a0c59752-564b-4bb6-9833-ab7c58a7f38e
source-git-commit: 2dde177933477dc9ac2ff5a55af1fd2366e18359
workflow-type: tm+mt
source-wordcount: '1618'
ht-degree: 0%

---

# 開發自訂應用程式 {#develop}

開始開發自訂應用程式之前：

* 確定所有 [必備條件](/help/understand-extensibility.md#prerequisites-and-provisioning) 符合。
* 安裝 [必要的軟體工具](/help/setup-environment.md#create-dev-environment).
* 另請參閱 [設定您的環境](setup-environment.md) 以確保您已準備好建立自訂應用程式。

## 建立自訂應用程式 {#create-custom-application}

請務必擁有 [[!DNL Adobe I/O] CLI](https://github.com/adobe/aio-cli) 本機安裝。

1. 若要建立自訂應用程式， [建立App Builder專案](https://developer.adobe.com/app-builder/docs/getting_started/first_app/#4-bootstrapping-new-app-using-the-cli). 若要這麼做，請執行 `aio app init <app-name>` 在您的終端機中。

   如果您尚未登入，這個命令會提示您登入 [Adobe Developer主控台](https://console.adobe.io/) 使用您的Adobe ID。 另請參閱 [此處](https://developer.adobe.com/app-builder/docs/getting_started/first_app/#3-signing-in-from-cli) 以取得從cli登入的詳細資訊。

   Adobe建議您登入。 如果您遇到問題，請依照指示操作 [建立應用程式而不登入](https://developer.adobe.com/app-builder/docs/getting_started/first_app/#42-developer-is-not-logged-in-as-enterprise-organization-user).

1. 登入後，按照CLI中的提示操作，並選取 `Organization`， `Project`、和 `Workspace` 用於應用程式。 選擇您建立時建立的專案和工作區 [設定您的環境](setup-environment.md). 出現提示時 `Which extension point(s) do you wish to implement ?`，請務必選取 `DX Asset Compute Worker`：

   ```sh
   $ aio app init <app-name>
   Retrieving information from [!DNL Adobe I/O] Console.
   ? Select Org My Adobe Org
   ? Select Project MyAdobe Developer App BuilderProject
   ? Which extension point(s) do you wish to implement ? (Press <space> to select, <a>
   to toggle all, <i> to invert selection)
   ❯◯ DX Experience Cloud SPA
   ◯ DX Asset Compute Worker
   ```

1. 出現以下提示時： `Which Adobe I/O App features do you want to enable for this project?`，選取 `Actions`. 請務必取消選取 `Web Assets` 選項做為網頁資產使用不同的驗證和授權檢查。

   ```bash
   ? Which Adobe I/O App features do you want to enable for this project?
   select components to include (Press <space> to select, <a> to toggle all, <i> to invert selection)
   ❯◉ Actions: Deploy Runtime actions
   ◯ Events: Publish to Adobe I/O Events
   ◯ Web Assets: Deploy hosted static assets
   ◯ CI/CD: Include GitHub Actions based workflows for Build, Test and Deploy
   ```

1. 出現提示時 `Which type of sample actions do you want to create?`，請務必選取 `Adobe Asset Compute Worker`：

   ```bash
   ? Which type of sample actions do you want to create?
   Select type of actions to generate
   ❯◉ Adobe Asset Compute Worker
   ◯ Generic
   ```

1. 依照其餘的提示操作，並在Visual Studio Code （或您喜愛的程式碼編輯器）中開啟新的應用程式。 它包含自訂應用程式的支架和範常式式碼。

   請在此處閱讀 [App Builder應用程式的主要元件](https://developer.adobe.com/app-builder/docs/getting_started/first_app/#5-anatomy-of-an-app-builder-application).

   範本應用程式會利用 [ASSET COMPUTESDK](https://github.com/adobe/asset-compute-sdk#asset-compute-sdk) 適用於應用程式轉譯的上傳、下載和協調，因此開發人員只需要實作自訂應用程式邏輯。 內部 `actions/<worker-name>` 資料夾， `index.js` 檔案是新增自訂應用程式程式碼的位置。

另請參閱 [自訂應用程式範例](#try-sample) 自訂應用程式的範例和想法。

### 新增認證 {#add-credentials}

當您在建立應用程式時登入時，系統會在ENV檔案中收集大部份App Builder認證。 不過，使用開發人員工具需要其他憑證。

<!-- TBD: Check if manual setup of credentials is required.
Manual set up of credentials is removed from troubleshooting and best practices page. Link was broken.
If you did not log in, refer to our troubleshooting guide to [set up credentials manually](troubleshooting.md).
-->

#### 開發人員工具儲存憑證 {#developer-tool-credentials}

用來測試自訂應用程式的開發人員工具，包含 [!DNL Asset Compute service] 需要雲端儲存容器來裝載測試檔案，以及接收和顯示應用程式產生的轉譯。

>[!NOTE]
>
>這與的雲端儲存區不同 [!DNL Adobe Experience Manager] as a [!DNL Cloud Service]. 它僅適用於使用Asset compute開發人員工具進行開發和測試。

確定具有 [支援的雲端儲存體容器](https://github.com/adobe/asset-compute-devtool#prerequisites). 如有需要，不同專案中的多位開發人員可共用此容器。

#### 新增認證至ENV檔案 {#add-credentials-env-file}

將開發人員工具的下列認證新增至App Builder專案根目錄中的ENV檔案：

1. 將絕對路徑新增至在App Builder專案中新增服務時所建立的私密金鑰檔案：

   ```conf
   ASSET_COMPUTE_PRIVATE_KEY_FILE_PATH=
   ```

1. 從Adobe Developer主控台下載檔案。 前往專案的根目錄，然後按一下右上角的「全部下載」。 檔案下載方式 `<namespace>-<workspace>.json` 作為檔案名稱。 执行下列操作之一：

   * 將檔案重新命名為 `console.json` 並將其移動到專案的根目錄中。
   * 或者，您也可以將絕對路徑新增至Adobe Developer主控台整合JSON檔案。 這是相同的 [`console.json`](https://developer.adobe.com/app-builder/docs/getting_started/first_app/#42-developer-is-not-logged-in-as-enterprise-organization-user) 已在專案工作區中下載的檔案。

      ```conf
      ASSET_COMPUTE_INTEGRATION_FILE_PATH=
      ```

1. 新增S3或Azure儲存體認證。 您只需要存取一個雲端儲存解決方案。

   ```conf
   # S3 credentials
   S3_BUCKET=
   AWS_ACCESS_KEY_ID=
   AWS_SECRET_ACCESS_KEY=
   AWS_REGION=
   
   # Azure Storage credentials
   AZURE_STORAGE_ACCOUNT=
   AZURE_STORAGE_KEY=
   AZURE_STORAGE_CONTAINER_NAME=
   ```

>[!TIP]
>
>此 `config.json` 檔案包含認證。 從您的專案中，將JSON檔案新增至 `.gitignore` 檔案以防止其共用。 這同樣適用於.env和.aio檔案。

## 執行應用程式 {#run-custom-application}

使用Asset compute Developer Tool執行應用程式之前，請正確設定 [認證](#developer-tool-credentials).

若要在開發人員工具中執行應用程式，請使用 `aio app run` 命令。 它將動作部署至 [!DNL Adobe I/O] 執行階段並在本機電腦上啟動開發工具。 此工具用於在開發期間測試應用程式請求。 以下是範例轉譯請求：

```json
"renditions": [
    {
        "worker": "https://1234_my_namespace.adobeioruntime.net/api/v1/web/example-custom-worker-master/worker",
        "name": "image.jpg"
    }
]
```

>[!NOTE]
>
>請勿使用 `--local` 標幟為 `run` 命令。 無法搭配使用 [!DNL Asset Compute] 自訂應用程式和Asset compute開發人員工具。 自訂應用程式是由 [!DNL Asset Compute Service] 無法存取在開發人員本機電腦上執行的動作。

另請參閱 [此處](test-custom-application.md) 如何測試和偵錯您的應用程式。 當您完成自訂應用程式的開發時， [部署您的自訂應用程式](deploy-custom-application.md).

## 嘗試Adobe提供的範例應用程式 {#try-sample}

以下是自訂應用程式的範例：

* [worker-basic](https://github.com/adobe/asset-compute-example-workers/tree/master/projects/worker-basic)
* [worker-animal-pictures](https://github.com/adobe/asset-compute-example-workers/tree/master/projects/worker-animal-pictures)

### 範本自訂應用程式 {#template-custom-application}

此 [worker-basic](https://github.com/adobe/asset-compute-example-workers/tree/master/projects/worker-basic) 是範本應用程式。 它只會複製來源檔案來產生轉譯。 此應用程式的內容為選擇時收到的範本 `Adobe Asset Compute` 建立aio應用程式時。

應用程式檔案， [`worker-basic.js`](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-basic/worker-basic.js) 使用 [`asset-compute-sdk`](https://github.com/adobe/asset-compute-sdk#overview) 若要下載來源檔案，請協調每個轉譯處理，並將產生的轉譯上傳回雲端儲存空間。

此 [`renditionCallback`](https://github.com/adobe/asset-compute-sdk#rendition-callback-for-worker-required) 定義在應用程式程式碼內，是執行所有應用程式處理邏輯的位置。 中的轉譯回呼 `worker-basic` 只需將來源檔案內容複製到轉譯檔案即可。

```javascript
const { worker } = require('@adobe/asset-compute-sdk');
const fs = require('fs').promises;

exports.main = worker(async (source, rendition) => {
    // copy source to rendition to transfer 1:1
    await fs.copyFile(source.path, rendition.path);
});
```

## 呼叫外部API {#call-external-api}

在應用程式程式碼中，您可以進行外部API呼叫，以協助處理應用程式。 以下為呼叫外部API的應用程式檔案範例。

```javascript
exports.main = worker(async function (source, rendition) {

    const response = await fetch('https://adobe.com', {
        method: 'GET',
        Authorization: params.AUTH_KEY
    })
});
```

例如， [`worker-animal-pictures`](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-animal-pictures/worker-animal-pictures.js#L46) 會使用，從Wikimedia對靜態URL發出擷取請求 [`node-httptransfer`](https://github.com/adobe/node-httptransfer#node-httptransfer) 資料庫。

<!-- TBD: Revisit later to see if this note is required.
>[!NOTE]
>
>For extra authorization for these API calls, see [custom authorization checks](#custom-authorization-checks).
-->

### 傳遞自訂引數 {#pass-custom-parameters}

您可以透過轉譯物件傳遞自訂已定義的引數。 它們可在應用程式內參照，位置如下： [`rendition` 指示](https://github.com/adobe/asset-compute-sdk#rendition). 以下是轉譯物件的範例：

```json
"renditions": [
    {
        "worker": "https://1234_my_namespace.adobeioruntime.net/api/v1/web/example-custom-worker-master/worker",
        "name": "image.jpg",
        "my-custom-parameter": "my-custom-parameter-value"
    }
]
```

存取自訂引數之應用程式檔案的範例為：

```javascript
exports.main = worker(async function (source, rendition) {

    const customParam = rendition.instructions['my-custom-parameter'];
    console.log('Custom paramter:', customParam);
    // should print out `Custom parameter: "my-custom-parameter-value"`
});
```

此 `example-worker-animal-pictures` 傳遞自訂引數 [`animal`](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-animal-pictures/worker-animal-pictures.js#L39) 以判斷要從Wikimedia擷取的檔案。

## 驗證和授權支援 {#authentication-authorization-support}

依預設，Asset compute自訂應用程式會隨App Builder專案的授權和驗證檢查提供。 這可透過設定 `require-adobe-auth` 註解至 `true` 在 `manifest.yml`.

### 存取其他AdobeAPI {#access-adobe-apis}

<!-- TBD: Revisit this section. Where do we document console workspace creation?
-->

將API服務新增至 [!DNL Asset Compute] 在設定中建立的Console工作區。 這些服務屬於產生的JWT存取權杖的一部分 [!DNL Asset Compute Service]. 可在應用程式動作中存取權杖和其他認證 `params` 物件。

```javascript
const accessToken = params.auth.accessToken; // JWT token for Technical Account with entitlements from the console workspace to the API service
const clientId = params.auth.clientId; // Technical Account client Id
const orgId = params.auth.orgId; // Experience Cloud Organization
```

### 傳遞協力廠商系統的認證 {#pass-credentials-for-tp}

若要處理其他外部服務的認證，請將這些認證作為預設引數傳遞給動作。 傳輸中會自動加密這些檔案。 如需詳細資訊，請參閱 [在執行階段開發人員指南中建立動作](https://www.adobe.io/apis/experienceplatform/runtime/docs.html#!adobedocs/adobeio-runtime/master/guides/creating_actions.md). 然後在部署期間使用環境變數設定它們。 這些引數可在以下位置存取： `params` 物件。

設定內的預設引數 `inputs` 在 `manifest.yml`：

```yaml
packages:
  __APP_PACKAGE__:
    actions:
      worker:
        function: 'index.js'
        runtime: 'nodejs:10'
        web: true
        inputs:
           secretKey: $SECRET_KEY
        annotations:
          require-adobe-auth: true
```

此 `$VAR` 運算式會從名為的環境變數中讀取值 `VAR`.

在開發期間，可以在本機ENV檔案中將值設定為 `aio` 除了從叫用殼層設定的變數之外，還會自動從ENV檔案讀取環境變數。 在此範例中，ENV檔案看起來像這樣：

```CONF
#...
SECRET_KEY=secret-value
```

對於生產部署，您可以在CI系統中設定環境變數，例如在GitHub動作中使用秘密。 最後，存取應用程式內的預設引數，如下所示：

```javascript
const key = params.secretKey;
```

## 調整應用程式大小 {#sizing-workers}

應用程式會在的容器中執行 [!DNL Adobe I/O] 執行階段為 [限制](https://www.adobe.io/apis/experienceplatform/runtime/docs.html#!adobedocs/adobeio-runtime/master/guides/system_settings.md) 可透過以下方式設定： `manifest.yml`：

```yaml
    actions:
      myworker:
        function: /actions/myworker/index.js
        limits:
          timeout: 300000
          memorySize: 512
          concurrency: 1
```

由於通常由Asset compute應用程式完成的更廣泛處理，因此更有可能需要調整這些限制以獲得最佳效能（足夠大以處理二進位資產）和效率（不會因未使用的容器記憶體而浪費資源）。

執行階段中動作的預設逾時為一分鐘，但可透過設定 `timeout` 限制（毫秒）。 如果您希望處理較大的檔案，請增加此時間。 請考慮下載來源、處理檔案及上傳轉譯所需的總時間。 如果動作逾時（即在指定的逾時限制之前未傳回啟動），執行階段會捨棄容器且不重複使用它。

asset compute應用程式本質上是網路和磁碟的輸入或輸出繫結。 必須先下載來源檔案，處理通常需要大量資源，然後才會再次上傳產生的轉譯。

動作容器可用的記憶體由指定 `memorySize` 以MB為單位。 目前，這也定義了容器取得的CPU存取許可權，最重要的是，這是使用執行階段成本的關鍵要素（容器越大，成本越高）。 當您的處理需要更多記憶體或CPU時，請在此處使用較大的值，但請注意不要因為容器越大，整體處理量越低而浪費資源。

此外，您也可以使用控制容器內的動作並行 `concurrency` 設定。 這是單一容器（相同動作）所取得的並行啟用數。 在此模型中，動作容器就像是Node.js伺服器，可接收多個並行請求，最多可達該限制。 如果未設定，執行階段中的預設值為200，這非常適合用於較小的App Builder動作，但通常對於Asset compute應用程式來說太大了，因為它們有較密集的本機處理和磁碟活動。 某些應用程式（視其實施而定）可能無法順利搭配並行活動運作。 asset computeSDK會將檔案寫入不同的唯一資料夾，藉此確保可分隔啟用作業。

測試應用程式以找出最佳數量 `concurrency` 和 `memorySize`. 較大的容器=較高的記憶體限制可能允許更多並行，但也可能對較低的流量造成浪費。
