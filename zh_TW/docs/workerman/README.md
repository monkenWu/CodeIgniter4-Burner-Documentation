---
title: Workerman
---

# Workerman

[Workerman](https://www.workerman.net/) 是一款由原生 PHP 開發的高效能應用容器，它不像 OpenSwoole 需要倚賴特別的外部 PHP Extension，或是需要像 RoadRunner 一樣需要一個外部的 Go Server ，它是個完全以 PHP 實作的事件驅動高效能框架。

## 如何實現

接收到 HTTP 連線後，Burner 會將 Workerman 的 Request 物件轉換成 PSR-7 請求物件，再透過 Burner-Core 轉換為 CodeIgniter4 請求物件，最後才開始執行 CodeIgniter4 應用程式。

待 CodeIgniter4 處理完請求後，Burner-Core 會將 CodeIgniter4 Request 物件轉換成 PSR-7 回應物件，再傳遞給 Workerman 驅動程式進行處理，最後會以 Workerman Response 物件進行響應。

## 開始使用

在使用 RoadRunner 之前，請先確認您的環境已經擁有 `PHP8^`、`php-pcntl`、`php-posix` 與非必要但推薦開啟的 [`php-event`](https://www.php.net/manual/en/book.event.php)等擴充外掛，也可以直接使用我們推薦的 [Docker 環境](/general/docker)。

因目前所釋出的版本為 Beta 版本，所以你需要在 `composer.json` 中加入以下設定：

```json
{
    "minimum-stability": "beta",
    "prefer-stable": true
}
```

當處理好這些前置作業後，你可以在專案根目錄下執行以下指令取得 Workerman 驅動程式：

```
composer require monken/codeigniter4-burner-workerman:^1.0@beta
```

{% info 備註 %}

如果在執行上述指令的當下你尚未安裝 CodeIgniter4-Burner 程式庫，Composer 將會自動拉取相關的依賴項目。

{% end %}

緊接著，你需要使用以下指令初始化 Burner 與 Workerman 驅動程式：

```
php spark burner:init Workerman
```

當指令執行成功後，你應該可以在 `project_root/app/Config` 資料夾中看到兩個被新增的檔案，分別是：`Burner.php` 與 `Workerman.php`，你可以透過調整這兩個檔案來細緻地控制你的伺服器設定。

{% info 備註 %}

如果你重複執行 `burner:init` 指令，已存在的組態設定檔案將會被重新命名而不是直接被覆蓋。

{% end %}

Bruner 已經為你預設了所有組態設定，你可以在需要時再去更動你的設定檔案。

當一切準備就緒後，你可以直接執行以下指令開啟你的伺服器：

```
php spark burner:start
```

在預設的情況下，你的伺服器應該會執行在 `8080` 連接埠上頭，你可以立即透過 `localhost:8080` 來瀏覽你的專案頁面。

順利的話，你的 CodeIgniter4 應該已經成功地運行在 Workerman 上。
