---
title: Workerman
---

# Workerman

[Workerman](https://github.com/walkor/workerman/) is a high-performance application container developed by native PHP. Unlike OpenSwoole, which relies on special external PHP extensions, or needs an external Go Server like RoadRunner, it is a completely PHP-implemented event-driven high-performance framework.

## How it works

After receiving an HTTP connection, Burner will convert the Workerman Request object to a PSR-7 request object, then through Burner-Core to convert it to a CodeIgniter4 request object, and then start executing the CodeIgniter4 application.

After CodeIgniter4 has processed the request, Burner-Core will convert the CodeIgniter4 Request object to a PSR-7 response object, and then pass it to the Workerman driver for processing, and finally respond with the Workerman Response object.

## Getting started

Before using Workerman, please make sure that your environment already has `PHP8^`, `php-pcntl`, `php-posix`, and non-essential but recommended [`php-event`](https://www.php.net/manual/en/book.event.php) extensions, or you can use our recommended [Docker environment](/general/docker).

After completing these prerequisites, you can execute the following command in the project root directory to obtain the Workerman driver:

```
composer require monken/codeigniter4-burner-workerman:^1.0@beta
```

{% info Note %}

If you do not have CodeIgniter4-Burner installed when you run the above command, Composer will automatically pull the relevant dependencies.

{% end %}
    
Next, you need to use the following command to initialize Burner and the Workerman driver:

```
php spark burner:init Workerman
```

When the command is executed successfully, you should be able to see two files added to the `project_root/app/Config` folder, namely: `Burner.php` and `Workerman.php`, you can adjust these two files to finely control your server settings.

{% info Note %}

If you run the `burner:init` command repeatedly, the existing configuration file will be renamed instead of being overwritten directly.

{% end %}

Bruner has already pre-configured all the settings for you, you can modify your configuration files when needed.

When everything is ready, you can run the following command to start your server:

```
php spark burner:start
```

By default, your server should run on the `8080` port, you can immediately browse your project page through `localhost:8080`.

If everything goes well, your CodeIgniter4 should be running on Workerman successfully.