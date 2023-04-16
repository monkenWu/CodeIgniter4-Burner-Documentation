---
title: RoadRunner 開發建議
---

# RoadRunner 開發建議

## 自動重新載入

RoadRunner 預設的情況下，必須在每次修改 php 檔案後重啟伺服器，你所做的修改才會生效，這在開發上似乎不那麼友善。

你可以修改你的 `.rr.yaml` 組態設定檔案，加入以下設定後以 `-d` 開發模式啟動 RoadRunner Server，它將會自動偵測 PHP 檔案是否修改，並即時重新載入 Worker 。

```yaml
reload:
  interval: 1s
  patterns: [ ".php" ]
  services:
    http:
      recursive: true
      ignore: [ "vendor" ]
      patterns: [ ".php", ".go", ".md" ]
      dirs: [ "." ]
```

## 在只有一個 Worker 的環境中開發與除錯

因為 RoadRunner 與其他伺服器軟體（Nginx、Apache）有著根本上的不同，每個 Codeigniter4 將會以 Worker 的形式持久化於記憶體中，HTTP 的請求會重複利用到這些 Worker 進行處裡。所以，我們最好在只有單個 Worker 的情況下開發軟體並測試是否穩定，以證明在多個 Woker 的實際環境中能正常運作。 

你可以參考以下 `.rr.yaml` 設定將 Worker 的數量降到最低：

```yaml
http:
  address: "0.0.0.0:8080"
  static:
    dir: "./public"
    forbid: [".htaccess", ".php"]
  pool:
    num_workers: 1
    # max_jobs: 64
    # debug: true
```
