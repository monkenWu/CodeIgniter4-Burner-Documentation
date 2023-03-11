---
title: 檔案上傳處理
---

# 檔案上傳處理

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

