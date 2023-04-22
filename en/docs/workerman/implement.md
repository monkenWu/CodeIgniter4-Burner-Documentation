---
title: Workerman Implement
---

# Workerman Implement

Workerman provides a basic HTTP server, and the details of its operation need to be implemented by the developer.

For the integration of Workerman and CodeIgniter, we provide a simple implementation that allows developers to get started quickly.

## HTTP Service

For Workerman, CodeIniger is a Worker that has been implemented.

It provides two main functions:

1. As a static file server, respond to static files in the `public` folder.
2. As an HTTP server, convert the request to PSR-7 format and process it with Burner Core.

You can focus on the `src/Worker/CodeIgniter.php` that Burner implements, and the key content is as follows:

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
When an HTTP request comes in, Burner will first check whether it is a static file request. If so, it will respond directly. Otherwise, Burner will convert the request to PSR-7 format and pass it to Burner Core for processing. Finally, Burner will convert the processing result to Workerman format and respond to the client.

## App Container

When a Workerman-based server is started, it can run more than one business logic Worker. You can start multiple Workers according to your needs to run different business logic.

For example, in Burner, we implement two Workers, respectively:

1. `CodeIgniter`: Used to process HTTP requests.
2. `FileMonitor`: Used to monitor file changes.

In `app/Config/Workerman.php`, we can see:

```php
public $serverWorkers = [
    \Monken\CIBurner\Workerman\Worker\CodeIgniter::class,
    // \Monken\CIBurner\Workerman\Worker\FileMonitor::class,
];
```

These two Workers are both inherited from `Monken\CIBurner\Workerman\Worker\WorkerRegistrar`, both implement the `initWorker` method. When you run `php spark burner:start`, Burner will automatically start these two Workers together (according to whether you declare all of them in your configuration).

Burner implements a Workerman-specific `AppContainer`, which contains all the Workerman Worker object instances and provides some methods to operate these instances.

You can focus on Burner's `src/Worker.php`, the key content is as follows:

```php
//Burner Core And Worker Base Settings
\Monken\CIBurner\App::setConfig(config('Burner'));
/** @var \Config\Workerman */
$workermanConfig = Factories::config('Workerman');
Config::staticSetting($workermanConfig);

AppContainer::registerWorkers($workermanConfig->serverWorkers);
AppContainer::init();
```

We will load the Worker needed by the server into `AppContainer`, and initialize these Workers through `AppContainer`. This means that you can add your own Worker to `AppContainer`, and initialize these Workers through `AppContainer`.

### Create your Worker

You can create a file like this in the `app/Workers` folder and name it `MyWorker.php`:

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

Then, you can add your Worker to `serverWorkers` in `app/Config/Workerman.php`:

```php
public $serverWorkers = [
    \Monken\CIBurner\Workerman\Worker\CodeIgniter::class,
    // \Monken\CIBurner\Workerman\Worker\FileMonitor::class,
    \App\Workers\MyWorker::class,
];
```

When you run `php spark burner:start`, your Worker will be automatically started by Burner.

When writing a Worker, you can use any class that complies with the namespace rules. Burner has already taken care of these annoying namespace loads for you.

If you need more notes on writing Workers, or more knowledge about Workerman, we recommend you to refer to the [official documentation](https://github.com/walkor/workerman-manual/tree/master/english/worker-development).

### Available Methods

#### AppContainer::getWorker

Through the `AppContainer::getWorker` method, after passing in the class name, you can get the specified Workerman Worker instance.

#### Parameters

| Parameter | Type | Description |
| --- | --- | --- |
| $workerClass | string | Worker class name |

#### Return Value

| Type | Description |
| --- | --- |
| Workerman\Worker | The specified Workerman Worker instance |
| null | No specified Worker instance was found |

#### Example

For example, the CodeIgniter Worker:

```php
use Monken\CIBurner\Workerman\Worker\CodeIgniter;
$worker = AppContainer::getWorker(CodeIgniter::class);
```

## Burner Events

Burner provides some events during the operation of the server, allowing developers to execute their own logic at a specific stage.

### Config::initWorker

This is usually used to initialize some of the things you need when the CodeIgniter Worker is initialized. You can define the logic you need to execute when the CodeIgniter Worker is initialized through the `initWorker` method in the configuration file `app/Config/Workerman.php`:


```php
use Workerman\Worker;
public function initWorker(Worker &$worker)
{
    //Your logic here
}
```
### Config::runtimeTcpConnection

When a TCP connection occurs in each CodeIgniter Worker, this event will be triggered. You can define the logic you need to execute through the `runtimeTcpConnection` method in the configuration file `app/Config/Workerman.php`:

```php
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
public function runtimeTcpConnection(TcpConnection &$tcpConnection, Request &$request)
{
    //Your logic here
}
```

## Events

CodeIgniter4 provides [events](https://codeigniter.com/user_guide/extending/events.html) that can be used to perform actions at specific points during execution. When CodeIgniter runs, it follows a certain execution order, but in some cases, you may want to perform some actions at a specific stage of execution.

Burner provides the following events for OpenSwoole HTTP:

### burnerAfterSendResponse

This is usually used to initialize after the response, if you have some global variables that need to be cleared or some common processing after the response, it is very suitable to declare it in this event. You can declare the `burnerAfterSendResponse` event, and when OpenSwoole responds, the logic you need to execute will be executed:

```php
// This code will be executed after sending the response to the client.
Events::on('burnerAfterSendResponse',static function(\Workerman\Connection\TcpConnection $connection)
{
    //Your logic
});
```

The way events are used is not limited to callback functions. Please refer to the methods mentioned in the [CodeIgniter4 User Guide](https://codeigniter.com/user_guide/extending/events.html#defining-an-event) to choose the best way for you.
