---
title: Workerman 實踐
---

# Workerman 實踐

Workerman 提供了基礎的 HTTP 伺服器，其運作細節則需要開發者自行實作。

針對 Workerman 與 CodeIgniter 的整合，我們提供了一套簡單的實作，讓開發者可以快速上手。

## HTTP Service

對於 Workerman 來說 CodeIniger 是一個被實作出來的 Worker。

它提供了兩個主要的功能：

1. 作為靜態檔案伺服器，響應在 `public` 資料夾下的靜態檔案。
2. 作為 HTTP 伺服器，將請求轉換為 PSR-7 格式，並交由 Burner Core 進行處理。

你可以關注 Burner 所實作的 `src/Worker/CodeIgniter.php` ，關鍵內容如下：

```php
$webWorker->onMessage = static function (TcpConnection $connection, Request $request) use ($workermanConfig) {
    $workermanConfig->runtimeTcpConnection($connection);

    // Static File
    $response = \Monken\CIBurner\Workerman\StaticFile::withFile($request);
    if ((null === $response) === false) {
        $connection->send($response);

        return;
    }

    // init psr7 request
    $_SERVER['HTTP_USER_AGENT'] = $request->header('User-Agent');
    $psrRequest                 = (new PsrRequest(
        $request->method(),
        $request->uri(),
        $request->header(),
        $request->rawBody(),
        $request->protocolVersion(),
        $_SERVER
    ))->withQueryParams($request->get())
        ->withCookieParams($request->cookie())
        ->withParsedBody($request->post() ?? [])
        ->withUploadedFiles($request->file() ?? []);
    unset($request);

    // process response
    if ($response === null) {
        /** @var \Psr\Http\Message\ResponseInterface */
        $response = \Monken\CIBurner\App::run($psrRequest);
    }

    $workermanResponse = new Response(
        $response->getStatusCode(),
        $response->getHeaders(),
        $response->getBody()->getContents()
    );

    $connection->send($workermanResponse);
    Events::trigger('burnerAfterSendResponse', $connection);
    \Monken\CIBurner\App::clean();
};
```

當有 HTTP 請求進來時，Burner 會先檢查是否為靜態檔案請求，若是則直接回應。否則，Burner 會將請求轉換為 PSR-7 格式，並交由 Burner Core 進行處理。最後，Burner 會將處理結果轉換為 Workerman 格式，並回應給客戶端。

## App Container

當一個基於 Workerman 的伺服器軟體被啟動時，它可以不只運行著單一業務邏輯的 Worker，你可以依據你的需求，啟動多個 Worker 來運行不同的業務邏輯。

舉個例子，在 Burner 中我們實作了兩種 Worker，分別是：

1. `CodeIgniter`：用於處理 HTTP 請求。
2. `FileMonitor`：用於監聽檔案變更。

在 `app/Config/Workerman.php` 中，我們可以看到：

```php
public $serverWorkers = [
    \Monken\CIBurner\Workerman\Worker\CodeIgniter::class,
    // \Monken\CIBurner\Workerman\Worker\FileMonitor::class,
];
```

這兩個 Worker 都是繼承自 `Monken\CIBurner\Workerman\Worker\WorkerRegistrar`，皆實作了 `initWorker` 方法。在你執行 `php spark burner:start` 時， Burener 會自動將這兩個 Worker 一起啟動（依據你的組態設定是否全數宣告）。

Burner 實作了一個 Workerman 專屬的 `AppContainer` ，這個類別中將保存著所有的 Workerman Worker 物件實體，並提供了一些方法來操作這些實體。

你可以關注 Burner 的 `src/Worker.php` ，關鍵內容如下：

```php
//Burner Core And Worker Base Settings
\Monken\CIBurner\App::setConfig(config('Burner'));
/** @var \Config\Workerman */
$workermanConfig = Factories::config('Workerman');
Config::staticSetting($workermanConfig);

AppContainer::registerWorkers($workermanConfig->serverWorkers);
AppContainer::init();
```

我們會將伺服器所需要使用到的 Worker 裝載到 `AppContainer` 中，並且透過 `AppContainer` 來統一初始化這些 Worker。這意味著，你可以在 `AppContainer` 中加入你自己的 Worker，並且透過 `AppContainer` 來統一初始化這些 Worker。

