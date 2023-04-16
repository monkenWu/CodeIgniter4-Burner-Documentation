---
title: 快速開始
---

# 快速開始

Burner 目前僅支援使用 Composer 安裝，這是因為 Burner 替不同驅動程式提供了一致的指令介面與自動載入方式，這使得 Burner 無法輕易地讓開發者自行部署。

## 安裝 Burner

首先，使用 Composer 將 Burner 安裝進你的 CodeIgniter4 專案中。

選擇一款合宜的驅動程式進行安裝，目前 Burner 提供你三種選擇：

1. 
    ```
    composer require monken/codeigniter4-burner-openswoole
    ```

2. 
    ```
    composer require monken/codeigniter4-burner-workerman
    ```

3. 
    ```
    composer require monken/codeigniter4-burner-roadrunner
    ```

在安裝之前，你得先確定目前的系統環境是否滿足了這些驅動程式的要求。你能夠先參考 [系統需求](/introduction) 提到的內容建構你的系統環境，同時我們也推薦你 [透過Docker建構](/general/docker) 你的開發環境。

{% info 備註 %}

若你需要，你也可以為你的專案安裝多個驅動程式，並在啟動時選擇你想要的驅動程式。你可以參考各驅動程式的「常用指令」條目，了解如何在執行伺服器啟動指令時選擇你想要的驅動程式。

{% end %}


## 初始化

依據你所選擇不同的驅動， `burner:init` 指令也需要傳入不同的參數。

```
php spark burner:init [OpenSwoole, Workerman, RoadRunner]
```

執行完成後，你會發現在 `app/Config` 目錄下多了兩個檔案，分別是 `Burner.php` 以及一個由驅動程式名稱命名的檔案，比如： `OpenSwoole.php` 。

Burner 是一款開箱即用的程式庫，已預設了所有組態設定。你可以直接啟動伺服器，直到有需要時再去調整組態設定的內容即可。

## 啟動伺服器

現在，你可以透過以下指令直接啟動你的伺服器。

```
php spark burner:start
```

在預設的情況下 Burner 將伺服器監聽的 `port` 設定在 `8080` ，現在打開你的瀏覽器瀏覽 `localhost:8080` ，你將會看到目前專案的 Home 頁面。

## 結語

一言以蔽之， Burner 簡單且輕量。

願它能成為你的 CodeIgnter4 專案的最佳燃料。

在開發前，推薦你繼續閱讀 [開發建議](/general/suggestion)
