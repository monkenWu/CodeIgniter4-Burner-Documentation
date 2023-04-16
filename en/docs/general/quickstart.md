---
title: Quickstart
---

# Quickstart

Burner currently only supports installation via Composer, this is because Burner provides a consistent interface and automatic loading for different drivers, which makes it difficult for developers to deploy Burner on their own.

## Install Burner

First, install Burner into your CodeIgniter4 project using Composer.

Choose a suitable driver to install, and Burner currently provides you with three options:

1. 
    ```
    composer require monkenwu/codeigniter4-burner-openswoole:1.0.0-beta.1
    ```
2. 
    ```
    composer require monkenwu/codeigniter4-burner-workerman
    ```
3. 
    ```
    composer require monkenwu/codeigniter4-burner-roadrunner:1.0.0-beta.1
    ```

Before installation, you must first make sure that the current system environment meets the requirements of these drivers. You can refer to the content mentioned in [System Requirements](/introduction) to build your system environment, and we also recommend you to [build your development environment through Docker](/general/docker).

{% info Note %}

If you need, you can also install multiple drivers for your project, and choose the driver you want when you start. You can refer to the "Commands" entry of each driver to learn how to select the driver you want when you execute the server startup command.

{% end %}

## Initialization

Depending on the driver you choose, the `burner:init` command also needs to pass different parameters.

```
php spark burner:init [OpenSwoole, Workerman, RoadRunner]
```

After execution, you will find that two files, `Burner.php` and a file named by the driver name, for example: `OpenSwoole.php`, are added to the `app/Config` directory.

Burner is a library that is ready to use out of the box. All configuration settings are preconfigured. You can start the server directly until you need to adjust the content of the configuration settings.

## Start the server

Now, you can start your server directly through the following command.

```
php spark burner:start
```

By default, Burner sets the `port` that the server listens to to `8080`. Now open your browser and browse `localhost:8080`, you will see the Home page of the current project.

## Conclusion

Burner is simple and lightweight.

We hope it can become the best fuel for your CodeIgnter4 project.

Before development, we recommend that you continue to read [Development Suggestions](/general/suggestion)
