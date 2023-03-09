---
title: Docker 環境
---

# Docker 環境

使用 Docker 建構你的開發與正式環境有助於你減輕對環境建置的負擔，不論你使用何種驅動程式都需要預裝或開啟一些特別的擴充外掛。同時，使用 Docker 建構的環境將能夠幫助你在 Windows 環境下使用 Workerman 與 OpenSwoole 驅動程式。

你可以透過 [CodeIgniter4-Burner-Docker](https://github.com/monkenWu/CodeIgniter4-Burner-Docker) 儲存庫拉取到我們所推薦的 Docker 配置。

## 環境備註

所有的推薦組態都基於 [webdevops/Dockerfile](https://github.com/webdevops/Dockerfile) 映像檔，這個映像檔囊括了常用的 PHP 擴充外掛，你可以透過[這份文件](https://dockerfile.readthedocs.io/en/latest/content/DockerImages/dockerfiles/php.html)來調整更多細節，使你的環境更符合你的專案。

同時 OpenSwoole 的 Dockerfile 環境安裝指令參考了 [OpenSwoole 官方 Dockerfile](https://github.com/openswoole/docker-openswoole/tree/master/dockerfiles) 。 Workerman 的 Dockerfile 額外裝了 `php-event` 擴充外掛，這將使 Workerman 能夠獲得更好的執行效能。

{% info 備註 %}

你可以依照你的專案需求在 Dockerfile 安裝你所需要的任何東西，你只需要記得它們都是基於 [alpine linux](https://www.alpinelinux.org/) 的映像檔。

{% end %}

## 開始使用

* 將你的 CodeIgniter4 專案置於 `app` 資料夾中
* 使用 `docker-compose build` 將建構你的本地環境
* 使用 `docker-compose up -d` 使容器運行在背景
* 使用 `docker-compose exec app bash` 進入容器之中
* 在預設的情況下，你將被定位在專案根目錄中，你可以開始安裝 Burner 以及選擇你所需要的驅動程式
* 別忘了 `docker-compose down` 或 `docker-compose stop` 可以停止你的容器

`docker-compose.yml` 預設的連接埠映射為 `8080:8080` ，你可以依據自己的需求調整這個設定。

{% info 備註 %}

如果你的 CodeIgniter4 並不包含 `vendor` 資料夾，記得使用 `composer install` 拉取專案所需的程式庫。

{% end %}

