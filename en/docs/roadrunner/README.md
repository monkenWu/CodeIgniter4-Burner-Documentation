---
title: RoadRunner
---

# RoadRunner

[RoadRunner](https://roadrunner.dev/) is a high-performance PHP application server, written in Go. It allows to execute long-running PHP applications in a persistent memory, using a pool of preloaded workers. RoadRunner is also a full-featured HTTP server supporting PSR-7.

## How it works

After receiving an HTTP connection, Burner will convert the PSR-7 request object to a CodeIgniter4 request object, and start executing the CodeIgniter4 application, and then convert the CodeIgniter4 response object to a PSR-7 response object, and then pass it to the RoadRunner server.

Unlike other PHP drivers, RoadRnner natively supports PSR-7, so Burner does not need to convert it again (from other Request objects to PSR-7 Request objects).

## Getting started

Before using RoadRunner, please make sure that your environment already has `PHP8^`, `php-curl`, `php-zip` and `php-sockets` extensions, or you can use our recommended [Docker environment](/general/docker).

After completing these prerequisites, you can run the following command to get the RoadRunner driver in the project root directory:

```
composer require monken/codeigniter4-burner-roadrunner:^1.0@beta
```

{% info Note %}

If you do not have CodeIgniter4-Burner installed when you run the above command, Composer will automatically pull the relevant dependencies.

{% end %}

Next, you need to use the following command to initialize Burner and the RoadRunner driver:

```
php spark burner:init RoadRunner
```

This command will first download the latest RoadRunner executable (i.e. rr or rr.exe), then move it to the `vendor/bin` directory of the project, and finally set the execution permission of `rr` or `rr.exe` to executable.

When the command is executed successfully, you should be able to see the `Burner.php` file in the `project_root/app/Config` folder, and find the `project_root/.rr.yaml` in the project root directory. You can adjust these two files to finely control your server settings.

{% info Note %}

If you run the `burner:init` command repeatedly, the existing configuration file will be renamed instead of being overwritten.

{% end %}

Bruner has already set up all the configuration settings for you, and you can modify your configuration files when needed.

When everything is ready, you can run the following command to start your server:

```
php spark burner:start
```

By default, your server should run on the `8080` port, and you can immediately browse your project page through `localhost:8080`.

If everything goes well, your CodeIgniter4 should be running on RoadRunner successfully.