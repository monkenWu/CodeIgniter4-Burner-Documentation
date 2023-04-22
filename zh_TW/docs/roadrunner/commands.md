---
title: RoadRunner 常用指令
---

# RoadRunner 常用指令

### 啟動伺服器

當你不傳任何參數時，預設會啟動伺服器。

```
php spark burner:start
```

預設 Burner 會讀取 `app/Burner.php` 中寫入的預設驅動。當然你也可以強制 Burner 使用 `RoadRunner` 驅動來執行指令，像這樣：

```
php spark burner:start --driver RoadRunner
```

{% info 備註 %}

`--driver RoadRunner` 這個參數也適用於下面提到的所有指令。

{% end %}

你也可以使用下面的參數來根據你的需求來構建你的指令。

* -c: 路徑到設定檔。
* -w: 設定工作目錄。
* --dotenv: 將環境變數從 .dotenv 檔案填入到環境中。
* -d: 啟動 pprof 伺服器。注意，這不是 debug，要使用 debug log 等級，請使用 [logs](https://roadrunner.dev/docs/plugins-logger/2.x/en)
* -s: 靜音模式。
* -o: 用你的值來覆蓋設定檔的值，例如：
    ```
    -o=http.address=:8080
    ```
    這會覆蓋 .rr.yaml 中的 http.address。

{% info 備註 %}

注意 Burner 已經使用了 `-c`, `-p` 和 `-w` 參數，你需要避免再次使用相同的參數。

{% end %}

#### 長駐模式

讓 RoadRunner 在背景執行。

```
php spark burner:start --daemon
```

使用這個模式 Burner 會將輸出導向到 `/dev/null`，你必須在 `.rr.yaml` 中定義 `log_output` 來像這樣：

```yaml
logs:
mode: development
output: stdout
file_logger_options:
    log_output: "{{log_path}}"
    max_size: 100
    max_age: 1
    max_backups : 5
    compress: false
```

### 停止伺服器

這個指令只能在後台模式下執行。

```
php spark burner:stop
```

強制關閉伺服器。

```
php spark burner:stop -f
```

### Workers 狀態

取得所有 Workers 目前的運行資訊。

```
php spark burner:rr workers
```

每秒更新一次的持續互動模式。

```
php spark burner:rr workers -i
```

### 更多指令

直接執行 RoadRunner 的 rr 指令。

```
php spark burner:rr [rr_comands]
```

你可以參考官方的 [RoadRunner 文件](https://roadrunner.dev/docs/app-server-cli/2.x/en) 來構建你的指令。 

{% info 備註 %}

注意 Burner 已經使用了 `-c`, `-p` 和 `-w` 參數，你需要避免再次使用相同的參數。

{% end %}

