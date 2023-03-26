---
title: OpenSwoole 常用指令
---

# OpenSwoole 常用指令

### 啟用伺服器

```
php spark burner:start
```

在預設的情況下，Burner 會讀取寫在 `app/Burner.php` 預設驅動程式。當然，你也可以強制在指令中指定 Burner 以 `OpenSwoole` 驅動程式啟動伺服器，就像這樣：

```
php spark burner:start --driver OpenSwoole
```

{% info 備註 %}

`--driver OpenSwoole` 這個參數也適用於下面提到的所有指令。

{% end %}

### 長駐模式

讓 OpenSwoole 在背景工作。

When you run the server with this option, Burner will ignore the Automatic reload setting.

```
php spark burner:start --daemon
```

{% info 備註 %}

當你的伺服器執行在常駐模式時，Burner 將會無視你所開啟的自動重載。

{% end %}

### stop server

Burner 會使用 `SIGTERM` 訊號關閉程序，會等待當前的任務執行完畢後再停止伺服器，不會直接殺死程序。

```
php spark burner:stop
```

### reload worker

就像 [官方文件](https://openswoole.com/docs/modules/swoole-server-reload#hot-code-linux-signal-trigger) 說的那樣，這個模式只會重新載入執行中的 Worker 。請注意，這個模式可能無法處理所有情況。例如：當你透過 `composer require/update` 對專案的核心 PHP 檔案產生改變時。

```
php spark burner:reload --mode worker
```

```
php spark burner:reload --mode task_worker
```

### restart worker

Burner 會替你停止伺服器後再重新啟動伺服器。

無論你的伺服器執行在長駐模式或開發模式下，都可以使用這個指令來重啟。

```
php spark burner:restart
```
