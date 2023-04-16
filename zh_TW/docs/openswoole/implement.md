---
title: OpenSwoole 實踐
---

# OpenSwoole HTTP

Burner 為 OpenSwoole 的 HTTP 伺服器建立了如同基礎設施級別的支援。你若選用了 OpenSwoole-HTTP 作為你的伺服器解決方案，在開發上你應該很難察覺的 Burner 的存在。

你可以在 `project_root/app/Config/OpenSwoole.php` 中的 `server` 方法中看到以下程式碼片段：

```php
$server->on('request', static function (Request $swooleRequest, Response $swooleResponse) {
    // Burner handles CodeIgniter4 entry points.
    Worker::httpProcesser($swooleRequest, $swooleResponse);
});
```

所有的 HTTP 請求處理都從 `Worker::httpProcesser` 開始，當這個靜態方法被呼叫後，Burner 便會開始一系列的預處理，並通知 CodeIgniter4 處理你的連線。再接下來，就會進入 CodeIgniter4 的處理流程，如同你所熟知的那樣，一如往常地開發你的應用程式即可。

如果你有一些特別的業務邏輯需要在 CodeIgniter4 處理請求之前先做一些處理，你只需要記得在一切完成後呼叫 `Worker::httpProcesser` 來處理你的請求。

## 事件

