---
title: OpenSwoole 開發建議
---

# OpenSwoole 開發建議

## Fast Cache

OpenSwoole 提供了一個快速的 Cache 系統，可以在多個 Worker 間共享資料，並且可以在多個 Worker 間同步資料。我們支援 [Swoole-Table](https://openswoole.com/docs/modules/swoole-table) 作為快取的儲存介面，你可以在你的專案中使用它，就像你熟悉的一樣！

你可以在 `app/Config/OpenSwoole.php` 設定檔中，加入以下設定打開 fast cache 功能：

```php
/**
 * Whether to open the Key/Value Cache provided by Burner.
 * Shared high-speed caching with Swoole-Table implementation.
 * 
 * @var boolean
 */
public $fastCache = true;

/**
 * Buerner Swoole-Table Driver Settings
 * 
 * @var string[]
 */
public $fastCacheConfig = [
    //Number of rows of the burner cache table
    'tableSize' => 4096,   
    //Key/value key Maximum length of string
    'keyLength' => 1024,
    //The maximum length of the key/value (if the save type is object, array, string).
    'valueStringLength' => 1024
];
```

### 使用 CodeIgniter4 Cache

CodeIgniter 提供了單例的 Cache 管理方式，無論你在程式的何處你都可以使用 `cache()` 函式來獲取 Cache 實體。Burner 也提供了一個 `Burner` 的 Cache Handler。 

你需要開啟 `app/Config/Cache` 並加入以下設定：

```php
public $validHandlers = [
    //hide
    'burner' => \Monken\CIBurner\BurnerCacheHandler::class
];
```

最後，你需要從 `Primary Handler` 切換到 `Burner`。

```php
public $handler = 'burner';
```

現在你可以在 Burner 提供的 Swoole-Table Cache 中，以同樣的方式操作 [Cache Library](https://www.codeigniter.com/user_guide/libraries/caching.html)。請注意，Swoole-Table 會因為伺服器重啟而失去所有資料，它只是一個資料在 Worker 之間共享的解決方案。

### 直接取得 Cache 實體

如果你的專案已經有使用了類似 Redis 的 Cache 系統，並且已使用 CodeIgniter4 的 Cache 管理機制，你也可以直接使用 Burner 提供的全域方法來獲取 Cache 實體，這樣你就不需要糾結於 Cache 只能有一個 Handler 的問題。

只需要將  `app/Config/OpenSwoole.php` 的 `fastCache` 設定為 `true`，然後在你的程式中使用以下方式來獲取 Cache 實體：

```php
$fastCache = \Monken\CIBurner\OpenSwoole\Worker::getFastCache();
```

## 自動重新載入

在預設的 OpenSwoole 狀況下，你必須在每次修改 PHP 檔案後，重新啟動伺服器，才能讓你的修改生效。這似乎不太友善。

你可以修改 `app/Config/OpenSwoole.php` 設定檔，加入以下設定並重新啟動伺服器。

```php
/**
 * Auto-scan changed files
 *
 * @var bool
 */
public $autoReload = true;

/**
 * Auto Reload Mode
 *
 * @var string restart or reload
 */
public $autoReloadMode = 'restart';
```

Burner 提供了兩種重新載入的方式，你可以透過調整 `autoReloadMode` 來切換。

* `restart` 意味著每次檔案變更時，伺服器都會自動重新啟動。就好像你自己關閉了伺服器，然後又重新啟動了一樣，這樣可以確保所有的 php 檔案都重新載入。
* `reload` 只會重新載入正在運行的 Worker，就像 [文件](https://openswoole.com/docs/modules/swoole-server-reload#hot-code-linux-signal-trigger) 所說的那樣。請注意，這種方式可能無法處理所有情況，例如，你可能會透過 `composer require/update` 來產生專案核心的 php 檔案的變更。

{% info 備註 %}

`Automatic reload` 方法非常耗費資源，請不要在正式環境中啟用此選項。

{% end %}

## 在只有一個 Worker 的環境中開發和除錯

由於 OpenSwoole 與其他伺服器軟體（如 Nginx、Apache）有根本上的差異，因此每個 Codeigniter4 將作為 Worker 的形式存在於 RAM 中，HTTP 請求將重用這些 Worker 來處理。因此，我們在只有一個 Worker 的環境中開發和測試穩定性，以證明它也可以在正式環境中的多個 Worker 中正常運行。

你可以參考 `app/Config/OpenSwoole.php` 設定檔中以下的設定，將 Worker 的數量降到最低：

```php

public $config = [
    'worker_num' => 1,
    /** hide */
];
```
