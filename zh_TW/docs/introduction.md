---
title: 歡迎使用 Burner！
---

# 歡迎使用 Burner！

Burner 是一款專屬於 CodeIgniter 的開箱即用的程式庫，它支援 [RoadRunner](https://roadrunner.dev/)、 [Workerman](https://github.com/walkor/workerman) ，與 [OpenSwoole](https://openswoole.com/) 等高效能網頁伺服器。你只需要開啟一些 php 擴充套件，即可大幅度地加速你的 CodeIgniter4 應用程式，使其能承受更高的負載並同時處理更多的連線。

## 系統需求

依據使用的驅動程式的不同，對於系統環境的需求也會有所不同。

### 基本需求

1. CodeIgniter Framework 4.3.0 以上
2. Composer
3. PHP8^

### OpenSwoole 

1. OpenSwoole 22^ 以上版本，你可以參考官方文件 [OpenSwoole 前置條件](https://openswoole.com/docs/get-started/prerequisites)

我們也推薦你閱讀：[如何安裝 OpenSwoole](https://openswoole.com/docs/get-started/installation)

{% info 備註 %}

OpenSwoole 不支援在 Windows 環境下原生執行。不過，你可以透過 WSL 或是 Docker 的方式使用 OpenSwoole。

{% end %}


### Workerman

1. 開啟 `php-pcntl` 外掛
2. 開啟 `php-posix` 外掛
3. 非必要，但我們推薦你安裝 [php-event](https://www.php.net/manual/en/book.event.php) 外掛，以取得更佳的執行效能

{% info 備註 %}

Workerman 不支援在 Windows 環境下原生執行。不過，你可以透過 WSL 或是 Docker 的方式使用 Workerman。

{% end %}

### RoadRunner

1. 開啟 `php-curl` 外掛
2. 開啟 `php-zip` 外掛
3. 開啟 `php-sockets` 外掛

## 架構

Burner 由數個程式庫組成，分別是：

* [`burner-core`](https://github.com/monkenWu/CodeIgniter4-Burner) 是一個單純的 `PSR7` 與 `CodeIgniter4` 的轉換器，並且包含一些 CodeIgniter4 的指令整合；由於 Codeigniter4 並沒有實作完整的 [HTTP message 介面](https://www.php-fig.org/psr/psr-7/) 介面，所以這個程式庫著重於 `PSR-7` 介面 與 Codeigniter4 HTTP 介面 的同步。

* [`burner-OpenSwoole`](https://github.com/monkenWu/CodeIgniter4-Burner-OpenSwoole) 使用了這個驅動程式，你的 CodeIngiter4 將能夠透過 OpenSwoole 執行。除了 HTTP 伺服器以外，Burner 還提供了 WebSocket 伺服器的支援。

* [`burner-Workerman`](https://github.com/monkenWu/CodeIgniter4-Burner-Workerman) 使用了這個驅動程式，你的 CodeIngiter4 將能夠透過 Workerman 執行，在一些[測試場景](https://www.techempower.com/benchmarks/#section=data-r21&hw=ph&test=query&l=zik073-sf&a=2) 中，Workerman 表現出了絕佳的效能優勢。

* [`burner-RoadRunner`](https://github.com/monkenWu/CodeIgniter4-Burner-RoadRunner) 使用了這個驅動程式，你的 CodeIngiter4 將能夠透過 RoadRunner 執行。與其他 Driver 不同，它具備一個由 `Go` 語言實作 PHP 伺服器，除了框架本身的支援外，官方也提供了許多便利的外掛。

你可以替你的專案選擇最適合的驅動程式，這些選擇沒有絕對的好壞，端看專案需求以及開發人員對於何種技術較為熟悉。

有了 Burner 的支援，你的專案將可以輕易地在不同的高效能 PHP 伺服器中自由切換，或以不同的 port 同時執行在兩個不同的伺服器上。比如：透過 RoadRunner 處理一般的 HTTP 請求，同時使用 OpenSwoole 處理 WebSocket 請求。

準備好開始探索 Burner 了嗎？

[快速開始](/general/quickstart) 
