---
title: "はじめに"
---

本書は､さくらのクラウドのモニタリングスイートに [OpenTelemetry Collector（otelcol）](https://opentelemetry.io/docs/collector/) でログ･メトリクスを送信するため設定ファイルのサンプル集です｡

## 免責

本書の執筆は個人の活動として行われているものであり､さくらインターネットが責任を負うものではありません｡

## さくらのクラウドのモニタリングスイートとは

[さくらのクラウドのモニタリングスイート](https://cloud.sakura.ad.jp/products/monitoring-suite/)は､システム監視を行うためのプラットフォームであり､ログ･メトリクス･トレーシングを包括的にサポートしています｡

本書ではその中でも､otelcol を用いてログ･メトリクスを送信する方法について説明します｡

## OpenTelemetry Collector（otelcol）とは？

OpenTelemetry Collector（通称 otelcol）は、様々な形式のメトリクスやログ、トレースデータを受け取り、変換・集約・エクスポートできるOSSのデータコレクターです。
クラウドやオンプレミス環境での監視・observability を実現するための中心的なコンポーネントです。

このリポジトリは、さくらのクラウドのモニタリングスイートと連携するための設定例や、テスト用のデータ送信スクリプトを含みます。

## 本書のサンプル

本書のサンプルは､[tokuhirom/monitoring-suite-otelcol-sample](https://github.com/tokuhirom/monitoring-suite-otelcol-sample/) で公開されています｡docker-compose が動く環境であれば､すぐに試すことができるようになっています｡

### `.env` ファイルの作成

.env を設定します｡

メトリクスの設定情報は以下のように取得してください｡

![alt text](/images/monitoringsuite-otelcol/metrics-settings.png)

ログの設定情報は以下のように取得してください｡

![alt text](/images/monitoringsuite-otelcol/logs-settings.png)

`.env` ファイルの中身は以下のように記述します｡

```ini
SAKURA_MONITORINGSUITE_METRICS_ENDPOINT=https://***.metrics.monitoring.global.api.sacloud.jp/prometheus
SAKURA_MONITORINGSUITE_METRICS_CREDENTIALS=met-***-***

SAKURA_MONITORINGSUITE_LOGS_ENDPOINT=***.logs.monitoring.global.api.sacloud.jp
SAKURA_MONITORINGSUITE_LOGS_CREDENTIALS=log-***-***
```

## サンプルの実行

作成した `.env` ファイルを用いて､以下のように実行すると､ログまたはメトリクスの送信が開始されるようになっています｡


```sh
dotenv -f ../.env run docker compose up --build
```

