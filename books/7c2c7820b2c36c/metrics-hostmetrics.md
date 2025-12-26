---
title: "hostmetrics を送信する"
---

モニタリングスイートにホストメトリクスを送信するサンプルです｡

## otelcol 設定例

シンプルにホストメトリクスをモニタリングスイートに送信する例です｡

https://github.com/tokuhirom/monitoring-suite-otelcol-sample/blob/main/metrics-hostmetrics/otelcol-config.yaml

## 実行方法

```sh
git clone git@github.com:tokuhirom/monitoring-suite-otelcol-sample.git
cd monitoring-suite-otelcol-sample/metrics-hostmetrics/
dotenv -f path/to/.env run docker compose up
```

## 出力例

otelcol の hostmetrics で取得されたデータが送信されます｡

![hostmetrics をモニタリングスイートで表示した様子](/images/monitoringsuite-otelcol/metrics-hostmetrics-result.png)
