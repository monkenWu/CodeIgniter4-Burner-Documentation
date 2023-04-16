---
title: RoadRunner 實踐
---

# RoadRunner HTTP

RoadRunner 是一個十分完整的 PHP 伺服器，我們幾乎不用特別的撰寫 Worker 的細部邏輯就能夠將它與 Burner-Core 整合起來。你若選用了 RoadRunner 作為你的伺服器解決方案，在開發上你應該很難察覺它的存在。

在 Burner 所實作的 worker.php 中，關鍵的部份如下：

```php
while (true) {
    // get psr7 request
    try {
        $request = $psr7->waitRequest();
        if (! ($request instanceof RequestInterface)) { // Termination request received
            break;
        }
    } catch (Exception $e) {
        $psr7->respond(new Response(400)); // Bad Request

        continue;
    }

    /** @var \Psr\Http\Message\ResponseInterface */
    $response = \Monken\CIBurner\App::run($request);

    // handle response object
    try {
        $psr7->respond($response);
        Events::trigger('burnerAfterSendResponse', $psr7);
        \Monken\CIBurner\App::clean();
    } catch (Exception $e) {
        $psr7->respond(new Response(500, [], 'Something Went Wrong!'));
    }
}
```

所有的 HTTP 請求從 `waitRequest` 開始，當這個方法獲取到最新的使用者請求後，就會立即執行 `App::run()` 方法，Burner 便會開始一系列的預處理，並通知 CodeIgniter4 處理你的連線。

再接下來，就會進入 CodeIgniter4 的處理流程，當 CodeIgniter4 處理完成後， `run()` 方法就會回傳 PSR-7 的響應物件，轉交給 `respond` 方法來處理。

了解了上述流程後，你會發現 RoadRunner 的實作邏輯並不複雜，如同你所熟知的那樣，一如往常地開發你的應用程式即可。

## 事件

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
事件的使用方式不單單只有回呼函數，請參考 [CodeIgniter4 使用手冊](https://codeigniter.tw/user_guide/extending/events.html#id3)提到的方法來選擇最適合你的宣告方式。

## 組態設定

由 Burner 自動產生的組態設定檔案將位於專案根目錄下的 `rr.yml`，它的內容可能會如下：

```yml
version: "2.7"

rpc:
  listen: tcp://127.0.0.1:6001

server:
  command: "php Worker.php -f=/app/CodeIgniter4-Burner/src/FrontLoader.php -a=/app/CodeIgniter4-Burner/dev/app/"
  # env:
  #   XDEBUG_SESSION: 1

http:
  address: "0.0.0.0:8080"
  static:
    dir: "/app/CodeIgniter4-Burner/dev/public"
    forbid: [".htaccess", ".php"]
  pool:
    num_workers: 1
    # max_jobs: 64
    # debug: true

# reload:
#   interval: 1s
#   patterns: [ ".php" ]
#   services:
#     http:
#       recursive: true
#       ignore: [ "vendor" ]
#       patterns: [ ".php", ".go", ".dmd" ]
#       dirs: [ "/app/CodeIgniter4-Burner/dev" ]

# logs:
#   mode: development
#   output: stdout
#   file_logger_options:
#     log_output: "/app/CodeIgniter4-Burner/dev/writable/logs/RoadRunner.log"
#     max_size: 100
#     max_age: 1
#     max_backups : 5
#     compress: false
```

你可能會發現在 `server.command` 的設定中，我們使用了 `php Worker.php -f -a` 來啟動 RoadRunner 的 Worker，這是因為我們需要在 Worker 中載入 CodeIgniter4 的核心，而這個核心需要 Burner 所提供的 `FrontLoader` 來處理。同時， `app` 資料夾的定位對於 CodeIgniter4 也是一件十分重要的事情，所以我們需要在啟動 Worker 時將這兩個參數傳入，而這兩個參數將會依賴 Burner 的初始化指令自動產生。

你可以參考官方的[完整版文件](https://github.com/roadrunner-server/roadrunner/blob/v2.12.3/.rr.yaml
)，與[官方使用手冊](https://roadrunner.dev/docs)依照自己的需求修改這個檔案。