---
title: Workerman 開發建議
---

# Workerman 開發建議

## 自動重新載入

在預設的情況下，每次修改 PHP 檔案後，你必須透過重新啟動伺服器使你的更改生效但這在開發過程中，似乎不這麼友好。

Burner 提供了一個自動重新載入的功能，只需要打開你的 `config/workerman.php` 檔案，將 `$serverWorkers` 的其中一個註解項目打開即可。

```php
public $serverWorkers= [
    \Monken\CIBurner\Workerman\Worker\CodeIgniter::class,
    //Open this line to enable auto reload
    \Monken\CIBurner\Workerman\Worker\FileMonitor::class,
];
```

你可以透過 `FileMonitor` 來監控你的檔案，當你的檔案有變動時，Burner 會自動重新啟動你的伺服器。

你也可以自行修改 `FileMonitor` 的監控路徑，只需要在 `config/workerman.php` 中，修改 `$autoReloadDir` 即可。

```php
public $autoReloadDir = '/app/CodeIgniter4-Burner/dev';
```

Burner 提供了兩種重新載入的方式，你可以透過調整 `config/workerman.php` 中的 `autoReloadMode` 來切換。

```php
public $autoReloadMode = 'restart';
```

* `restart` 意味著每次檔案變更時，伺服器都會自動重新啟動。就好像你自己關閉了伺服器，然後又重新啟動了一樣，這樣可以確保所有的 php 檔案都重新載入。
* `reload` 只會重新載入正在運行的 Worker，就像 [文件](https://www.workerman.net/doc/workerman/faq/reload-principle.html) 所說的那樣。請注意，這種方式可能無法處理所有情況，例如，透過 `composer require/update` 產生的專案核心檔案變更。

{% info 備註 %}

自動重新載入非常耗費資源，請不要在正式環境中打開它。

{% end %}

## 在只有一個 Worker 的環境中開發和除錯

由於 Workerman 與其他伺服器軟體（如 Nginx、Apache）有根本上的差異，因此每個 Codeigniter 將作為 Worker 的形式存在於 RAM 中，HTTP 請求將重用這些 Worker 來處理。因此，我們在只有一個 Worker 的環境中開發和測試穩定性，以證明它也可以在正式環境中的多個 Worker 中正常運行。

你可以參考 `app/Config/Workerman.php` 設定檔中以下的設定，將 CodeIgniter Worker 的數量降到最低：

```php
public $workerCount = 1;
```
