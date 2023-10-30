---
title: 開發建議
---

# 開發建議

## Codeigniter4 Request 與 Response 物件

Codeigniter4 並沒有實作完整的 [HTTP message 介面](https://www.php-fig.org/psr/psr-7/)，所以這個程式庫著重於 `PSR-7 介面` 與 `Codeigniter4 HTTP 介面` 的同步。

基於上述原因，在開發上，你應該使用 Codeigniter4 所提供的 `$this->request` 或是使用全域函數 `\Config\Services::('request')` 取得正確的 request 物件；使用 `$this->response` 或是 `\Config\Services::('response')` 取得正確的 response 物件。

請注意，在建構給予使用者的響應時，不論是 `header` 或 `set-cookies` 應該避免使用 PHP 內建的方法進行設定。而是使用 [Codeigniter4 響應物件](https://codeigniter.tw/user_guide/outgoing/response.html) 提供的 `setHeader()` 與 `setCookie()` 進行設定。 

## 覆寫 Codeigniter4 Response 物件

在開發的過程中你可能會於 CodeIgniter4 的控制器中使用到 `response()->send()` 這個方法，你可以於 [說明手冊](https://codeigniter.com/user_guide/outgoing/response.html#CodeIgniter\HTTP\Response::send) 中了解到這個方法的用處。透過這個方法，CodeIgniter 會迅速地輸出 Body 內容、Headers 與 Cookies，並結束程式的執行。顯然，這在 Burner 中是不可行的，Burner 的 PHP 程序必須一直在記憶體中執行以保持最高效能，使用這個方法會導致你的程式出現錯誤。

因此，我們建議你在開發時避免使用這個方法，在控制器中明確地使用 `return` 來結束程式的執行。若是在你原有的專案中已使用了大量的 `response()->send()` 也不用擔心，你可以透過使用 Burner 提供的覆寫方法來解決這個問題。

Burner 採用 CodeIgniter4 官方推薦的[核心擴充方法](https://codeigniter.com/user_guide/extending/core_classes.html#extending-core-classes)，覆寫了 `Response` 類別底下的 `send()`，刪除了所有可能導致伺服器錯誤的程式碼。

請移動至你的專案底下的 `{Project_Root}/app/Config/Services.php` ，並加入以下程式碼：

```php
use Monken\CIBurner\Bridge\Override\Response;

/**
 * Override Response
 *
 * @return ResponseInterface
 */
public static function response(?App $config = null, bool $getShared = true)
{
    if ($getShared) {
        return static::getSharedInstance('response', $config);
    }

    $config ??= config('App');

    return new Response($config);
}
```

## 以 return 結束控制器邏輯

在 Controller 中，盡量使用 return 結束程式邏輯，不論是視圖的響應或是 API 響應，減少使用 `echo` 輸出內容可以避免很多錯誤，就像這個樣子。

```php
<?php

namespace App\Controllers;

use CodeIgniter\API\ResponseTrait;

class Home extends BaseController
{
	use ResponseTrait;

	public function index()
	{
		//Don't use :
		//echo view('welcome_message');
		return view('welcome_message');
	}

	/**
	 * send header
	 */
	public function sendHeader()
	{
		$this->response->setHeader("X-Set-Auth-Token",uniqid());
		return $this->respond(["status"=>true]);
	}

}
```

## 使用內建 Session 程式庫

我們只針對 Codeigniter4 內建 [Session 程式庫](https://codeigniter.tw/user_guide/libraries/sessions.html) 進行支援，並不保證使用 `session_start()` 與 `$_SESSION` 是否能照常運作。所以，你應該避免使用 PHP 內建的 Session 方法，而是以 Codeigniter4 框架內建的程式庫為主。

## 外部連線

不論你使用的驅動程式為何，我們都認為你應該以「單例」的方式管理專案所依賴的外部連線實體，比如：DB, Cache 或是任何需要長連線的外部系統。

這是因為，每個 Worker 都會長時間地存活在記憶體中重複地處理 HTTP 請求，這使的我們降低了 PHP 重複編譯與載入的效能開銷。倘若外部連線實體會在每一個請求開始時重新連線，這將會大大地拖慢執行效能。

Burner 將在伺服器啟動後接管 CodeIgniter4 的生命週期，在長連線管理中，我們只針對 Codeigniter4 內建的 `Database` 程式庫與 `Cache` 程式庫進行直接支援，並不保證 PHP 內建的方法是否能照常運作。

對此，你應該避免使用內建的 PHP 方法，而是以 Codeigniter4 框架內建的程式庫為主。

{% info 備註 %}

當然，若是你十分熟悉你所選擇的驅動程式與你需要長連線的目標系統，也可以自行處理連線，使其能長時間地留存在記憶體中。

{% end %}

### 資料庫與快取程式庫

在 Burner 的預設設定中， Worker 的資料庫與快取的連線是持久的，並會在連線失效時自動重新連線。所有進入 Worker 的請求都將使用同一個連線實體。

如果你不想要這個預設設定，希望每個進入 Worker 的請求都使用重新連線的 DB 或 Cache 連線實體。請調整 Config/Burner.php 中的下列設定：

```php
public $dbAutoClose = true;
public $cacheAutoClose = true;
```

## 檔案上傳處理

Burner 將會把請求中被上傳的檔案資訊重新賦值給 `$_FILES` ，因此 Codeingiter4 的 [檔案上傳類別](https://codeigniter.tw/user_guide/libraries/uploaded_files.html) 可以正常運作。但是，PHP 內建的檔案上傳處理方法，比如：`move_uploaded_file()` 或是 `is_uploaded_file()`，這些方法在持久化的 PHP 伺服器中將無法正常運作。因此，你將無法正常使用 CodeIgniter4 的 [`$file->move()`](https://codeigniter.tw/user_guide/libraries/uploaded_files.html#moving-files) 方法來處理用者上傳的檔案。

{% info 備註 %}
如果你使用的是 `OpenSwoole` 驅動程式，你可以正常使用所有上傳檔案的處理方法。
{% end %}

我們推薦你以下列方式在 Controller 處理檔案上傳：

```php
public function fileUpload()
{
    /**
     * @var \CodeIgniter\HTTP\Files\UploadedFile[]
     */
    $files = $this->request->getFiles();    
    foreach ($files as $file) {
        $newFileName = $file->getRandomName();
        $newFileNamePath = WRITEPATH . 'uploads' . DIRECTORY_SEPARATOR . $newFileName;
        if(BURNER_DRIVER == 'OpenSwoole'){
            // if you use OpenSwoole driver, you can use move()
            if ($file->isValid() && ! $file->hasMoved()) {
                $file->move(WRITEPATH . 'uploads' , $newFileName);
            }
        }else{
            // if you use RoadRunner or Workerman driver, you should use rename(or other method) to move file to new path.
            if (!$file->hasMoved()) {
                rename($file->getTempName(), $newFileNamePath);
            }
        }
    }
}
```

## Service 管理

CodeIgniter4 提供了 [Services](https://codeigniter.tw/user_guide/concepts/services.html) 功能使我們能夠在整個專案中共享物件實體，Burner 將會在伺服器啟動後接管 Service 的生命週期。在每次的請求處理完畢後，Burner 將會自動釋放 Service 的物件實體，並在下一次的請求中重新建立 Service 的物件實體。

若你實作了自己的 Service 物件，或是你認為某些 Service 物件不需要重新建立，你可以在 `app/Config/Burner.php` 中的 `$skipInitServices` 陣列中設定：

```php
public $skipInitServices = [
    // type service name here
    'validation',
];
```

當你將服務名稱加入 `$skipInitServices` 陣列中後，Burner 便不會在每次請求中清除該服務的物件實體。此時，你需要自行處理你所設定的 Service 物件的生命週期。

## Worker 數量

因為 RoadRunner、OpenSwoole 與 Workerman 與其他伺服器軟體（Nginx、Apache）有著根本上的不同，每個 Codeigniter4 將會以 Worker 的形式持久化於記憶體中，HTTP 的請求會重複利用到這些 Worker 進行處裡。所以，我們最好在只有單個 Worker 的情況下開發軟體並測試是否穩定，以證明在多個 Woker 的實際環境中能正常運作。 

## 處理變數拋出

如果你在開發環境中碰到了一些需要確認的變數、或物件內容，無論在程式的何處，你都可以使用全域函數 `dump()` 來將錯誤拋出到終端機上。
