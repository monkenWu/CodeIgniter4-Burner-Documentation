---
title: RoadRunner
---

# RoadRunner

[RoadRunner](https://roadrunner.dev/) 是一款由 Go 語言撰寫的高效能 PHP 伺服器，它讓 PHP 應用程式能夠以 Worker 的方式執行，重複使用長時間存活在記憶體中的 PHP 應用程式，屆此達到高效能的目的。同時，RoadRunner 也是一款完整支援 PSR-7 的 HTTP 伺服器。

## 如何實現

接收到 HTTP 連線後，Burner 會將 PSR-7 請求物件轉換為 CodeIgniter4 請求物件，並開始執行 CodeIgniter4 應用程式，最後將 CodeIgniter4 回應物件轉換為 PSR-7 回應物件，再傳遞給 RoadRunner 伺服器。

與其他 PHP 驅動程式相異的是，RoadRnner 原生就支援 PSR-7，因此 Burner 並不需要再進行轉換（從其他的 Request 物件轉換成 PSR-7 Request 物件）。

## 開始使用

在使用 RoadRunner 之前，請先確認您的環境已經擁有 `PHP8^`、`php-curl`、`php-zip` 與 `php-sockets` 等擴充外掛，也可以直接我們所推薦的 [Docker 環境](/general/docker)。

因目前所釋出的版本為 Beta 版本，所以你需要在 `composer.json` 中加入以下設定：

```json
{
    "minimum-stability": "beta",
    "prefer-stable": true
}
```

當處理好這些前置作業後，你可以在專案根目錄下執行以下指令取得 RoadRunner 驅動程式：

```
composer require monken/codeigniter4-burner-roadrunner:^1.0@beta
```

{% info 備註 %}

如果在執行上述指令的當下你尚未安裝 CodeIgniter4-Burner 程式庫，Composer 將會自動拉取相關的依賴項目。

{% end %}

緊接著，你需要使用以下指令初始化 Burner 與 RoadRunner 驅動程式：

```
php spark burner:init RoadRunner
```

這個指令會先下載最新的 RoadRunner 執行檔（即 rr 或 rr.exe），再將它移動至專案 `vendor/bin` 目錄下，最後將 `rr` 或 `rr.exe` 的執行權限設定為可執行。

當指令執行成功後，你應該可以在 `project_root/app/Config` 資料夾中看到`Burner.php` 檔案，並在專案根目錄中發現 `project_root/.rr.yaml`，你可以透過調整這兩個檔案來細緻地控制你的伺服器設定。

{% info 備註 %}

如果你重複執行 `burner:init` 指令，已存在的組態設定檔案將會被重新命名而不是直接被覆蓋。

{% end %}

Bruner 已經為你預設了所有組態設定，你可以在需要時再去更動你的設定檔案。

當一切準備就緒後，你可以直接執行以下指令開啟你的伺服器：

```
php spark burner:start
```

在預設的情況下，你的伺服器應該會執行在 `8080` 連接埠上頭，你可以立即透過 `localhost:8080` 來瀏覽你的專案頁面。

順利的話，你的 CodeIgniter4 應該已經成功地運行在 RoadRunner 上。
