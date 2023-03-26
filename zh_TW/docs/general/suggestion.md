---
title: 開發建議
---

# 開發建議

## Codeigniter4 Request 與 Response 物件

Codeigniter4 並沒有實作完整的 [HTTP message 介面](https://www.php-fig.org/psr/psr-7/)，所以這個程式庫著重於 `PSR-7 介面` 與 `Codeigniter4 HTTP 介面` 的同步。

基於上述原因，在開發上，你應該使用 Codeigniter4 所提供的 `$this->request` 或是使用全域函數 `\Config\Services::('request')` 取得正確的 request 物件；使用 `$this->response` 或是 `\Config\Services::('response')` 取得正確的 response 物件。

請注意，在建構給予使用者的響應時，不論是 `header` 或 `set-cookies` 應該避免使用 PHP 內建的方法進行設定。而是使用 [Codeigniter4 響應物件](https://codeigniter.tw/user_guide/outgoing/response.html) 提供的 `setHeader()` 與 `setCookie()` 進行設定。 

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

無論你使用哪種驅動程式，都有著無法正確傳遞 `$_FILES` 內容的缺陷。所以 Codeingiter4 的 [檔案上傳類別](https://codeigniter.tw/user_guide/libraries/uploaded_files.html) 將無法正確運作。對此，我們提供了符合 PSR-7 規範的檔案上傳類別，讓你可以正確地在 RoadRunner 中處理檔案上傳。就算你將專案切換到了其他伺服器環境（spark serve、Apache、Nginx）執行，這個類別依舊可以正常使用，並且不需要修改任何程式碼。

你可以在控制器（或任何地方），使用下述程式碼片段取得使用者上傳的檔案。

```php
SDPMlab\Ci4Roadrunner\UploadedFileBridge::getPsr7UploadedFiles()
```

這個方法將回傳以 Uploaded File 物件組成的陣列。此物件可用的方法與 [PSR-7 Uploaded File Interface](https://www.php-fig.org/psr/psr-7/#36-psrhttpmessageuploadedfileinterface) 中規範的一樣。

以下範例將展示一個標準的 CodeIgniter4 控制器處理單一檔案上傳以及多重檔案上傳的最簡方式。

```php
<?php

namespace App\Controllers;

use CodeIgniter\API\ResponseTrait;
use SDPMlab\Ci4Roadrunner\UploadedFileBridge;

class FileUploadTest extends BaseController
{
    use ResponseTrait;

    protected $format = "json";

    /**
     * form-data 
     */
    public function fileUpload()
    {
        $files = UploadedFileBridge::getPsr7UploadedFiles();
        $data = [];
        foreach ($files as $file) {
            $fileNameArr = explode('.', $file->getClientFilename());
            $fileEx = array_pop($fileNameArr);
            $newFileName = uniqid(rand()) . "." . $fileEx;
            $newFilePath = WRITEPATH . 'uploads' . DIRECTORY_SEPARATOR . $newFileName;
            $file->moveTo($newFilePath);
            $data[$file->getClientFilename()] = md5_file($newFilePath);
        }
        return $this->respondCreated($data);
    }

    /**
     * form-data multiple upload
     */
    public function fileMultipleUpload()
    {
        $files = UploadedFileBridge::getPsr7UploadedFiles()["data"];
        $data = [];
        foreach ($files as $file) {
            $fileNameArr = explode('.', $file->getClientFilename());
            $fileEx = array_pop($fileNameArr);
            $newFileName = uniqid(rand()) . "." . $fileEx;
            $newFilePath = WRITEPATH . 'uploads' . DIRECTORY_SEPARATOR . $newFileName;
            $file->moveTo($newFilePath);
            $data[$file->getClientFilename()] = md5_file($newFilePath);
        }
        return $this->respondCreated($data);
    }
}
```

## Worker 數量

因為 RoadRunner、OpenSwoole 與 Workerman 與其他伺服器軟體（Nginx、Apache）有著根本上的不同，每個 Codeigniter4 將會以 Worker 的形式持久化於記憶體中，HTTP 的請求會重複利用到這些 Worker 進行處裡。所以，我們最好在只有單個 Worker 的情況下開發軟體並測試是否穩定，以證明在多個 Woker 的實際環境中能正常運作。 

## 處理變數拋出

如果你在開發環境中碰到了一些需要確認的變數、或物件內容，無論在程式的何處，你都可以使用全域函數 `dump()` 來將錯誤拋出到終端機上。
