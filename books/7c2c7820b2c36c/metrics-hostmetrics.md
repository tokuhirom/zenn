---
title: "hostmetrics を送信する"
---

モニタリングスイートにホストメトリクスを送信するサンプルです｡

## otelcol 設定例

今回はサンプルなので､docker container の中の procfs を見た結果を送信していますが､実際にはホスト側の `/proc` をマウントして実装する必要があります｡

https://github.com/tokuhirom/monitoring-suite-otelcol-sample/blob/main/docs/metrics-hostmetrics/otelcol-config.yaml

## 実行方法

```sh
dotenv -f ../.env run docker compose up
```

## 出力例

otelcol の hostmetrics で取得されたデータが送信されます｡

![hostmetrics をモニタリングスイートで表示した様子](/images/monitoringsuite-otelcol/metrics-hostmetrics-result.png)
