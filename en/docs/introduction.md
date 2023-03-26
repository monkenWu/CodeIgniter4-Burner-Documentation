---
title: Welcome to Burner!
---

# Welcome to Burner!

Burner is a out-of-box  library for CodeIgniter4 that supports [RoadRunner](https://roadrunner.dev/) , [Workerman](https://github.com/walkor/workerman) ，and [OpenSwoole](https://openswoole.com/) . You only need to open some php extensions to greatly accelerate your CodeIgniter4 application, making it able to withstand higher loads and handle more connections at the same time.

## System Requirements

The system requirements for Burner are different depending on the driver you choose.

### Basic Requirements

1. CodeIgniter Framework 4.3.0 or later
2. Composer
3. PHP8^

### OpenSwoole

1. OpenSwoole 22^ or later, you can refer to the official document [OpenSwoole Prerequisites](https://openswoole.com/docs/get-started/prerequisites)

We also recommend you to read: [How to install OpenSwoole](https://openswoole.com/docs/get-started/installation)

{% info Note %}

OpenSwoole does not support native execution in the Windows environment. However, you can use OpenSwoole via WSL or Docker.

{% end %}

### Workerman

1. Enable `php-pcntl` extension
2. Enable `php-posix` extension
3. Not required, but we recommend you to install [php-event](https://www.php.net/manual/en/book.event.php) extension to get better performance

{% info Note %}

Workerman does not support Windows environment.But you can use Workerman via WSL or Docker.

{% end %}

### RoadRunner

1. Enable `php-curl` extension
6. Enable `php-zip` extension
5. Enable `php-sockets` extension

## Architecture

Burner is composed of several libraries, including:

* [`burner-core`](https://github.com/monkenWu/CodeIgniter4-Burner) is a simple `PSR7` and `CodeIgniter4` converter, and also includes some CodeIgniter4 commands; because Codeigniter4 does not implement the full [HTTP message interface](https://www.php-fig.org/psr/psr-7/), this library focuses on the synchronization of the `PSR-7` interface and the Codeigniter4 HTTP interface.

* [`burner-OpenSwoole`](https://github.com/monkenWu/CodeIgniter4-Burner-OpenSwoole) uses this driver, your CodeIngiter4 will be able to run through OpenSwoole

* [`burner-Workerman`](https://github.com/monkenWu/CodeIgniter4-Burner-Workerman) uses this driver, your CodeIngiter4 will be able to run through Workerman. In some [test scenarios](https://www.techempower.com/benchmarks/#section=data-r21&hw=ph&test=query&l=zik073-sf&a=2), Workerman has shown excellent performance advantages.

* [`burner-RoadRunner`](https://github.com/monkenWu/CodeIgniter4-Burner-RoadRunner) uses this driver, your CodeIngiter4 will be able to run through RoadRunner. Unlike other Drivers, it has a PHP server implemented in `Go`, in addition to the framework's own support, the official also provides many convenient plug-ins.

You can choose the most suitable driver for your project, and there is no absolute good or bad. It depends on the project requirements and the familiarity of the developer with the technology.

With the support of Burner, your project will be able to freely switch between different high-performance PHP servers, or run on two different servers at the same time on different ports. For example, through RoadRunner to handle general HTTP requests, while using OpenSwoole to handle WebSocket requests.

Are you ready to explore Burner?

[Quick Start](/general/quickstart)
