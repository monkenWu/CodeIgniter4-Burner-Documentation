---
title: Development Suggestion
---

# Development Suggestion

## Using Codeigniter4 Request and Response object

Codeigniter4 does not implement the complete [HTTP message interface](https://www.php-fig.org/psr/psr-7/), hence this library focuses on the synchronize of `PSR-7 interface` and `Codeigniter4 HTTP interface`.

Base on the reasons above, You should use `$this->request`, provided by Codeigniter4, or the global function `/Config/Services::('request')` to fetch the correct request object; Use `$this->response` or `/Config/Services::('response')` to fetch the correct response object.

Please be noticed, while constructing response for the users during developing, you should prevent using PHP built-in methods to conduct `header` or `set-cookies` settings. Using the `setHeader()` and `setCookie()`, provided by the [Codeigniter4 Response Object](https://codeigniter.com/user_guide/outgoing/response.html), to conduct setting.

## Override Codeigniter4 Response object

During the development, you may use `response()->send()` in CodeIgniter4's Controller, you can learn about this method in the [documentation](https://codeigniter.com/user_guide/outgoing/response.html#CodeIgniter\HTTP\Response::send). Through this method, CodeIgniter will quickly output the Body content, Headers and Cookies, and end the execution of the program. Obviously, this is not feasible in Burner, the PHP program of Burner must be executed in memory to maintain the highest efficiency. Using this method will cause your program to fail.

Therefore, we recommend that you avoid using this method during development, and use `return` to end the execution of the program in the Controller. If you have used a lot of `response()->send()` in your original project, don't worry, you can use the override method provided by Burner to solve this problem.

Burner uses the [Extension Core Method](https://codeigniter.com/user_guide/extending/core_classes.html#extending-core-classes) recommended by CodeIgniter4, and overrides `send()` under the `Response` class, deleting all code that may cause server errors.

Please move to `{Project_Root}/app/Config/Services.php` under your project, and add the following code:

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

Burner will reassign the uploaded file information in the request to `$_FILES`, so the [File Upload Class](https://codeigniter.com/user_guide/libraries/uploaded_files.html) of Codeingiter4 can work properly. However, the built-in PHP file upload processing methods, such as: `move_uploaded_file()` or `is_uploaded_file()`, these methods will not work properly in the persistent PHP server. Therefore, you will not be able to use CodeIgniter4's [`$file->move()`](https://codeigniter.com/user_guide/libraries/uploaded_files.html#moving-files) method to process the files uploaded by the user.

{% info Note %}
If you are using the `OpenSwoole` driver, you can use all the file upload processing methods normally.
{% end %}

We recommend that you handle file uploads in the Controller in the following ways:

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
```

## Service Management

CodeIgniter4 provides [Services](https://codeigniter.com/user_guide/concepts/services.html) to allow us to share object instances throughout the project. Burner will take over the life cycle of the Service after the server starts. After each request is processed, Burner will automatically release the object instance of the Service and re-establish the object instance of the Service in the next request.

If you have implemented your own Service object, or you think that some Service objects do not need to be re-established, you can set it in the `$skipInitServices` array in `app/Config/Burner.php`:

```php
public $skipInitServices = [
    // type service name here
    'validation',
];
```

After you add the service name to the `$skipInitServices` array, Burner will not clear the object instance of the service on each request. At this time, you need to handle the life cycle of the Service object you set yourself.

## Worker-num

Because RoadRunner, OpenSwoole and Workerman are fundamentally different from other server software (Nginx, Apache), each Codeigniter4 will be persisted in memory in the form of a Worker, and the HTTP request will be repeatedly reused to handle these Workers. So, we should develop and test software in a single Worker environment to prove that it can work properly in a real environment with multiple Wokers.

## Handle variable dump

If you encountered some variables or object content that needed to be confirmed in `-d` development mode, you can use the global function `dump()` to throw errors onto the terminal no matter where the program is.
