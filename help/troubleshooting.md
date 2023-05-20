---
title: 疑难解答 [!DNL Asset Compute Service]
description: 疑難排解和偵錯自訂應用程式，使用 [!DNL Asset Compute Service].
exl-id: 017fff91-e5e9-4a30-babf-5faa1ebefc2f
source-git-commit: 2dde177933477dc9ac2ff5a55af1fd2366e18359
workflow-type: tm+mt
source-wordcount: '291'
ht-degree: 1%

---

# 疑难解答 {#troubleshoot}

可協助您針對Asset compute服務進行疑難排解的一些一般疑難排解提示如下：

* 請確定JavaScript應用程式在啟動時不會當機。 這類當機通常與缺少程式庫或相依性有關。
* 請確定應用程式的所有要安裝的相依性都被引用 `package.json` 檔案。
* 確保任何因清理失敗而導致的錯誤不會產生隱藏原始問題的錯誤。

* 第一次使用新工具啟動開發人員工具時 [!DNL Asset Compute Service] 整合時，如果Asset compute事件日誌未完全設定，則第一個處理請求可能會失敗。 請等候一段時間以設定日誌，然後再傳送另一個請求。
* 如果您在傳送Asset compute時遇到錯誤 `/register` 或 `/process` 請求，確定所有必要的API都已新增至 [!DNL Adobe I/O] 專案和工作區 — 亦即Asset compute、 [!DNL Adobe I/O] 活動， [!DNL Adobe I/O] 事件管理，以及 [!DNL Adobe I/O] 執行階段。

## 登入問題透過 [!DNL Adobe I/O] CLI {#login-via-aio-cli}

如果您無法登入 [!DNL Adobe Developer Console] [透過 [!DNL Adobe I/O] CLI](https://developer.adobe.com/app-builder/docs/getting_started/first_app/#3-signing-in-from-cli)，然後手動新增開發、測試和部署自訂應用程式所需的認證：

1. 導覽至您的Adobe Developer App Builder專案和工作區，網址為 [Adobe Developer主控台](https://console.adobe.io/) 並按下 **[!UICONTROL 下載]** 從右上角。 開啟此檔案，並將此JSON儲存至您電腦上的安全位置。

1. 導覽至Adobe Developer App Builder應用程式中的ENV檔案。

1. 新增 [!DNL Adobe I/O] 執行階段認證。 取得 [!DNL Adobe I/O] 下載的JSON中的執行階段認證。 認證位於 `project.workspace.services.runtime`. 新增 [!DNL Adobe I/O] 中的執行階段認證 `AIO_runtime_XXX` 變數：

   ```json
   AIO_runtime_auth=
   AIO_runtime_namespace=
   ```

1. 在步驟1中新增已下載JSON的絕對路徑：

   ```json
       ASSET_COMPUTE_INTEGRATION_FILE_PATH=
   ```

1. 設定其餘的 [必要的認證](develop-custom-application.md) 開發人員工具所需。

<!-- TBD for later:
Add any best practices for developers in this section:
* Any items to take care of when creating projects.
* Any naming conventions, reserved keywords, etc.?
* Any terms that can become a source of confusion later based on our OOTB naming.

* If required, add limitations for custom applications and spin those off as best practices.
* Do NOT borrow any content from https://git.corp.adobe.com/nui/nui/blob/master/doc/worker_api.md. It is outdated and irrelevant for 3rd party custom applications.
-->
