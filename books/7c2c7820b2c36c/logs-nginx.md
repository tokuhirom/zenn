---
title: "nginx のログを送信する"
---

nginx のアクセスログを､さくらのクラウドのモニタリングスイートに送信するサンプルです｡

## nginx の設定

`log_format` の設定を行い､JSON 形式でログを出力することにより､モニタリングスイートで扱いやすくすることが出来ます｡`http_latency_sec` 等のフィールド名は､モニタリングスイートで予約されている名前です｡マニュアルの [ログの構造化](https://manual.sakura.ad.jp/cloud/appliance/monitoring-suite/about.html#monitoring-suite-log-structure) セクションにまとまっています｡`http_latency_sec`, `http_request_method` などの､モニタリングスイート側で定義されているフィールドを送信することで､扱いやすくなります｡

https://github.com/tokuhirom/monitoring-suite-otelcol-sample/blob/main/docs/logs-nginx/nginx.conf

## otelcol の設定

https://github.com/tokuhirom/monitoring-suite-otelcol-sample/blob/main/docs/logs-nginx/otelcol-config.yaml

## 実行結果

このようにログの内容を確認できます｡

![alt text](/images/monitoringsuite-otelcol/logs-nginx-result.png)

