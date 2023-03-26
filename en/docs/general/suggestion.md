---
title: Development Suggestion
---

# Development Suggestion

## Using Codeigniter4 Request and Response object

Codeigniter4 does not implement the complete [HTTP message interface](https://www.php-fig.org/psr/psr-7/), hence this library focuses on the synchronize of `PSR-7 interface` and `Codeigniter4 HTTP interface`.

Base on the reasons above, You should use `$this->request`, provided by Codeigniter4, or the global function `/Config/Services::('request')` to fetch the correct request object; Use `$this->response` or `/Config/Services::('response')` to fetch the correct response object.

Please be noticed, while constructing response for the users during developing, you should prevent using PHP built-in methods to conduct `header` or `set-cookies` settings. Using the `setHeader()` and `setCookie()`, provided by the [Codeigniter4 Response Object](https://codeigniter.com/user_guide/outgoing/response.html), to conduct setting.

## Use return to stop controller logic

Inside the Controller, try using return to stop the controller logic. No matter the response of view or API, reduce the `echo` output usage can avoid lets of errors, just like this:

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

## Use the built-in Session library

We only focus on supporting the Codeigniter4 built-in [Session library](https://codeigniter.com/user_guide/libraries/sessions.html), and do not guarantee if using `session_start()` and `$_SEEEION` can work as normal. So, you should avoid using the PHP built-in Session method, change to the Codeigniter4 framework built-in library.

## External Connections

No matter what driver you are using, we believe that you should manage the external connections in a singleton way, such as: DB, Cache or any other external systems that need long connections.

This is because each Worker will live in memory for a long time to repeatedly process HTTP requests, which makes us reduce the performance overhead of PHP repeated compilation and loading. If the external connection instance will reconnect at the beginning of each request, this will greatly slow down the execution performance.

Burner will take over the life cycle of CodeIgniter4 after the server starts, and in the long connection management, we only directly support the Codeigniter4 built-in `Database` library and `Cache` library, and do not guarantee that the built-in PHP method will work as normal.

For this, you should avoid using the built-in PHP method, and use the Codeigniter4 framework built-in library as the main.

{% info Note %}

Of course, if you are very familiar with the driver you choose and the target system you need to connect to, you can also handle the connection yourself to keep it in memory for a long time.

{% end %}

### Database and Cache Library

In the default settings of Burner, the connection of the Worker's database and cache is persistent, and will automatically reconnect when the connection is invalid. All requests that enter the Worker will use the same connection instance.

If you do not want this default setting, and want each request that enters the Worker to use a DB or Cache connection instance that reconnects. Please adjust the following settings in Config/Burner.php:

```php
public $dbAutoClose = true;
public $cacheAutoClose = true;
```

## File Upload

No matter which driver you are using, there is a defect that cannot correctly pass the contents of `$_FILES`. So the [File Upload Class](https://codeigniter.com/user_guide/libraries/uploaded_files.html) of Codeingiter4 will not work properly. For this, we provide a file upload class that meets the PSR-7 specification so that you can correctly handle file uploads in the heigh performance PHP server. Even if you switch your project to another server environment (spark serve, Apache, Nginx) to run, this class can still be used normally and does not require any code changes.

You can use the following code snippet to get the files uploaded by the user in the controller (or anywhere).

```php
SDPMlab\Ci4Roadrunner\UploadedFileBridge::getPsr7UploadedFiles()
```

This method will return an array of Uploaded File objects. The methods available on this object are the same as those specified in the [PSR-7 Uploaded File Interface](https://www.php-fig.org/psr/psr-7/#36-psrhttpmessageuploadedfileinterface).

The following example will show the minimal way a standard CodeIgniter4 controller handles single file uploads as well as multiple file uploads.

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

## Worker-num

Because RoadRunner, OpenSwoole and Workerman are fundamentally different from other server software (Nginx, Apache), each Codeigniter4 will be persisted in memory in the form of a Worker, and the HTTP request will be repeatedly reused to handle these Workers. So, we should develop and test software in a single Worker environment to prove that it can work properly in a real environment with multiple Wokers.

## Handle variable dump

If you encountered some variables or object content that needed to be confirmed in `-d` development mode, you can use the global function `dump()` to throw errors onto the terminal no matter where the program is.
