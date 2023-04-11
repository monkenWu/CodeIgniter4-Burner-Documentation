---
title: OpenSwoole Suggestion
---

# OpenSwoole Suggestion

## Fast Cache

OpenSwoole provides a fast Cache system that can share data between multiple Workers and synchronize data between multiple Workers. We support [Swoole-Table](https://openswoole.com/docs/modules/swoole-table) as the storage interface for caching, and you can use it in your project, just like you are familiar with it!

You can add the following settings to the `app/Config/OpenSwoole.php` configuration file to turn on the fast cache function:

```php
/**
 * Whether to open the Key/Value Cache provided by Burner.
 * Shared high-speed caching with Swoole-Table implementation.
 * 
 * @var boolean
 */
public $fastCache = true;

/**
 * Buerner Swoole-Table Driver Settings
 * 
 * @var string[]
 */
public $fastCacheConfig = [
    //Number of rows of the burner cache table
    'tableSize' => 4096,   
    //Key/value key Maximum length of string
    'keyLength' => 1024,
    //The maximum length of the key/value (if the save type is object, array, string).
    'valueStringLength' => 1024
];
```

### Using CodeIgniter4 Cache

CodeIgniter provides a singleton Cache management method, and you can use the `cache()` function anywhere in the program to get the Cache instance. Burner also provides a `Burner` Cache Handler.

You need to open `app/Config/Cache` and add the following settings:

```php
public $validHandlers = [
    //hide
    'burner' => \Monken\CIBurner\BurnerCacheHandler::class
];
```

Finally, you need to switch from `Primary Handler` to `Burner`.

```php
public $handler = 'burner';
```

Now you can operate the [Cache Library](https://www.codeigniter.com/user_guide/libraries/caching.html) in the same way as the Swoole-Table Cache provided by Burner. Please note that Swoole-Table will lose all data due to server restart, and it is only a solution for sharing data between Workers.

### Get Cache instance directly

If your project has already used a Cache system similar to Redis, and has used the CodeIgniter4 Cache management mechanism, you can also directly use the global method provided by Burner to get the Cache instance, so you don't have to worry about the problem that Cache can only have one Handler.

Just set `fastCache` to `true` in `app/Config/OpenSwoole.php` and use the following in your program to get the Cache entity:

```php
$fastCache = \Monken\CIBurner\OpenSwoole\Worker::getFastCache();
```

## Automatic reload

By default, in the OpenSwoole state, you must restart the server after each modification to the PHP file to make your changes effective. This does not seem very friendly.

You can modify the `app/Config/OpenSwoole.php` configuration file and add the following settings and restart the server.

```php
/**
 * Auto-scan changed files
 *
 * @var bool
 */
public $autoReload = true;

/**
 * Auto Reload Mode
 *
 * @var string restart or reload
 */
public $autoReloadMode = 'restart';
```

Burner provides two ways to reload, you can switch by adjusting `autoReloadMode`.

* `restart` means that the server will automatically restart every time the file is changed. Just like you turn off the server yourself and then restart it, this can ensure that all php files are reloaded.
* `reload` will only reload the running Worker, just like the [document](https://openswoole.com/docs/modules/swoole-server-reload#hot-code-linux-signal-trigger) says. Please note that this mode may not handle all cases. For example: When you make changes to the core PHP files of your project through `composer require/update`.

{% info Note %}

`Automatic reload` method is very resource-intensive, please do not enable this option in the production environment.

{% end %}

## Debugging with only one Worker

Since OpenSwoole is fundamentally different from other server software (such as Nginx, Apache), each Codeigniter4 exists as a Worker in RAM, and HTTP requests reuse these Workers to handle them. Therefore, we develop and test the stability in the environment with only one Worker to prove that it can also run normally in the formal environment with multiple Workers.

You can refer to the following settings in the `app/Config/OpenSwoole.php` configuration file and reduce the number of Workers to the minimum:

```php
public $config = [
    'worker_num' => 1,
    /** hide */
];
```
