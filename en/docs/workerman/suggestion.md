---
title: Workerman Suggestion
---

# Workerman Suggestion

## Auto Reload

By default, you must restart the server after each modification to the PHP file, but this doesn't seem very friendly in the development process.

Burner provides a feature for automatic reloading. Just open the `config/workerman.php` file and uncomment one of the items in `$serverWorkers`.

```php
public $serverWorkers= [
    \Monken\CIBurner\Workerman\Worker\CodeIgniter::class,
    //Open this line to enable auto reload
    \Monken\CIBurner\Workerman\Worker\FileMonitor::class,
];
```

You can use `FileMonitor` to monitor your files, and when your files are modified, Burner will automatically restart your server.

You can also modify the monitoring path of `FileMonitor` yourself. Just modify `$autoReloadDir` in `config/workerman.php`.

```php
public $autoReloadDir = '/app/CodeIgniter4-Burner/dev';
```

Burner provides two ways to reload, you can switch by modifying the `autoReloadMode` in `config/workerman.php`.

```php
public $autoReloadMode = 'restart';
```

* `restart` means that the server will automatically restart each time the file is modified. It's like you shut down the server yourself and then restarted it, which can ensure that all php files are reloaded.
* `reload` will only reload the running Worker, just like [the document](https://www.workerman.net/doc/workerman/faq/reload-principle.html) says. Please note that this method may not work in all cases, for example, when the core files of the project generated by `composer require/update` change.

{% info Note %}

Automatic reloading is very resource-intensive, so please don't turn it on in the production environment.

{% end %}

## Debugging with only one Worker

Since Workerman has a fundamental difference with other server software (such as Nginx, Apache), each Codeigniter exists as a Worker in RAM, and HTTP requests will reuse these Workers to handle them. Therefore, we develop and test the stability in an environment with only one Worker to prove that it can also run normally in a formal environment with multiple Workers.

You can refer to the following settings in the `app/Config/Workerman.php` configuration file to reduce the number of CodeIgniter Workers to the minimum:

```php
public $workerCount = 1;
```