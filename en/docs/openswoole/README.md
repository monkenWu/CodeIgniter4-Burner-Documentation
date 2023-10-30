---
title: OpenSwoole
---

# OpenSwoole

OpenSwoole is a new version forked from Swoole, and they have different maintenance teams. When Burner was developed, OpenSwoole had already reached version `22.0.0`, and in this version the namespace has been migrated from `Swoole` to `OpenSwoole`, which means that Burner will not support running under `OpenSwoole` below version `22.0.0`, and it is not compatible with `Swoole`.

## How it works

When receiving an HTTP request, the OpenSwoole driver will convert the OpenSwoole request object to a request object that implements the `PSR-7` interface, and then pass the `PSR-7` request object to `burner-core` for processing. When `burner-core` receives the `PSR-7` request object, it will convert it to a request object that is exclusive to `CodeIgniter4`, and include some pre-processing required by the framework.

When `CodeIgniter4` has finished processing the request, `burner-core` will convert the `CodeIgniter4` response object to a `PSR-7` response object, and then return it to the OpenSwoole driver.

## Getting started

Before using the OpenSwoole driver provided by Burner, you must ensure that your current system environment already has `PHP8^` and the `OpenSwoole` extension. You can refer to [How to install OpenSwoole](https://openswoole.com/docs/get-started/installation), or use our recommended [Docker environment](/general/docker).

Because the current version is a Beta version, you need to add the following settings in `composer.json`:

```json
{
    "minimum-stability": "beta",
    "prefer-stable": true
}
```

After completing these prerequisites, you can execute the following command to obtain the OpenSwoole driver in the project root directory:

```
composer require monken/codeigniter4-burner-openswoole:^1.0@beta
```

{% info Note %}

If you do not have the CodeIgniter4-Burner library installed when you execute the above command, Composer will automatically pull the related dependencies."

{% end %}

Next, you need to use the following command to initialize the Burner and OpenSwoole driver:

```
php spark burner:init OpenSwoole [http or websocket]
```

You can freely choose whether your server uses HTTP or Websocket mode after the command, and the default parameter is `http`. The `http` parameter will generate a general HTTP server configuration file, and if you use the `websocket` parameter, it will generate a WebSocket-specific (including HTTP) configuration file.

When the command is executed successfully, you should be able to see two new files added to the `project_root/app/Config` folder, which are: `Burner.php` and `OpenSwoole.php`, you can adjust these two files to finely control your server settings.

{% info Note %}

If you repeatedly execute the `burner:init` command, the existing configuration file will be renamed instead of being overwritten.

{% end %}

Bruner has already set all the configuration settings, and you can change your configuration file when needed.

When everything is ready, you can start your server directly by executing the following command:

```
php spark burner:start
```

By default, your server should run on port `8080`, and you can immediately browse your project page through `localhost:8080`.

If everything goes well, your CodeIgniter4 should be successfully running on OpenSwoole.
