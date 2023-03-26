---
title: OpenSwoole suggestions
---

# OpenSwoole suggestions

## Fast Cache

OpenSwoole provides a fast cache system that can be shared between multiple workers and synchronized between multiple workers. We support [Swoole-Table](https://openswoole.com/docs/modules/swoole-table) as the storage interface for the cache, which you can use in your project just like you are familiar with!

You can add the following settings in the `app/Config/OpenSwoole.php` configuration file and restart the server.

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

Add the following settings to the `app/Config/Cache.php` configuration.

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

Now you can operate the [Cache Library](https://www.codeigniter.com/user_guide/libraries/caching.html) in the same way as the Burner provided Swoole-Table Cache. Please note that Swoole-Table will lose all data due to server restart, it is just a solution to share data between workers.

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
