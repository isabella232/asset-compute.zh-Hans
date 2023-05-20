---
title: 測試和偵錯 [!DNL Asset Compute Service] 自訂應用程式
description: 測試和偵錯 [!DNL Asset Compute Service] 自訂應用程式。
exl-id: c2534904-0a07-465e-acea-3cb578d3bc08
source-git-commit: 2dde177933477dc9ac2ff5a55af1fd2366e18359
workflow-type: tm+mt
source-wordcount: '812'
ht-degree: 0%

---

# 測試並偵錯自訂應用程式 {#test-debug-custom-worker}

## 執行自訂應用程式的單元測試 {#test-custom-worker}

安裝 [Docker案頭](https://www.docker.com/get-started) 在您的電腦上。 若要測試自訂背景工作，請在應用程式的根目錄執行下列命令：

```bash
$ aio app test
```

<!-- TBD
To run tests for a custom application, run `aio asset-compute test-worker` command at the root of the custom application application.

Document interactively running `adobe-asset-compute` commands `test-worker` and `run-worker`.
-->

這會執行自訂單元測試架構，以便在專案中執行Asset compute應用程式動作，如下所述。 它是透過中的設定連結的 `package.json` 檔案。 也可以有JavaScript單元測試，例如Jest。 `aio app test` 執行兩者。

此 [aio-cli-plugin-asset-compute](https://github.com/adobe/aio-cli-plugin-asset-compute#install-as-local-devdependency) 外掛程式會內嵌為自訂應用程式中的開發相依性，因此不需要安裝在建置/測試系統上。

### 應用程式單元測試架構 {#unit-test-framework}

asset compute應用程式單元測試架構可讓您在不編寫任何程式碼的情況下測試應用程式。 它仰賴應用程式轉譯檔案原則的來源。 必須設定特定檔案和資料夾結構，以使用測試來源檔案、選用引數、預期的轉譯和自訂驗證指令碼來定義測試案例。 依預設，會比較轉譯以取得位元組相等。 此外，您可使用簡單的JSON檔案輕鬆模擬外部HTTP服務。

### 新增測試 {#add-tests}

測試應在 `test` 的根層級資料夾 [!DNL Adobe I/O] 專案。 每個應用程式的測試案例應位於路徑中 `test/asset-compute/<worker-name>`，每個測試案例各有一個資料夾：

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

請檢視 [自訂應用程式範例](https://github.com/adobe/asset-compute-example-workers/) 以取得一些範例。 詳細參考如下。

### 測試輸出 {#test-output}

包含自訂應用程式記錄的詳細測試輸出可在 `build` 位於Adobe Developer App Builder應用程式根目錄的資料夾，如 `aio app test` 輸出。

### 模擬外部服務 {#mock-external-services}

您可以定義以模擬動作中的外部服務呼叫 `mock-<HOST_NAME>.json` 檔案，其中HOST_NAME是您要模擬的主機。 一個使用案例範例是應用程式，可對S3進行個別呼叫。 新的測試結構如下所示：

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

模擬檔案是JSON格式的http回應。 如需詳細資訊，請參閱 [本檔案](https://www.mock-server.com/mock_server/creating_expectations.html). 如果要模擬多個主機名稱，請定義多個 `mock-<mocked-host>.json` 檔案。 以下是的範例模型檔案 `google.com` 已命名 `mock-google.com.json`：

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

範例 `worker-animal-pictures` 包含 [模擬檔案](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-animal-pictures/test/asset-compute/worker-animal-pictures/simple-test/mock-upload.wikimedia.org.json) 與其互動的Wikimedia服務。

#### 在測試案例之間共用檔案 {#share-files-across-test-cases}

如果您共用，建議使用相對符號連結 `file.*`， `params.json` 或 `validate` 跨多個測試的指令碼。 Git支援這些功能。 請務必為您的共用檔案指定唯一的名稱，因為您可能有不同的檔案。 在以下範例中，測試會混合及比對一些共用檔案，以及它們自己的檔案：

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

### 測試預期的錯誤 {#test-unexpected-errors}

錯誤測試案例不應包含預期的 `rendition.*` 檔案且應該定義預期的 `errorReason` 內部 `params.json` 檔案。

>[!NOTE]
>
>如果測試案例不包含預期的 `rendition.*` 檔案且未定義預期的 `errorReason` 內部 `params.json` 檔案，則視為包含下列專案的錯誤案例： `errorReason`.

錯誤測試案例結構：

```json
<error_test_case>/
    file.jpg
    params.json
```

含有錯誤原因的引數檔：

```javascript
{
    "errorReason": "SourceUnsupported",
    // ... other params
}
```

請參閱完整清單和說明 [asset compute錯誤原因](https://github.com/adobe/asset-compute-commons#error-reasons).

## 對自訂應用程式進行偵錯 {#debug-custom-worker}

下列步驟顯示如何使用Visual Studio Code偵錯自訂應用程式。 它可讓您檢視即時記錄、點選中斷點和逐步執行程式碼，以及在每次啟用時即時重新載入本機程式碼變更。

這些步驟中的許多步驟通常都是透過以下方式自動化： `aio` 開箱即用，請參閱以下的「偵錯應用程式」一節： [Adobe Developer App Builder檔案](https://developer.adobe.com/app-builder/docs/getting_started/first_app). 目前，以下步驟包含因應措施。

1. 安裝最新的 [wskdebug](https://github.com/apache/openwhisk-wskdebug) 來自GitHub和選購的 [Ngrok](https://www.npmjs.com/package/ngrok).

   ```shell
   npm install -g @openwhisk/wskdebug
   npm install -g ngrok --unsafe-perm=true
   ```

1. 新增至您的使用者設定JSON檔案。 它會持續使用舊版VS程式碼偵錯工具，新版則會 [部分問題](https://github.com/apache/openwhisk-wskdebug/issues/74) 使用wskdebug： `"debug.javascript.usePreview": false`.
1. 關閉透過以下方式開啟的任何應用程式例項： `aio app run`.
1. 使用部署最新的程式碼 `aio app deploy`.
1. 僅執行Asset computeDevtool，使用 `aio asset-compute devtool`. 保持開啟。
1. 在VS程式碼編輯器中，將以下偵錯設定新增至 `launch.json`：

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

   擷取 `ACTION NAME` 從輸出 `aio app deploy`.

1. 選取 `wskdebug worker` 從run/debug設定，然後按播放圖示。 等待它開始，直到它顯示 **[!UICONTROL 已準備好啟用]** 在 **[!UICONTROL 偵錯主控台]** 視窗。

1. 按一下 **[!UICONTROL 執行]** 在Devtool中。 您可以看到在VS程式碼編輯器中執行的動作，並且記錄開始顯示。

1. 在程式碼中設定中斷點，然後再次執行，應該會點選。

任何程式碼變更都會即時載入，並在下次啟用時立即生效。

>[!NOTE]
>
>自訂應用程式中的每個請求都有兩個啟用。 第一個要求是Web動作，會在SDK程式碼中非同步呼叫自身。 第二個啟用是點選程式碼的啟用。
