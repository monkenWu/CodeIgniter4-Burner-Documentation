---
title: OpenSwoole
---

# OpenSwoole

OpenSwoole 是從 Swoole 分叉出的新版本，它們有著相異的維護團隊。在 Burner 開發時 OpenSwoole 已迎來了 `22.0.0` 的更新，在這個版本中命名空間已從 `Swoole` 遷移至 `OpenSwoole` ，這意味著 Burner 將不支援在 `22.0.0` 以下的 `OpenSwoole` 下執行，也沒辦法相容於 `Swoole` 。

## 如何實現

接收到 HTTP 請求時，OpenSwoole 驅動程式會將 OpenSwoole 請求物件轉換成 `PSR-7` 介面的請求物件，接著再將 `PSR-7` 請求物件傳遞給 `burner-core` 進行處理。 `burner-core` 接收到 `PSR-7` 請求物件時會將它轉換為 `CodeIgniter4` 專屬的請求物件，並包含一些框架所需的預處理。

當 `CodeIgniter4` 將請求處理完畢後， `burner-core` 會將 `CodeIgniter4` 的響應物件轉換為 `PSR-7` 響應物件，再將其傳回給 OpenSwoole 驅動程式。

## 開始使用

在使用 Burner 提供的 OpenSwoole 驅動程式之前，你得確保目前的系統環境已經擁有 `PHP8^` 與 `OpenSwoole` 擴充外掛。你可以參考 [如何安裝 OpenSwoole](https://openswoole.com/docs/get-started/installation)，或者是使用我們推薦的 [Docker 環境](/general/docker)。

當處理好這些前置作業後，你可以在專案根目錄下執行以下指令取得 OpenSwoole 驅動程式：

```
composer require monken/codeigniter4-burner-openswoole:1.0.0-beta.1
```

{% info 備註 %}

如果在執行上述指令的當下你尚未安裝 CodeIgniter4-Burner 程式庫，Composer 將會自動拉取相關的依賴項目。

{% end %}

緊接著，你需要使用以下指令初始化 Burner 與 OpenSwoole 驅動程式：

```
php spark burner:init OpenSwoole [http or websocket]
```

你可以在指令後方自由選擇你的伺服器要使用 HTTP 或 Websocket 模式執行，這個參數在未傳入的情況下預設為 `http` 。 `http` 參數會產生一般的 HTTP 伺服器組態設定檔案，如果使用 `websocket` 參數，則會產生 WebSocket 專屬（包括 HTTP）的組態設定檔案。

當指令執行成功後，你應該可以在 `project_root/app/Config` 資料夾中看到兩個被新增的檔案，分別是：`Burner.php` 與 `OpenSwoole.php`，你可以透過調整這兩個檔案來細緻地控制你的伺服器設定。

{% info 備註 %}

如果你重複執行 `burner:init` 指令，已存在的組態設定檔案將會被重新命名而不是直接被覆蓋。

{% end %}

Bruner 已經為你預設了所有組態設定，你可以在需要時再去更動你的設定檔案。

當一切準備就緒後，你可以直接執行以下指令開啟你的伺服器：

```
php spark burner:start
```

在預設的情況下，你的伺服器應該會執行在 `8080` 連接埠上頭，你可以立即透過 `localhost:8080` 來瀏覽你的專案頁面。

順利的話，你的 CodeIgniter4 應該已經成功地運行在 OpenSwoole 上。