### 製作自己的 Worker

你可以在 `app/Workers` 資料夾下，建立一個像是這樣的檔案，並起名為 `MyWorker.php`：

```php
<?php

namespace App\Workers;

use Monken\CIBurner\Workerman\Worker\WorkerRegistrar;
use Workerman\Worker;

class MyWorker extends WorkerRegistrar
{
    public function initWorker(): Worker
    {
        $worker = new Worker();
        $worker->name = 'MyWorker';
        $worker->count = 1;
        $worker->onWorkerStart = function () {
            print_r('MyWorker is running' . PHP_EOL );
        };
        return $worker;
    }
}
```

接著，你可以在 `app/Config/Workerman.php` 中，將你的 Worker 加入到 `serverWorkers` 中：

```php
public $serverWorkers = [
    \Monken\CIBurner\Workerman\Worker\CodeIgniter::class,
    // \Monken\CIBurner\Workerman\Worker\FileMonitor::class,
    \App\Workers\MyWorker::class,
];
```

當你執行 `php spark burner:start` 時，你的 Worker 將會被 Burner 自動啟動。

再撰寫 Worker 時你可以盡情地使用任何符合命名空間規則的類別，這些煩人的命名空間載入，Burner 已為你處理好了。

若你需要更多 Worker 撰寫時的注意事項，或是更多關於 Workerman 的知識，我們推薦你參閱[官方文件](https://www.workerman.net/doc/workerman/worker.html)。

### 可用方法

#### AppContainer::getWorker

透過 `AppContainer::getWorker` 方法，傳入類別名稱後，你就可以取得指定的 Workerman Worker 實體。

#### 參數

| 參數 | 型別 | 說明 |
| --- | --- | --- |
| $workerClass | string | Worker 類別名稱 |

#### 回傳值

| 型別 | 說明 |
| --- | --- |
| Workerman\Worker | 指定的 Workerman Worker 實體 |
| null | 沒有找到指定的 Worker 實體 |

#### 範例

以 CodeIgniter Worker 為例：

```php
use Monken\CIBurner\Workerman\Worker\CodeIgniter;
$worker = AppContainer::getWorker(CodeIgniter::class);
```


## Burner 事件

Burner 提供了一些伺服器運作中的事件，讓開發者能夠在特定階段執行自己的邏輯。

### Config::initWorker

這通常會用於在 CodeIgniter Worker 初始化時，進行一些你所需要的初始化。你可以透過組態設定檔案 `app/Config/Workerman.php` 中的 `initWorker` 方法，定義每當 CodeIgniter Worker 初始化時你所需要執行的邏輯：

```php
use Workerman\Worker;
public function initWorker(Worker &$worker)
{
    //Your logic here
}
```
### Config::runtimeTcpConnection

當每個 CodeIgniter Worker 發生 TCP 連線時，這個事件將會被觸發。你可以透過組態設定檔案 `app/Config/Workerman.php` 中的 `runtimeTcpConnection` 方法，定義每當 CodeIgniter Worker 發生 TCP 連線時將你所需要執行的邏輯：

```php
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
public function runtimeTcpConnection(TcpConnection &$tcpConnection, Request &$request)
{
    //Your logic here
}
```

## CodeIgniter4 事件

CodeIgniter4 提供了[事件](https://codeigniter.tw/user_guide/extending/events.html)的功能，當 CodeIgniter 執行時，它會遵循著一定的執行順序，然而在某些情況下，你可能會想在執行過程的特定階段執行一些動作。

Burner 在 HTTP 中提供了以下事件給予開發者進行使用：

### burnerAfterSendResponse

這通常會用於在響應後的初始化，若你擁有某些需要在響應後清空的全域變數或某些通用的處理，都非常適合在這個事件中宣告。你可以透過宣告 `burnerAfterSendResponse` 事件，在每當 RoadRunner 響應後將會執行你需要執行的邏輯：

```php
// This code will be executed after sending the response to the client.
Events::on('burnerAfterSendResponse',static function(\OpenSwoole\Http\Server $server)
{
    // Your logic
});
```

事件的使用方式不局限於回呼函數，請參考 [CodeIgniter4 使用手冊](https://codeigniter.tw/user_guide/extending/events.html#id3)提到的方法來選擇最適合你的宣告方式。