CodeIgniter4 提供了[事件](https://codeigniter.tw/user_guide/extending/events.html)的功能，當 CodeIgniter 執行時，它會遵循著一定的執行順序，然而在某些情況下，你可能會想在執行過程的特定階段執行一些動作。

Burner 在 HTTP 中提供了以下事件給予開發者進行使用：

### burnerAfterSendResponse

這通常會用於在響應後的初始化，若你擁有某些需要在響應後清空的全域變數或某些通用的處理，都非常適合在這個事件中宣告。你可以透過宣告 `burnerAfterSendResponse` 事件，在每當 OpenSwoole 響應後將會執行你需要執行的邏輯：

```php
// This code will be executed after sending the response to the client.
Events::on('burnerAfterSendResponse',static function(\OpenSwoole\Http\Server $server)
{
    // Your logic
});
```
事件的使用方式不單單只有回呼函數，請參考 [CodeIgniter4 使用手冊](https://codeigniter.tw/user_guide/extending/events.html#id3)提到的方法來選擇最適合你的宣告方式。

# OpenSwoole Websocket

Burner 為 OpenSwoole WebSocket 伺服器提供了功能上的整合，在開發上你必須遵循著一些使用方式，即可深入地將 WebSocket 深入地整合進你的 CodeIgniter4 專案之中。

首先，你得透過 `php spark burner:init OpenSwoole websocket` 指令來取得 WebSocket 專用的組態設定檔 `project_root/app/Config/OpenSwoole.php` ，你應該會在其中看到這個成員方法發生了改變（與 HTTP 的組態設定不同）。

```php
public function server(Server $server)
{
    $server->on('open', static function (Server $server, Request $request) {
        Worker::setWebsocket($request);
        Worker::push(
            data: 'hi! It\'s Burner Websocket!',
            fd: $request->fd
        );
    });

    $server->on('message', static function (Server $server, Frame $frame) {
        // Burner handles CodeIgniter4 entry points.
        Worker::websocketProcesser($frame, static function (Server $server, Frame $frame) {
            // Frame Not Found Handling
        });
    });

    $server->on('request', static function (Request $swooleRequest, Response $swooleResponse) {
        // Burner handles CodeIgniter4 entry points.
        Worker::httpProcesser($swooleRequest, $swooleResponse);
    });

    $server->on('close', static function (Server $server, int $fd) {
        Worker::unsetWebsocket($fd);
        fwrite(STDOUT, sprintf(
            "client-%d is closed\n",
            $fd,
        ));
    });
}
```

上述的程式碼片段中，你可以看到 `on('open')` 事件，這個事件會在 WebSocket 連線建立時被觸發，你可以在這個事件中進行一些初始化的動作，例如驗證連線者的身份或拒絕使用者連線。你可以注意到，這個事件中的 `Worker::setWebsocket` 方法，這個方法會將連線者的請求資訊儲存到 Burner 提供的 Websocket Pool。這是一個必要的動作，因為 Burner 必須透過這個請求資訊來觸發 CodeIgniter4 的處理。

接著，你可以看到 `on('message')` 事件，這個事件會在 WebSocket 收到訊息時被觸發。你可以注意到，這個事件中的 `Worker::websocketProcesser` 方法，這個方法會將收到的訊息傳遞給 CodeIgniter4 進行處理，並且在 CodeIgniter4 處理完畢。如果你有一些特別的業務邏輯需要在 CodeIgniter4 處理請求之前先做一些處理，你只需要記得在一切完成後呼叫 `Worker::websocketProcesser` 來處理你的請求。

最後，你可以看到 `on('close')` 事件，這個事件會在 WebSocket 連線關閉時被觸發。你可以注意到，這個事件中的 `Worker::unsetWebsocket` 方法，這個方法會將連線者的請求資訊從 Burner 提供的 Websocket Pool 中移除。

另外，WebSocket 也能夠處理普通的 HTTP 請求，透過 `on('request')` 事件，你可以將 WebSocket 伺服器當作一般的 HTTP 伺服器來使用。

## 推送訊息

### 簡單實現

Burner 提供了 `Worker::push` 方法來推送訊息給 WebSocket 連線者，無論你在何處，都可以透過以下的方式來推送訊息給使用者：

```php
use Monken\CIBurner\OpenSwoole\Worker;
\Worker::push(
    data: 'hi! It\'s Burner Websocket!',
    fd: Worker::getFrame()->fd
);
```

我們推薦你在 CodeIgniter4 的控制器中使用 `Worker::push` 方法來推送訊息，就像這樣：

```php
<?php

namespace App\Controllers;

use Monken\CIBurner\OpenSwoole\Worker;

class BurnerWebsocket extends BaseController
{
    public function connect()
    {
        $frameData = Worker::getFrame();
        $nowUserFd = $frameData->fd;
        $data      = $frameData->data;

        Worker::push(
            data: printf('Hi! %s', $nowUserFd),
            fd: $nowUserFd
        );
    }
}
```

這個控制器可能是透過以下的路由來訪問：

```php
$routes->get('burner/websocket', 'BurnerWebsocket::connect');
```

當使用者透過 Websocket 與 `burner/websocket` 與 Burner 握手後，Burner 隨即會將這個連線給傳遞給 CodeIgniter4，讓你可以在 CodeIgniter4 中進行處理。在這裡，你可以盡情地使用 CodeIgniter4 的功能來處理你的業務邏輯。

## 事件

### burnerAfterPushMessage

當 Burner 推送訊息給 WebSocket 連線者後，會觸發 `burnerAfterPushMessage` 事件，你可以在這個事件中進行一些後續處理。

你可以透過以下的方式來註冊 `burnerAfterPushMessage` 事件：

```php
Events::on('burnerAfterSendResponse',static function(\OpenSwoole\Http\Server $server, int $fd, bool $pushResult)
{
    // $server 是 OpenSwoole 的 Server 實例
    // $fd 是目前正在推送的連線者的 fd
    // $pushResult 是推送訊息的結果
});
```

### burnerAfterPushAllMessage

當 Burner 推送訊息給所有 WebSocket 連線者後，會觸發 `burnerAfterPushAllMessage` 事件，你可以在這個事件中進行一些後續處理。

你可以透過以下的方式來註冊 `burnerAfterPushAllMessage` 事件：

```php
Events::on('burnerAfterPushAllMessage',static function(\OpenSwoole\Http\Server $server, array $fds)
{
    // $server 是 OpenSwoole 的 Server 實例
    // $fds 已推送的連線者的 fd 陣列
});
```

# 可用方法

## 通用方法

### Worker::getServer

取得 OpenSwoole 的伺服器實例。

#### 使用範例
 
`getServer()` 的回傳值依據組態設定的不同，可能是 `OpenSwoole\Server` 或 `OpenSwoole\WebSocket\Server`。

```php
use Monken\CIBurner\OpenSwoole\Worker;
$server = Worker::getServer();
```

### Worker::getFastCache

取得共享的 FastCache 實例。

#### 使用範例

```php
use Monken\CIBurner\OpenSwoole\Worker;
$fastCache = Worker::getFastCache();
```

將回傳實作 `CodeIgniter\Cache\CacheInterface` 的物件。

你可以參考 [OpenSwoole 開發建議](/openswoole/suggestion) 來了解更多關於 FastCache 的資訊。

### Worker::httpProcesser

處理 Http 連線。

#### 參數

| 參數 | 型別 | 必填/預設 | 說明 |
| --- | --- | --- | --- |
| request | \OpenSwoole\Http\Request | 必填 | OpenSwoole 的 Request 實例 |
| response | \OpenSwoole\Http\Response | 必填 | OpenSwoole 的 Response 實例 |

#### 使用範例

這個方法通常會在 OpenSwoole 的 `request` 事件中使用，Burner 會將這個請求傳遞給 CodeIgniter4，讓你可以在 CodeIgniter4 中進行處理。

```php
use Monken\CIBurner\OpenSwoole\Worker;
$server->on('request', static function (Request $swooleRequest, Response $swooleResponse) {
    // Burner handles CodeIgniter4 entry points.
    Worker::httpProcesser( $swooleRequest, $swooleResponse );
});
```

## WebSocket 相關方法

### Worker::setWebsocket

設定 WebSocket 連線的相關設定。

#### 參數

| 參數 | 型別 | 必填/預設 | 說明 |
| --- | --- | --- | --- |
| request | \OpenSwoole\Http\Request | 必填 | OpenSwoole 的 Request 實體 |

#### 使用範例

這個方法通常會在 OpenSwoole 的 `open` 事件中使用，在 WebSocket 交握成功後， Burner 需要將 WebSocket 連線時的 Request 儲存在 Websocket Client Pool 中，以便在後續的事件中使用。

```php
use Monken\CIBurner\OpenSwoole\Worker;

$server->on('open', static function (Server $server, Request $request) {
    Worker::setWebsocket($request);
});
```

### Worker::websocketProcesser

處理 WebSocket 連線。

#### 參數

| 參數 | 型別 | 必填/預設 | 說明 |
| --- | --- | --- | --- |
| frame | \OpenSwoole\WebSocket\Frame | 必填 | OpenSwoole 的 WebSocket Frame 實例 |
| notFoundHandler | callable | 選填 | 在 Websocket Client Pool 尋找不到對應的 Frame 時，會呼叫這個 callback |

#### 使用範例

這個方法通常會在 OpenSwoole 的 `message` 事件中使用，Burner 會將這個請求傳遞給 CodeIgniter4，讓你可以在 CodeIgniter4 中進行處理。

這個方法倚賴 `Worker::setWebsocket` 產生的 Websocket Client Pool ，你必須將所有交握後合法的 WebSocket 連線傳遞給 `Worker::setWebsocket`。你才可以再後續的事件中使用 `Worker::websocketProcesser`。

```php
use Monken\CIBurner\OpenSwoole\Worker;

$server->on('message', static function (Server $server, Frame $frame) {
    // Burner handles CodeIgniter4 entry points.
    Worker::websocketProcesser($frame, static function (Server $server, Frame $frame) {
        // Client Not Found Handling
    });
});
```

### Worker::push

將訊息推送給 WebSocket 連線者。

#### 參數

| 參數 | 型別 | 必填/預設 | 說明 |
| --- | --- | --- | --- |
| data | string or binary | 必填 | 要推送的訊息 |
| fd | int | 目前連線者的fd | 要推送的連線者的 fd |
| opcode | int | `WEBSOCKET_OPCODE_TEXT` | 傳遞資料的類型。OpenSwoole 的 `WEBSOCKET_OPCODE_TEXT` 、 `WEBSOCKET_OPCODE_BINARY` 或 `WEBSOCKET_OPCODE_PING` |

#### 使用範例

```php
use Monken\CIBurner\OpenSwoole\Worker;
use OpenSwoole\WebSocket\Server;

Worker::push(
    data: 'hi! It\'s Burner Websocket!',
    fd: Worker::getFrame()->fd,
    opcode: Server::WEBSOCKET_OPCODE_TEXT
);
```

### Worker::pushAll

將訊息推送給所有 WebSocket 連線者，或是列舉出一些連線者的 fd 來推送訊息。

#### 參數

| 參數 | 型別 | 必填/預設 | 說明 |
| --- | --- | --- | --- |
| data | string or callable | 必填 | 要推送的訊息，或是一個回呼函數 |
| fds | array | 與伺服器建立起連線的所有客戶端 | 要推送的連線者的 fd |
| opcode | int | `Server::WEBSOCKET_OPCODE_TEXT` | 傳遞資料的類型。OpenSwoole 的 `WEBSOCKET_OPCODE_TEXT` 、 `WEBSOCKET_OPCODE_BINARY` 或 `WEBSOCKET_OPCODE_PING` |

#### 以 String 推送

將字串推送給所有或是指定的連線者。

```php
use Monken\CIBurner\OpenSwoole\Worker;
use OpenSwoole\WebSocket\Server;

Worker::pushAll(
    data: 'hi! It\'s Burner Websocket!',
    fds: [1, 2, 3],
    opcode: Server::WEBSOCKET_OPCODE_TEXT
);
```

#### 以 Callback 推送

透過 Callback 在推送訊息時包含更多的處理邏輯。

```php
use Monken\CIBurner\OpenSwoole\Worker;
use OpenSwoole\WebSocket\Server;

Worker::pushAll(
    data: function ($fd) {
        // $fd 是目前正在推送的連線者的 fd
        // 你可以在這裡進行一些處理，然後再回傳要推送的訊息
        return sprintf('hi! It\'s Burner Websocket! fd: %d', $fd);
    },
    fds: [1, 2, 3],
    opcode: Server::WEBSOCKET_OPCODE_TEXT
);
```

#### 以 Callback 推送並包含更多設定

你可以在回呼函數中回傳一個陣列，來包含更多的設定。

```php
use Monken\CIBurner\OpenSwoole\Worker;
use OpenSwoole\WebSocket\Server;

Worker::pushAll(
    data: function ($fd) {
        // $fd 是目前正在推送的連線者的 fd
        // 你可以在這裡進行一些處理，然後再回傳要推送的訊息
        return [
            // 要推送的訊息
            'message' => sprintf('hi! It\'s Burner Websocket! fd: %d', $fd),
            // 傳遞資料的類型
            'opcode' => Server::WEBSOCKET_OPCODE_TEXT
        ];
    },
    fds: [1, 2, 3]
);
```

### Worker::getFrame

你可以在程式的任何地方，取得目前的 WebSocket Frame。

#### 使用範例

```php
use Monken\CIBurner\OpenSwoole\Worker;

$frame = Worker::getFrame();
```

`$frame` 的型別為 `OpenSwoole\WebSocket\Frame`，你可以參考 [OpenSwoole 的文件](https://openswoole.com/docs/modules/swoole-websocket-frame) 來了解更多。

### Worker::unsetWebsocket

將 Clint 從 Websocket Client Pool 中移除。

#### 參數

| 參數 | 型別 | 必填/預設 | 說明 |
| --- | --- | --- | --- |
| fd | int | 必填 | 要移除的連線者的 fd |

#### 使用範例

這個方法通常在 OpenSwoole 的 close 事件中使用，將已經中段的連線者從 Websocket Client Pool 中移除。

```php
use Monken\CIBurner\OpenSwoole\Worker;

$server->on('close', static function (Server $server, int $fd) {
    Worker::unsetWebsocket($fd);
});
```
