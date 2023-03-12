---
title: OpenSwoole HTTP
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
Events::on('burnerAfterSendResponse',static function(\OpenSwoole\Http\Server $server)
{
    //Your logic
});
```

事件的使用方式不單單只有回呼函數，請參考 [CodeIgniter4 使用手冊](https://codeigniter.tw/user_guide/extending/events.html#id3)提到的方法來選擇最適合你的宣告方式。