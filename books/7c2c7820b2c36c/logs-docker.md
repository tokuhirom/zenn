---
title: "Docker の JSON ログを送信する"
---

Docker の json ログを､さくらのクラウドのモニタリングスイートに送信するサンプルです｡

## otelcol 設定例

https://github.com/tokuhirom/monitoring-suite-otelcol-sample/blob/main/docs/logs-docker/otelcol-config.yaml

## 実行

```sh
dotenv -f ../.env run docker compose up
```

## 実行結果

このようにログの内容を確認できます｡

![alt text](/images/monitoringsuite-otelcol/logs-docker-result.png)


