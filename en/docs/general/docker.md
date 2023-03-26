---
title: Docker Environment
---

# Docker Environment

Using Docker to build your development and production environments can help you reduce the burden of environment setup, regardless of which driver you use. You need to pre-install or enable some special extensions. At the same time, the environment built with Docker can help you use the Workerman and OpenSwoole drivers under the Windows environment.

You can pull the Docker configuration we recommend from the [CodeIgniter4-Burner-Docker](https://github.com/monkenWu/CodeIgniter4-Burner-Docker) repository.

## Environment notes
All recommended configurations are based on the [webdevops/Dockerfile](https://github.com/webdevops/Dockerfile) image, which includes common PHP extensions. You can adjust more details through [this document](https://dockerfile.readthedocs.io/en/latest/content/DockerImages/dockerfiles/php.html) to make your environment more suitable for your project.

At the same time, the Dockerfile environment installation instructions of OpenSwoole refer to the [official Dockerfile of OpenSwoole](https://github.com/openswoole/docker-openswoole/tree/master/dockerfiles). The Dockerfile of Workerman additionally installed php-event extension, which will make Workerman get better performance.

{% info Note %}

You can install anything you need in the Dockerfile according to your project requirements. You just need to remember that they are all based on the [alpine Linux](https://www.alpinelinux.org/)  image.

{% end %}

## Getting Started

* Place your CodeIgniter4 project in the app folder
* Use `docker-compose build` to build your local environment
* Use `docker-compose up -d` to run the container in the background
* Use `docker-compose exec app bash` to enter the container
* By default, you will be located in the project root directory, you can start installing Burner and choose the driver you need
* Don't forget that `docker-compose down` or `docker-compose stop` can stop your container
* The default port mapping of `docker-compose.yml` is `8080:8080`, you can adjust this setting according to your own needs.

{% info Note %}

If your CodeIgniter4 does not contain the vendor folder, remember to use `composer install` to pull the libraries required by the project.

{% end %}