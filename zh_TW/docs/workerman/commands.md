---
title: Workerman 常用指令
---

# Workerman 常用指令

### 啟動伺服器

當你不傳入任何參數時，伺服器預設會以 debug 模式啟動。

```
php spark burner:start
```

預設情況下，Burner 會讀取 `app/Burner.php` 中的預設驅動。當然，你也可以強制 Burner 以 `Workerman` 驅動來執行指令，像這樣使用參數：

```
php spark burner:start --driver Workerman
```

{% info 備註 %}

`--driver Workerman` 這個參數也適用於下面提到的所有指令。

{% end %}

### 長駐模式

讓 RoadRunner 在背景執行。

```
php spark burner:start --daemon
```

此時，Burner 會將輸出導向到 `/dev/null`，你必須在 `app/Config/Workerman.php` 中定義 `logFile` ：

```php
public $logFile = 'YourloggingPath/workerman.log';
```

### stop server

```
php spark burner:stop
```

### 重新啟動伺服器

伺服器將會被真正的關閉，然後重新啟動。

```
php spark burner:restart
```

### 重新載入伺服器

伺服器會汰換掉所有的 worker，然後載入新的 worker。

```
php spark burner:reload
```

{% info 備註 %}

這種方式可能無法處理所有情況，例如，透過 `composer require/update` 產生的專案檔案改變。

{% end %}

### 伺服器狀態

```
php spark burner:workerman status
```

### more command

直接將指令傳遞給 Workerman 的入口 php 檔案。

```
php spark burner:workerman [workerman_comands]
```

你可以參考官方 [Workerman 文件](https://github.com/walkor/workerman#available-commands) 來構建你的指令。
