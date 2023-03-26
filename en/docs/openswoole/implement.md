---
title: OpenSwoole Implement
---

# OpenSwoole HTTP

Burner provides a HTTP server that is similar to infrastructure-level support. If you choose OpenSwoole-HTTP as your server solution, you should not be able to notice the existence of Burner in your development.

You can see the following code snippet in the `server` method in `project_root/app/Config/OpenSwoole.php`:

```php
$server->on('request', static function (Request $swooleRequest, Response $swooleResponse) {
    // Burner handles CodeIgniter4 entry points.
    Worker::httpProcesser($swooleRequest, $swooleResponse);
});
```

All HTTP requests are handled from `Worker::httpProcesser`, and when this static method is called, it will convert the OpenSwoole request object to a request object that implements the `PSR-7` interface, and then pass the `PSR-7` request object to `burner-core` for processing. When `burner-core` receives the `PSR-7` request object, it will convert it to a request object that is exclusive to `CodeIgniter4`, and include some pre-processing required by the framework.

When `CodeIgniter4` has finished processing the request, `burner-core` will convert the `CodeIgniter4` response object to a `PSR-7` response object, and then return it to the OpenSwoole driver.

## Events

CodeIgniter4 provides [events](https://codeigniter.com/user_guide/extending/events.html) that can be used to perform actions at specific points during execution. When CodeIgniter runs, it follows a certain execution order, but in some cases, you may want to perform some actions at a specific stage of execution.

Burner provides the following events for OpenSwoole HTTP:

### burnerAfterSendResponse

This is usually used to initialize after the response, if you have some global variables that need to be cleared or some common processing after the response, it is very suitable to declare it in this event. You can declare the `burnerAfterSendResponse` event, and when OpenSwoole responds, the logic you need to execute will be executed:

```php
Events::on('burnerAfterSendResponse',static function(\OpenSwoole\Http\Server $server)
{
    //Your logic
});
```

The way events are used is not limited to callback functions. Please refer to the methods mentioned in the [CodeIgniter4 User Guide](https://codeigniter.com/user_guide/extending/events.html#defining-an-event) to choose the best way for you.

# OpenSwoole Websocket

Burner provides a WebSocket server integration for OpenSwoole WebSocket Server. In development, you must follow some usage rules to deeply integrate WebSocket into your CodeIgniter4 project.

First, you need to use the `php spark burner:init OpenSwoole websocket` command to get the WebSocket-specific configuration file `project_root/app/Config/OpenSwoole.php`, you should see the following member method change (different from the HTTP configuration).

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
            // Not Found Handling
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

In the above code snippet, you can see the `on('open')` event, which will be triggered when the WebSocket connection is established. You can perform some initialization actions in this event, such as verifying the identity of the connecting user or rejecting the user's connection. You can notice that the `Worker::setWebsocket` method in this event will store the request information of the connecting user to the Websocket Pool provided by Burner. This is a necessary action because Burner must use this request information to trigger the processing of CodeIgniter4.

Next, you can see the `on('message')` event, which will be triggered when the WebSocket receives a message. You can notice that the `Worker::websocketProcesser` method in this event will pass the received message to CodeIgniter4 for processing, and then process the request after CodeIgniter4 is completed. If you have some special business logic that needs to be processed before CodeIgniter4 processes the request, you only need to remember to call `Worker::websocketProcesser` to process your request after everything is done.

Finally, you can see the `on('close')` event, which will be triggered when the WebSocket connection is closed. You can notice that the `Worker::unsetWebsocket` method in this event will remove the request information of the connecting user from the Websocket Pool provided by Burner.

In addition, WebSocket can also handle normal HTTP requests through the `on('request')` event. You can use the WebSocket server as a normal HTTP server.

## Push Message

### Simple Implementation

Burner provides the `Worker::push` method to push messages to WebSocket connections. No matter where you are, you can push messages to users through the following method:

```php
use Monken\CIBurner\OpenSwoole\Worker;
Worker::push(
    data: 'hi! It\'s Burner Websocket!',
    fd: Worker::getFrame()->fd
);
```

We recommend that you use the `Worker::push` method in the CodeIgniter4 controller to push messages, just like this:

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

This Controller may be accessed through the following rote:
    
```php
$routes->get('burner/websocket', 'BurnerWebsocket::connect');
```

After the user shakes hands with Burner via Websocket and `burner/websocket`, Burner then passes the connection to CodeIgniter4 so that you can work on it in CodeIgniter4. Here, you can use CodeIgniter4's features to your heart's content to work with your business logic.

## Available Methods

### Worker::push

Push messages to WebSocket connections.

#### Parameters

| Parameter | Type | Description |
| --- | --- | --- |
| data | string or binary | The message to push |
| fd | int | The fd of the connection to push |
| opcode | int | The type of data to pass. OpenSwoole's `WEBSOCKET_OPCODE_TEXT`, `WEBSOCKET_OPCODE_BINARY` or `WEBSOCKET_OPCODE_PING` |

#### Example

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

Push messages to all WebSocket connections, or enumerate some connection fd to push messages.

#### Parameters

| Parameter | Type | Description |
| --- | --- | --- |
| data | string or callable | The message to push, or a callback function |
| fds | array | The fd of the connection to push, if not passed in, it will be all the current clients connected to the server |
| opcode | int | The type of data to pass. OpenSwoole's `WEBSOCKET_OPCODE_TEXT`, `WEBSOCKET_OPCODE_BINARY` or `WEBSOCKET_OPCODE_PING` |

#### Push with String

Pushes the string to all connections.

```php
use Monken\CIBurner\OpenSwoole\Worker;
use OpenSwoole\WebSocket\Server;

Worker::pushAll(
    data: 'hi! It\'s Burner Websocket!',
    fds: [1, 2, 3],
    opcode: Server::WEBSOCKET_OPCODE_TEXT
);
```

#### Push with Callback

Push messages with Callback to include more processing logic.

```php
use Monken\CIBurner\OpenSwoole\Worker;
use OpenSwoole\WebSocket\Server;

Worker::pushAll(
    data: static function (int $fd) {
        // $fd is the fd of the connection to push
        // You can do some special processing here, and then return the message to push
        return sprintf('hi! It\'s Burner Websocket! fd: %d', $fd);
    },
    fds: [1, 2, 3],
    opcode: Server::WEBSOCKET_OPCODE_TEXT
);
```

#### Push with Callback and more setting

You can also return an array to set the `opcode` of the message to push.

```php
use Monken\CIBurner\OpenSwoole\Worker;
use OpenSwoole\WebSocket\Server;

Worker::pushAll(
    data: static function (int $fd) {
        // $fd is the fd of the connection to push
        // You can do some special processing here, and then return the message to push
        return [
            // The message to push
            'message' => sprintf('hi! It\'s Burner Websocket! fd: %d', $fd),
            // The type of data to pass
            'opcode' => Server::WEBSOCKET_OPCODE_BINARY 
        ];
    },
    fds: [1, 2, 3]
);
```

### Worker::getFrame

You can get the current WebSocket Frame at any where in your program.

#### Example

```php

use Monken\CIBurner\OpenSwoole\Worker;

$frame = Worker::getFrame();
```

The type of `$frame` is `OpenSwoole\WebSocket\Frame`, you can refer to the [OpenSwoole documentation](https://openswoole.com/docs/modules/swoole-websocket-frame) to learn more.

## Events

### burnerAfterPushMessage

When Burner pushes a message to a WebSocket connection, the `burnerAfterPushMessage` event will be triggered, and you can do some post-processing in this event.

You can register the `burnerAfterPushMessage` event as follows:

```php

Events::on('burnerAfterSendResponse',static function(\OpenSwoole\Http\Server $server, int $fd, bool $pushResult)
{
    // $server is the OpenSwoole Server instance
    // $fd is the fd of the current connection
    // $pushResult is the result of pushing the message
});

```

### burnerAfterPushAllMessage

When Burner pushes a message to all WebSocket connections, the `burnerAfterPushAllMessage` event will be triggered, and you can do some post-processing in this event.

You can register the `burnerAfterPushAllMessage` event as follows:

```php

Events::on('burnerAfterPushAllMessage',static function(\OpenSwoole\Http\Server $server, array $fds)
{
    // $server is the OpenSwoole Server instance
    // $fds is the fd of the current connection array
});

```
